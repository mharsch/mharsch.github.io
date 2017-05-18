---
id: 99
title: visualizing filesystems with parallel sets
date: 2010-11-30T14:05:14+00:00
author: mharsch
layout: post
guid: http://blog.harschsystems.com/?p=99
permalink: /2010/11/30/visualizing-filesystems-with-parallel-sets/
twitter_cards_summary_img_size:
  - 'a:6:{i:0;i:851;i:1;i:251;i:2;i:3;i:3;s:24:"width="851" height="251"";s:4:"bits";i:8;s:4:"mime";s:9:"image/png";}'
categories:
  - Uncategorized
tags:
  - Visualization
---
Every so often, I run out of disk space. This inevitably leads me to take on a search-and-destroy mission: eliminate large files to free up space. Having spent some time recently studying data visualization techniques, I thought it would be an interesting exercise to try out different graphical tools to see which can best answer the question at hand: &#8220;Where are the files that are hogging my disk space?&#8221;.

I was very amused to find out that the (now common) data visualization known as a treemap was invented specifically to solve [this very problem](http://www.cs.umd.edu/hcil/treemap-history/). The treemap uses area to represent the numerical value of interest (in this case: disk space utilization). The filesystem hierarchy can also be encoded by the grouping and coloring of the subdivided rectangles, though this isn&#8217;t demonstrated well by the trivial example here. Consider a 3 level directory structure that contains files totaling 100MB. The structure is as follows:
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2011/01/treemap_example_hierarchy.png" alt="" title="treemap_example_hierarchy" width="851" height="251" class="aligncenter size-full wp-image-115" srcset="http://blog.harschsystems.com/wp-content/uploads/2011/01/treemap_example_hierarchy.png 851w, http://blog.harschsystems.com/wp-content/uploads/2011/01/treemap_example_hierarchy-300x88.png 300w, http://blog.harschsystems.com/wp-content/uploads/2011/01/treemap_example_hierarchy-768x227.png 768w" sizes="(max-width: 851px) 100vw, 851px" />](http://blog.harschsystems.com/wp-content/uploads/2011/01/treemap_example_hierarchy.png)
  
If the 100MB was evenly distributed across all the 3rd level directories, the treemap would look like this:
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/even_treemap.png" alt="" title="even_treemap" width="898" height="451" class="aligncenter size-full wp-image-101" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/even_treemap.png 898w, http://blog.harschsystems.com/wp-content/uploads/2010/11/even_treemap-300x151.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/even_treemap-768x386.png 768w" sizes="(max-width: 898px) 100vw, 898px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/even_treemap.png)
  
However, if the distribution was not uniform, the treemap will show us (using area) which directories use a disproportionate amount of space:
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/uneven_treemap.png" alt="" title="uneven_treemap" width="898" height="451" class="aligncenter size-full wp-image-102" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/uneven_treemap.png 898w, http://blog.harschsystems.com/wp-content/uploads/2010/11/uneven_treemap-300x151.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/uneven_treemap-768x386.png 768w" sizes="(max-width: 898px) 100vw, 898px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/uneven_treemap.png)
  
In the above graphic, we see that a3 is consuming 50% of the total area, while b2 and b3 consume 30% and 12% respectively. If this was a picture of my hard drive, I&#8217;d know to concentrate my file deletion efforts in directory a3 first, then b2 and so on.

**Parallel Sets**
  
An interesting data visualization technique called [Parallel Sets](http://kosara.net/papers/2010/Kosara_BeautifulVis_2010.pdf) offers (perhaps) a more flexible alternative to the treemap for representing the breakdown of a data set by categories. Let&#8217;s use the same directory example as above. A Parallel Sets view of the evenly distributed case might look something like this:
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets1.png" alt="" title="parsets1" width="864" height="523" class="aligncenter size-full wp-image-103" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets1.png 864w, http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets1-300x182.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets1-768x465.png 768w" sizes="(max-width: 864px) 100vw, 864px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets1.png)
  
This view shows higher level directories as an aggregation of their sub-directories &#8211; a nice effect that is less obvious in the treemap approach. The effect even more pronounced in the asymmetric example (using the same values as the treemap above):
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets2.png" alt="" title="parsets2" width="864" height="523" class="aligncenter size-full wp-image-104" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets2.png 864w, http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets2-300x182.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets2-768x465.png 768w" sizes="(max-width: 864px) 100vw, 864px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets2.png)
  
Looking at this view, I can clearly see that a3 and b2 are the low hanging fruit in my search for space hogs. We can eliminate the root level directory since it doesn&#8217;t add much value:
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets3.png" alt="" title="parsets3" width="864" height="523" class="aligncenter size-full wp-image-105" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets3.png 864w, http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets3-300x182.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets3-768x465.png 768w" sizes="(max-width: 864px) 100vw, 864px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/parsets3.png)
  
**Adding more dimensions**
  
At this point It&#8217;s probably a toss-up as to which approach is better. Both pictures answer the original question about which directories are contributing most to the space consumption problem. The treemap however is reaching it&#8217;s limits in terms of expressing more data, whereas the parallel sets are just getting warmed up. Let&#8217;s switch to a slightly more realistic directory tree example:
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2011/01/dir2.png" alt="" title="dir2" width="728" height="251" class="aligncenter size-full wp-image-114" srcset="http://blog.harschsystems.com/wp-content/uploads/2011/01/dir2.png 728w, http://blog.harschsystems.com/wp-content/uploads/2011/01/dir2-300x103.png 300w" sizes="(max-width: 728px) 100vw, 728px" />](http://blog.harschsystems.com/wp-content/uploads/2011/01/dir2.png)
  
Let us also change our original question and instead consider file count instead of file size as the unit of measure. The resulting pictures would look similar, except wider ribbons mean more files rather than more space consumed.
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types1.png" alt="" title="file_types1" width="920" height="481" class="aligncenter size-full wp-image-107" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types1.png 920w, http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types1-300x157.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types1-768x402.png 768w" sizes="(max-width: 920px) 100vw, 920px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types1.png)
  
Now let&#8217;s add the dimension &#8220;file type&#8221; where type {executable, data, text}.
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types2.png" alt="" title="file_types2" width="920" height="481" class="aligncenter size-full wp-image-108" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types2.png 920w, http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types2-300x157.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types2-768x402.png 768w" sizes="(max-width: 920px) 100vw, 920px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/file_types2.png)
  
Now we can see the data both converge and fan-out according to the extra property that we&#8217;ve specified. From the diagram, we can learn the following:

  * The contents of /usr/bin and /usr/lib are all executable, while /usr/include is all text
  * /var/tmp has text and data files, while /var/adm is all text
  * User &#8216;mharsch&#8217; has a bit of everything in his home directory, while user &#8216;root&#8217; has just text.

This same layering can be done with any attribute that is categorical (i.e. fits into a discrete number of buckets). Imagine breaking down all files by when they were last accessed (hour, day, week, month, year). Or bucketizing file size to see which directories contain many small files vs few large files.

Since the data is independent of the filesystem hierarchy, we don&#8217;t have to constrain ourselves to the tree view if it doesn&#8217;t make sense for the arbitrary question we want to ask. For example, if we want to know the relative breakdown of files owned by user &#8216;jsmith&#8217; vs &#8216;jdoe&#8217; broken down by file type and access history, we might end up looking at a picture like this:
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary1.png" alt="" title="arbitrary1" width="1153" height="608" class="aligncenter size-full wp-image-109" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary1.png 1153w, http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary1-300x158.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary1-768x405.png 768w, http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary1-1024x540.png 1024w" sizes="(max-width: 1153px) 100vw, 1153px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary1.png)
  
By altering the order of specified attributes, you can greatly affect the complexity of the resulting image. By choosing the variable &#8220;file type&#8221; followed by &#8220;last accessed&#8221; and finally &#8220;user&#8221;, we get a much busier (and less helpful) visual:
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary2.png" alt="" title="arbitrary2" width="1153" height="608" class="aligncenter size-full wp-image-110" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary2.png 1153w, http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary2-300x158.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary2-768x405.png 768w, http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary2-1024x540.png 1024w" sizes="(max-width: 1153px) 100vw, 1153px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/arbitrary2.png)
  
**Going beyond the filesystem**
  
Since computer systems are full of related, categorical data &#8211; why constrain ourselves to the filesystem? Let&#8217;s look at a measurement of cpu utilization over a certain interval. During the measurement, 2 processes were running: `cpuhog` and `failread`. Each process spent some of it&#8217;s time in user mode and some in kernel mode. If we wanted to get a graphical sense of how the cpu spent it&#8217;s time during the interval, we could use combinations of stacked time-series graphs (the approach used by [Analytics](http://wikis.sun.com/download/attachments/186238602/2010_Q3_ANALYTICS.pdf){.broken_link}):
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/analytics.png" alt="" title="analytics" width="794" height="847" class="aligncenter size-full wp-image-111" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/analytics.png 794w, http://blog.harschsystems.com/wp-content/uploads/2010/11/analytics-281x300.png 281w, http://blog.harschsystems.com/wp-content/uploads/2010/11/analytics-768x819.png 768w" sizes="(max-width: 794px) 100vw, 794px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/analytics.png)
  
Or we could apply our new tool Parallel Sets to the problem and get something like this:
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2010/11/cpu_proc.png" alt="" title="cpu_proc" width="1153" height="608" class="aligncenter size-full wp-image-112" srcset="http://blog.harschsystems.com/wp-content/uploads/2010/11/cpu_proc.png 1153w, http://blog.harschsystems.com/wp-content/uploads/2010/11/cpu_proc-300x158.png 300w, http://blog.harschsystems.com/wp-content/uploads/2010/11/cpu_proc-768x405.png 768w, http://blog.harschsystems.com/wp-content/uploads/2010/11/cpu_proc-1024x540.png 1024w" sizes="(max-width: 1153px) 100vw, 1153px" />](http://blog.harschsystems.com/wp-content/uploads/2010/11/cpu_proc.png)
  
Both visuals tell the story that `cpuhog` spends most of it&#8217;s time running in user mode, while `failread` is spending most if it&#8217;s time in kernel mode. The Analytics version puts the data in the context of a timeline, while the parallel sets image does not. For the purposes of spotting trends over time, the Analytics version would be the way to go. If a historical context wasn&#8217;t important (say, for a point-in-time snapshot) the parallel sets view may be a stronger choice, since it combines the data into one graph and uses slightly less ink.

**Conclusion**
  
I think Parallel Sets has a future in visualizing certain aspects of computer systems. [The tools available now](http://eagereyes.org/parallel-sets) are primitive (only accepting CSV data, and not allowing empty fields), but I think the tool is powerful enough that developers will be compelled to integrate it into their GUIs.

Update (1/1/2011): I just realized that the filesystem hierarchy diagrams (which were png files created using GraphVis) were not showing up on IE or Firefox. I&#8217;ve fixed these files, and they should work on all browsers now. If you read this post before using a browser other than Safari, you missed out on 2 diagrams. My apologies.