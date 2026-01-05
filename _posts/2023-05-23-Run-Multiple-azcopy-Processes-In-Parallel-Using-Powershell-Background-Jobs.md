---
layout: post
title:  Run azcopy processes in parallel using Powershell Background Jobs
author: Rik Groenewoud
tags: microsoft azure azuread aad identitymanagement
---

## Intro

For a migration project of a large number of blobs from one storage account to another, I was looking for a way to speed up the process. It turned out that Powershell background jobs were a nice way to do this. It does cost a lot of CPU power to run multiple ```azcopy``` processes simultaneously but with a big VM running in the Azure Cloud, it turned out we could speed up the process significantly. In the end we transferred close to **one million** files in 8 to 10 minutes.

In this blog I show how we did it.

## How it works

With the ```Start-Job``` cmdlet a Powershell job starts in the background. This way it is possible to run multiple jobs in parallel. In this use case I had to make sure each background job copied over a unique portion of the containers in the storage account. This way I could make sure that the jobs would not interfere with each other. I used:

```powershell
    $containers = az storage container list --account-name $storageAccount --only-show-errors --num-results '*' --sas-token $sourceTokenTsv | ConvertFrom-Json
```

to get a list of all containers in the storage account. I then divided the containers up in separate arrays like this:

```powershell
    for ($i = 0; $i -lt $containers.count; $i += 2) {
        $arrays += , @($containers[$i..($i + 1)]);
    }
 ```

Then for each array I started the background job like this:

```powershell
    Start-Job -ArgumentList $storageAccount, $array, $targetContainers, $targetStorageAccount, $targetToken, $sourceTokenTsv, $targetTokenTsv -ScriptBlock {

        foreach ($container in $args[1]) {

        if ($args[2].name.Contains($container.name) -eq $false) {
            az storage container create --name $container.name --account-name $args[3] --sas-token $args[4]
        }
        Write-Warning "Copying $($container.name) from $($args[0]) to $($args[3]) "
        azcopy copy "https://$($args[0]).blob.core.windows.net/$($container.name)?$($args[5])" "https://$($args[3]).blob.core.windows.net/$($container.name)?$($args[6])" --recursive --overwrite ifSourceNewer --log-level NONE

        $total = $args[1].Count
        $counter = $counter + 1

        Write-Host "Processed container $($counter) of $($total)"
    }
    }
```

The ```-Argumentlist``` is the place where the parameters for the ```-scriptblock``` are set. In this case I passed in the storage account name, the array of containers, the target containers, the target storage account, the target token, the source token and the target token.

As soon as the jobs are running, ```Get-Job``` can be used to see an overview of all running Jobs and there status.
```Receive-Job``` can get the output of the job. Using the ```-keep``` switch the history of the output is preserved.

This was basically it. You can test out what is the best container to job ratio for your use case. For one of the storage accounts I used 5 parallel jobs, for others I used more. The problem with too much simultaneous jobs is that you will run out of memory and it will take a long time before the jobs start running. So you have to find the right balance for your use case.

If you have any questions or remarks, please let me know in the comments.

<script src="https://giscus.app/client.js"
        data-repo="RikGr/cloudwoud"
        data-repo-id="R_kgDOHLlC9w"
        data-category="Announcements"
        data-category-id="DIC_kwDOHLlC984CO_2O"
        data-mapping="pathname"
        data-reactions-enabled="0"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="light"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>