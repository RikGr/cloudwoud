---
layout: post
title:  Use Resource Graph Queries in Azure Alerts
author: Rik Groenewoud
tags: microsoft azure observability alerts monitoring
---
## Intro

Over the last years, I tried several methods to keep track of the expiration date of manually imported SSL certificates in Azure.  I created rather extensive PS scripts that scraped all SSL certificates from App Services, App Gateways. By running this script in a Logic App, I could throw alerts when the expiration date was coming close.
Another approach I tried,was the SSL health check from the Availability monitoring in App Insights. This solution looked pretty good, but in practice ít caused confusion. The availability alert triggered on SSL certicates less than 30 days before expiration while the website was perfectly reachable. It was not clear that this alert could also be the SSL certificate that was about to be expired in 30 days. Furthermore, you have to make sure that all custom domains are covered with availability alerts. And what about the SSL certificates on App Gateways?

## Resource Graph alerts

 Recently I found out that it now is possible to use **Resource Graph queries** inside Azure Alerts. This makes it possible to query on resource properties and create an alert based on values on these properties. This means I can query for the expiration date of SSL certificates directly and create an alert based on that query.

This is how I approach the creation of such an alert:

1. Go to the Resource Graph Explorer in the Azure Portal
2. Create your desired query. My query looks like this:

```kusto
Resources
| where type == "microsoft.web/certificates"
| extend ExpirationDate = todatetime(properties.expirationDate)
| project ExpirationDate, name, resourceGroup, properties.expirationDate
| where ExpirationDate < (datetime(2023-11-30T18:21:38.0000000Z)) + 30d
| order by ExpirationDate asc
```

3. Create an alert rule and use Custom ```Log Search``` as ```Signal Name```. In the query field you can now paste the query you created in the Resource Graph Explorer. The only thing you have to do is add `arg("").` in front of `Resources` like this:

```kusto
arg("").Resources | where type == "microsoft.web/certificates" | extend ExpirationDate = todatetime(properties.expirationDate) | project ExpirationDate, name, resourceGroup, properties.expirationDate | where ExpirationDate < now() + 30d | order by ExpirationDate asc
```

For ```Measure``` I use ```Table Rows``` where Agregation is Count and Agregation granularity is 5 minutes. The ```threshold``` can be set to ```1```.

It is important to note that the **identity** of the alert must have **Reader** access to the Log Analytics Workspace AND the resources you want to get data from. In my case I gave the system assigned identity reader role on subscription level to be sure all SSL certificates in the subscription are covered.

This is the end result of the condition:

![Alert condition](/images/blog-7.1.png)

## Benefits

- The main benefit I see, is that we now can use a "normal" alert to monitor on resource specific properties or conditions. It is no longer need to write your own scripts or use App Insights or come up with another workaround.
- Wit a query we "scan" the selected scope for certificates. This way you we sure you don't miss any certificates

## Other use cases?

Another alert I created by a using Resource Graph Kusto query is a check on the state of Logic Apps and Function Apps. When they are disabled, the alert triggers. The queries looked like this:

```kusto
arg("").resources | where type == "microsoft.logic/workflows" | where properties.state != "Enabled"
```

and

```kusto
arg("").resources | where type == "microsoft.web/sites" | where properties.kind == "functionapp" | where properties.state != "Running"
```

If you have other nice use cases, please let me know in the comments!

## Further Reading

- [Announcement of this feature](https://azure.microsoft.com/en-us/updates/public-preview-azure-log-alerts-support-for-azure-resource-graph-arg/)
- [Azure Docs on creating Log Alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-create-log-alert-rule)

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
