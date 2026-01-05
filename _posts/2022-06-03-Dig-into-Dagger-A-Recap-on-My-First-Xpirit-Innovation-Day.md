---
layout: post
title: Dig into Dagger. A Recap on My First Xpirit Innovation Day
author: Rik Groenewoud
tags: microsoft azure dagger iac pulumi cicd docker
---

![InnoDay](/images/blog-2.1.jpg)

Bring all Xpirit employees together on a external location and let them work on innovative subjects of their own choosing. Then at the end of the day present the findings to each other in 5 minute lighting talks. This is the concept of the Xpirit Innovation Day in a nutshell. Innovation must be understood in the broadest sense of the word. So while most subjects indeed are focused on the newest technologies, others dive into creating new product propositions for our customers, our working culture or creating a new hiring assessment to name just a few examples.

I decided to join a topic with the nice alliterating title: Let's Dig into Dagger. I had read some LinkedIn post when Dagger went live, so I knew a little bit of what it was and was curious to see how it worked and performed. Together with my colleague Chris van Sluijsveld we were a team of two. Another team wanted to look into [Pulumi](https://www.pulumi.com/); a promising Infrastructure as Code (IaC) platform. We decided to connect our two subjects. <strong>To goal we set was that we want to be able to deploy the Pulumi infrastructure with our Dagger CD pipeline at the end of the day.</strong>

## What is dagger?

Let's start with a little background information of the tool itself. As stated on their [website](https://dagger.io) with Dagger you can build powerful CI/CD pipelines quickly, then run them anywhere. Written in [CUE](https://cuelang.org/), these pipelines run in a Docker container which means they can run on any platform that supports Docker. This includes running the pipeline _locally_, a great benefit when you are building and setting up a pipeline and what to get quick feedback and results.
For more information see the above mentioned website and their [docs](https://docs.dagger.io/) page. We also used the [github repo](https://github.com/dagger) for examples and extra details.

Just to be clear: the Dagger we talk about here is not the same as [Google Dagger](https://github.com/google/dagger\), the dependency injector.

## Can we make it work?

As we started our day and installed Dagger on our machines, it became clear quite early that there was no out-of-the-box support for Microsoft Azure as of yet. We found no documentation or code examples for deploying Azure infra or an integration with Pipelines. This meant we had to build up the pipeline from scratch.
So what are the building blocks we can work with? It all starts with a _Plan_ and in this plan you can define _Actions_ in which you define what you actually want to do. You can write and define your own Actions but luckily there are a lot of Actions out there you can import and use in your Dagger Plan (such as actions for Build and Run stuff).

We decided to start with installing the Azure CLI on the container. After the first few hours of trial and error, Googling and local testing we managed to do this. After lunch we decided to use a different container image. We started with the standard Alpine Linux image which is very fast and lightweight but has almost nothing pre-installed. We actually found an Docker image that had Pulumi already installed so this gave us a head start and less manually installs before the actual deployment could start.

We integrated Dagger into our Github environment were our repo was located as well. With the GitHub Action template provided in the [documentation](https://docs.dagger.io/1201/ci-environment) we had the Action up and running in no time.
It turned out our biggest challenge was to find out how the secrets in GitHub, which we used for authentication with Azure, could be imported into the dagger script and then be used in the bash script Action we used to execute the Pulumi commands. Eventually we figured out the correct syntax. Just when the first Lighting talk started at 3PM sharp, we managed to do a complete run of our pipeline and build the Pulumi Infrastructure into Azure!

## The pipeline

So we ended up with this code for the complete pipeline:

```
package pulumi

import (
	"dagger.io/dagger"
	//"dagger.io/dagger/core"
	//"universe.dagger.io/alpine"
	"universe.dagger.io/bash"
	"universe.dagger.io/docker"
)

dagger.#Plan & {
	client: {
		filesystem: {
			"./": read: {
				contents: dagger.#FS
			}
		}
		env: {
			PULUMI_ACCESS_TOKEN: string
			CLIENTID: string
			CLIENTSECRET: string
			TENANTID: string
		}
		}
	actions: {
		 	deps: docker.#Build & {
			steps: [
				docker.#Pull & {
					source: "pulumi/pulumi"
				},
				docker.#Copy & {
					contents: client.filesystem."./".read.contents
					dest:     "/src"
				},
				bash.#Run & {
					workdir: "/src"
					script: contents: #"""
						apt-get update
						curl -sL https://aka.ms/InstallAzureCLIDeb | bash
						apt-get install -y dotnet-sdk-6.0
						"""#
				}
			]
		}
			deployment: bash.#Run & {
			input:   deps.output
			workdir: "/src"
			args: [ client.env.PULUMI_ACCESS_TOKEN, client.env.CLIENTID, client.env.CLIENTSECRET, client.env.TENANTID ]
			script: contents: #"""
				export PULUMI_ACCESS_TOKEN=$1
				printenv
				cd pulumi
				pulumi login
				pulumi whoami
				export ARM_CLIENT_ID=$2
				export ARM_CLIENT_SECRET=$3
				export ARM_TENANT_ID=$4
				export ARM_SUBSCRIPTION_ID="f317d45c-55f5-4341-8d49-990b06d1c9a5"
				pulumi stack select xpirit-innovationday/dev
				pulumi up --yes
				"""#
		}
	}
}
```

### Action 1: The dependencies

In the first part of the script we establish the environment. With the <code>docker.#Pull</code> action we pul the pulumi docker container that will run the rest of the pipeline. With the <code>docker.#Copy</code> we import the pulumi files into the local environment. Lastly, with the <code>bash.#Run</code> action we install the Azure CLI on the container. This makes it possible to talk to Azure.

```
deps: docker.#Build & {
    steps: [
        docker.#Pull & {
            source: "pulumi/pulumi"
        },
        docker.#Copy & {
            contents: client.filesystem."./".read.contents
            dest:     "/src"
        },
        bash.#Run & {
            workdir: "/src"
            script: contents: #"""
                apt-get update
                curl -sL https://aka.ms/InstallAzureCLIDeb | bash
                apt-get install -y dotnet-sdk-6.0
                """#
        }
    ]
}
```

### Action 2: The Pulumi deployment

In the second part we run the actual deployment of the Pulumi IaC. As said we had some struggles to figure out the correct syntax for the environment variables. In the script itself we refer to the variables with <code>$1</code> for the first in the args array, <code>$2</code> for the second etc. etc.

```
deployment: bash.#Run & {
input:   deps.output
workdir: "/src"
args: [ client.env.PULUMI_ACCESS_TOKEN, client.env.CLIENTID, client.env.CLIENTSECRET, client.env.TENANTID ]
script: contents: #"""
	export PULUMI_ACCESS_TOKEN=$1
	printenv
	cd pulumi
	pulumi login
	pulumi whoami
	export ARM_CLIENT_ID=$2
	export ARM_CLIENT_SECRET=$3
	export ARM_TENANT_ID=$4
	export ARM_SUBSCRIPTION_ID="f317d45c-55f5-4341-8d49-990b06d1c9a5"
	pulumi stack select xpirit-innovationday/dev
	pulumi up --yes
	"""#
}
```

## Pro's and Cons

I am sure there are more pros and cons for this tool but for me the biggest pros and cons are:

### Pros

<ul>
 <li> Test locally. This means a very fast feedback loop and with that speed up the development of the pipeline.</li>
 <li> You can integrate Dagger with any other CI tool you already use. </li>
</ul>

### Cons

<ul>
 <li> Since I am not familiar with CUE this means yet another scripting language I have to get into before I can easily work with Dagger. </li>
 <li> Not much documentation and support for Dagger in general and for Microsoft especially. Of course this can change in the future, but for now, Dagger does not seem mature enough to switch over from well-known CI/CD solutions we already have in place. </li>
</ul>

## Conclusion

For me this day was a great success. Working together with colleagues I did not work with before and explore a new and unknown technology. I want to thank [Chris van Sluijsveld](https://xpirit.com/team/chris-van-sluijsveld/) for taking the lead on this exploration. We started the day blank and really had to figure out how we could make this work. As the hours progressed we got more familiar with the CUE syntax and all that Dagger had to offer.
I would say that Dagger definitely has potential and time will tell if it will get traction and further support from the community. I would say that today it is just too early to use this in a customer production setting, especially when you want to use this with Microsoft Azure and i.e. Azure DevOps Pipelines.

At the end of the day there was a big surprise left. The theme of the day was _"Chef et les Ops"_ and as it turned out they saved the best for last. Three colleagues with a big love for cooking had been busy all day in a professional kitchen to take their cooking skills to the next level. We were served no less than 9 (!) courses all very delicious and accompanied by the best carefully selected wines. The quality was what you can expect from Xpirit: without compromise; simply the best you can get! If you are interested on the how and why of this marvelous dinner see [this article](https://www.linkedin.com/pulse/taste-innovation-xpirit-bv/?trackingId=EBDJkuNnTVq27VxunTEjhA%3D%3D).

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
