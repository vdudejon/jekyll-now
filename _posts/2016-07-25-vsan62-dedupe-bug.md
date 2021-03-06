---
layout: post
title: Bug in VSAN 6.2 De-dupe scanning running on hybrid datastores
permalink: vsan62-dedupe-bug.html
description: Some Description
date: 2016-07-25 9:00:00
tags: vsan bug sexigraf
published: true
---
We’ve been testing out VSAN here at work and noticed that one of the clusters we rolled out had serious latency issues. We initially blamed the application running on the hosted VMs, but when it continued to get worse we finally opened a case with VMware. Here’s a chart of the kind of stats we were seeing (courtesy of SexiGraf):

![VSAN Performance Before]({{ site.baseurl }}images\2014-07-25-vsan62-bug-1.png){: .center-image }

Read latency in particular was very high on the datastore level, IOPS weren’t great, and Read Cache Hit Rate was low. We also saw that read and write latency was high on the VM level. After we opened a ticket with VMware, they discovered an undocumented bug in VSAN 6.2 where deduplication scanning is running even though deduplication is turned off (and actually unsupported in hybrid mode VSAN altogether). They provided the following solution: 


>For each host in the VSAN cluster:
>
1. Enter maintenance mode
2. SSH to the host and run: "esxcfg-advcfg -s 0 /LSOM/lsomComponentDedupScanType"
3. Reboot the host

After we applied the fix, the cluster rebalanced for a little while and came back looking much, much better. In the below graph, you can see right when the fix was applied and see read latency drop, IOPS increase, and read cache hit rate jump to the high 90-percents:

![VSAN Performance Before]({{ site.baseurl }}images\2014-07-25-vsan62-bug-2.png){: .center-image }

And for good measure, this is how it’s looked since:

![VSAN Performance Before]({{ site.baseurl }}images\2014-07-25-vsan62-bug-3.png){: .center-image }

So to summarize, if you are running hybrid VSAN 6.2, you should definitely check your latency and read cache hit rate. If you’re experiencing high latency and poor read cache hit rate, go through and change /LSOM/lsomComponentDedupScanType on all your hosts to 0. I can’t take credit for actually discovering this, so thank you to my coworker [@per_thorn](https://twitter.com/per_thorn) for tracking it down. And thank you [@thephuck](https://twitter.com/thephuck) for letting me write it up on [his blog](http://thephuck.com)!

Originally posted (by me) at [ThepHuck.com](http://thephuck.com/virtualization/bug-in-vsan-6-2-de-dupe-scanning-running-on-hybrid-datastores/)

## UPDATE
VMware has posted a KB about this, which I did not realize at the time of writing the blog: [https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2146267](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2146267)
