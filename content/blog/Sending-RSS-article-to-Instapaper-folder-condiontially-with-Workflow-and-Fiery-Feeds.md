---
title: Sending RSS article to Instapaper folder condiontially with Workflow and Fiery Feeds
date: 2018-04-02T21:46:48+08:00
---

I use RSS reader all the time, from Goole Reader to Feedly, but as my current read flow, I almost never read articles on the RSS app but save directly into Instapaper for the ones matches my interests. Newsify was my first choice on iOS, it's very convenience by using it's customizable quick action to save articles, e.g. long press save article to Instapaper.

But there is another problem when saving too much article into the inbox folder of Instapaper, I can't decide which article to read firstly. As currently I have 220 articles need to be filter daily, some need to ready quickly such as news, others need to be read carefully even repeatedly(e.g. coding tutorials). I can not rely on Instapaper's estimate reading time order function although it is very helpful.

Until Fiery Feeds arrive with it URL Actions function to rescue. You can define customized URL Scheme with it to share the article link to any service can receive, of course Workflow. So my idea is, predefine some feeds url or domain names in Workflow, once it received the ULR from Fiery Feeds it will check if it is matches the patterns defined, if Yes save to Instapaper with special tags, if No save to Instapaper inbox.

``` url
workflow://x-callback-url/run-workflow?name=Instapaper%20Learn&id=B9E76B0C-1A3F-4A8D-B5B6-C5CB0A54697B&input={url}&x-success={callback}&x-source={source}
```

![](../img/fiery-feeds-action-instacondition.jpg)

It only passed `{url}`, that's what I need in Workflow, you can import to yours by [this link](https://workflow.is/workflows/3f82dc5039b54d898bd275ccc3435005).

![](../img/workflow-fiery-feeds-instacondition.jpg)
