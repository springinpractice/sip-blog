---
layout: post
title: "Aggregating TP90 data with Splunk"
date: 2013-09-25 19:38
comments: true
categories: [Quick Tips]
---
One of the challenges around [TP90](http://stackoverflow.com/questions/17435438/what-do-we-mean-by-top-percentile-or-tp-based-latency) data is aggregating it. I wrote about this [here](http://stats.stackexchange.com/questions/49017/options-for-aggregating-dispersion-data), and offered a solution based on histograms. Here I'm going to describe another approach, this time based on weighted averages.

<!-- more -->

To be concrete, suppose we have web page response times, and we've computed hourly TP90s for them. Now we want to know the TP90 for a week. To get the exact answer, we'd need to look at all the response times for the whole week, but that can be pretty expensive for a busy site. The challenge is to estimate the week's TP90 based on the hourlies.

We can do that by computing a [weighted average](http://en.wikipedia.org/wiki/Weighted_arithmetic_mean), where we use counts to establish weights. The weight for any given hour is the number of requests that hour divided by the total number of requests that week. Apply the hourly weights to the hourly TP90s and then sum them all up to get the weighted average over the week.

Here's how to do it in [Splunk](http://www.splunk.com/). The trick here is the [eventstats](http://docs.splunk.com/Documentation/Splunk/5.0.5/SearchReference/Eventstats) command, which makes the sum of the hourly counts available on a per-row basis so we can use it to calculate weights.

    index=webRequestSummary earliest=-7d@d latest=@d
        | eventstats sum(hourlyCount) as totalCount
        | eval weight = hourlyCount / totalCount
        | eval weightedHourlyTP90 = weight * hourlyTP90
        | sum(weightedHourlyTP90) as weightedAvgTP90

That's it.
