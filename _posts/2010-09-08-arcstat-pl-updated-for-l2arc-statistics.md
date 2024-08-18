---
id: 31
title: arcstat.pl updated for L2ARC statistics
date: 2010-09-08T17:03:26+00:00
author: mharsch
layout: post
guid: http://blog.harschsystems.com/?p=31
permalink: /2010/09/08/arcstat-pl-updated-for-l2arc-statistics/
categories:
  - Tools
tags:
  - arcstat
  - L2ARC
  - zfs
---
Three years ago, Neelakanth Nadgir released arcstat.pl: a Perl script that prints ZFS ARC statistics in a vmstat-like fashion.  I found this tool to be very useful, but it was missing support for L2ARC statistics (the [L2ARC](https://www.brendangregg.com/blog/2008-07-22/zfs-l2arc.html) hadn&#8217;t been invented yet).  With neel&#8217;s permission, I&#8217;ve <a href="http://github.com/mharsch/arcstat" target="_blank">updated arcstat.pl</a> to support L2ARC statistics.  Now it can be used to view ARC and L2ARC data side-by-side: providing a more complete view of ZFS caching performance on a system configured with L2ARC devices.

Let&#8217;s say I&#8217;m interested in watching the following stats over time:

  * ARC Accesses, Hits/sec, Misses/sec, Hit percentage
  * L2ARC Accesses, Hits/sec, Misses/sec, Hit percentage
  * ARC size and L2ARC size

I&#8217;ll specify the output fields I&#8217;m interested in as follows:

    
    $ ./arcstat.pl -f 
    >read,hits,miss,hit%,l2read,l2hits,l2miss,l2hit%,arcsz,l2size
    

At first, the L2ARC is cold, so we either hit in the ARC, or we miss both ARC and L2ARC (and are forced to do disk i/o):

    
    read  hits  miss  hit%  l2read  l2hits  l2miss  l2hit%  arcsz  l2size
      8K    7K   272    96     272       0     272       0   912M    101M
      9K    9K   205    97     205       0     205       0   919M    102M
      7K    6K   281    96     281       0     281       0   922M    102M
      2K    2K   119    94     119       0     119       0   922M    103M
      8K    8K   517    94     517       0     517       0   930M    103M
      2K    2K   161    92     161       0     161       0   933M    103M
      1K    1K    46    97      46       0      46       0   934M    103M
      1K    1K    38    97      38       0      38       0   935M    103M
      1K    1K    21    98      21       0      21       0   911M    104M
      1K    1K    28    97      28       0      28       0   912M    104M
      1K    1K    18    98      18       0      18       0   913M    104M
      1K    1K    98    93      98       0      98       0   914M    104M
    

Time passes&#8230; L2ARC warms up &#8230; After a while we start to see L2ARC Hits:

    
    read  hits  miss  hit%  l2read  l2hits  l2miss  l2hit%  arcsz  l2size
      3K    3K     0   100       0       0       0       0     1G      1G
      5K    5K     2    99       2       1       1      50     1G      1G
      2K    2K     0   100       0       0       0       0     1G      1G
      5K    5K     3    99       3       1       2      33     1G      1G
      4K    4K     0   100       0       0       0       0     1G      1G
      4K    4K     3    99       3       1       2      33     1G      1G
      5K    5K     3    99       3       1       2      33     1G      1G
      4K    4K     2    99       2       1       1      50     1G      1G
      5K    5K    17    99      17       4      13      23     1G      1G
    

More time passes&#8230; As the ARC churns, it&#8217;s hit ratio drops because it can&#8217;t cover the entire working set.  L2ARC accesses increase, as ARC misses are picked up by the L2ARC (becoming L2ARC hits):

    read  hits  miss  hit%  l2read  l2hits  l2miss  l2hit%  arcsz  l2size
      8K    8K    13    99      13      12       1      92     1G      3G
      7K    7K    18    99      18      12       6      66     1G      3G
      8K    8K    83    98      83      79       4      95     1G      3G
     768   182   586    23     586     532      54      90     1G      3G
      3K    2K   521    84     521     510      11      97     1G      3G
      8K    7K   620    92     620     619       1      99     1G      3G
      9K    9K   662    93     662     660       2      99     1G      3G
      4K    3K   732    82     732     732       0     100     1G      3G
      5K    5K   345    93     345      76     269      22     1G      3G
     13K   13K   496    96     496     372     124      75     1G      3G
      2K    1K   742    68     742     704      38      94     1G      3G
      2K    1K   780    62     780     757      23      97     1G      3G
      2K    1K   781    60     781     781       0     100     1G      3G
      2K    1K   714    70     714     709       5      99     1G      3G
      2K    1K   846    61     846     831      15      98     1G      3G
      2K    2K   781    73     781     767      14      98     1G      3G
      4K    3K   649    86     649     590      59      90     1G      3G
    

I&#8217;ve found this tool (used along with &#8216;iostat&#8217; or &#8216;zpool iostat -v&#8217;) to be a nice way to get a feel for how the ARC + L2ARC are performing. It will definitely give you a feel for how long the L2ARC takes to warm-up for your workload. The above examples were snipped from a timespan of 2 hours while compiling the [illumos gate](https://www.illumos.org/projects/illumos-gate).
