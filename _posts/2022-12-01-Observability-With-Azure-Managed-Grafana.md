---
layout: post
title: Observability with Azure Managed Grafana
author: Rik Groenewoud
tags: microsoft azure observability grafana
---

During my visit to Monitorama 2022 it became clear to me that the phenomenon of "Observability" is here to stay. 
The people from Honeycomb.io even wrote an O'Reilly book on the subject called: [Observability Engineering](https://info.honeycomb.io/observability-engineering-oreilly-book-2022). 

After I read the book and learned more on what is meant by Observability, my initial skepticism ("This must be the new buzz word for plain monitoring!") made place for enthusiasm. I truly believe it can help DevOps Engineers running complex distributed operations in the cloud. 

So when the call for papers came for our bi-annual XPRT magazine, I was eager to write an article on this subject. Together with my good colleague [Casper Dijkstra](https://xpirit.com/team/casper-dijkstra/) we decided to write on Observability theory and how to bring this into practice with Azure Managed Grafana. 

Below I give a short summary of the article. This hopefully triggers you to read the full version via the link provided at the bottom of this post.
## Summary

**Observability**

The definition used in the book "Observability Engineering": 

*“... our definition of “observability” for software systems is a measure of how well you can understand and explain any state your system can get into no matter how novel or bizarre”*

**Azure Managed Grafana and Observability**

Microsoft embraces the idea of Observability. They advocate the use of resources like Log Analytics Workspaces to centralize all logging and metric data and make it possible to query this combined data. 
For visualization Azure has basic dashboard functionality. We argue that Grafana can be a great addition to this. With the new Azure Managed Grafana offering it becomes very easy to start using Grafana. 

The most important assets are: 

- Usability: Quickly setup good looking dashboards with lots of choice in visualizations
- Bring together multiple data sources, also from outside of Azure (i.e. Azure DevOps)
- Not only visualize but also dive into the data by querying realtime logging and metrics from multiple data sources
- Large community with lots of dashboard templates to quickly be up to speed
- Integrate dashboards in your DevOps way of working by deploying dashboards as code using CI/CD tooling 

We then show a basic setup of how to run dashboards-as-code using a DevOps Pipeline and tell more about the cost aspect of this offering.

**Conlusion**

By bringing all these data sources together and combining them in a smart and useful way in Azure Managed Grafana, observability becomes something real and will add value to the existing tools that Azure offers. In the current age of highly complex distributed systems it is no longer about only trying to prevent issues from occurring, but to make sure engineers have the right tools to literally observe what is going on and to locate the issues as quickly as possible. 

## Full version
[Link](https://xpirit.com/wp-content/uploads/2022/10/Xpirit_XPRT_magazine_13_final.pdf?utm_campaign=Xpirit%20-%20Magazine%2013&utm_source=download-page) to the magazine with our article **from page 45**.

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
