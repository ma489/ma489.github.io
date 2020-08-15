---
title: Improving Listify
date: 2016-10-16T12:24:00+00:00
layout: single
permalink: /2016/10/improving-listify/
classes: wide
---
<span style="font-weight: 400;"><a href="http://www.mansourahmed.com/2016/06/spotify-playlister/">Listify&#8217;</a>s latency, while tolerable, could do with some improvement (p<span style="color: #000000;">erformance was a secondary concern while building it; I felt that the application functionality was far more interesting). &nbsp;So I’ve decided to address this issue and blog about it as I go along.&nbsp;</span></span>

<span style="font-weight: 400;"><span style="color: #000000;">Quick recap: Listify&nbsp;retrieves BBC&nbsp;radio schedule information in order to create Spotify playlists (aside: unfortunately the BBC don’t provide a clean API to retrieve this information, so there’s a fair bit of scraping going on). In order to&nbsp;retrieve that information, the application needs to send requests&nbsp;to the BBC Radio&nbsp;web site.</span>

![High-level architecture](/assets/img/Improving-Listify-I1-e1476537639411.jpg)

<span style="font-weight: 400;">So, for instance, to create a Spotify playlist, the application must retrieve the radio tracklist for that particular episode of that particular show. This involves making a relatively-expensive external network request to grab that information. Another similar request is then made to create the Spotify playlist.&nbsp;</span><span style="font-weight: 400;">Fortunately the retrieved&nbsp;information is cached in Listify’s local database, so that if the playlist for this episode is requested again, the information is instead retrieved more quickly from the database. But of course,&nbsp;the&nbsp;first user after&nbsp;that playlist still has to take that initial latency hit! Let&#8217;s fix this&#8230;.</span>

<span style="font-weight: 400;">Here are some sample numbers from a basic test, using <a href="https://en.wikipedia.org/wiki/ApacheBench">Apache Benchmark</a>. I&#8217;ve got a (<a href="https://en.wikipedia.org/wiki/Docker_(software)">Docker</a>) containerised version of the web app running locally, so these numbers are more useful because they&nbsp;exclude the network time from a user machine to the Listify server. Here I&#8217;m hitting the show list for Radio 1, 100 times serially.</span>

{% highlight shell %}
$ ab -n 100 localhost/shows/radio1
...
Percentage of the requests served within a certain time (ms)
50%    34
95%    49
100%   1249 (longest request)
{% endhighlight %}

<span style="font-weight: 400;">Now, generally the&nbsp;response time isn’t so bad, but as you can see the longest response time sucks &#8211; these are the times for the first visitor to that page. Sure, that cost is amortized over the subsequent&nbsp;requests, but it’s still&nbsp;an unpleasant experience for that first user!</span>

<span style="font-weight: 400;">One&nbsp;solution to this is to <a href="http://paweljaniak.co.za/2014/01/07/memcached-lessons/#cache-warming">pre-warm the cache</a>. That is, have an automated process run ahead of time doing the job of the first visitor, so that no real human visitors have to endure that latency. My, rather simple, approach is to have a <a href="https://en.wikipedia.org/wiki/Cron">cron</a> job run nightly that uses <a href="https://en.wikipedia.org/wiki/Wget">wget</a> to&nbsp;recursively crawl the site and hit each page.</span>

{% highlight shell %}
wget -r --delete-after localhost/stations
{% endhighlight %}

<span style="font-weight: 400;">This is the basic form of the command, but <a href="https://www.gnu.org/software/wget/manual/html_node/Recursive-Retrieval-Options.html">there are additional options</a> (e.g. ignore <a href="https://en.wikipedia.org/wiki/Robots_exclusion_standard">robots.txt</a>, specifying a max recursive depth, waiting in between requests, ignore certain pages, etc.).&nbsp;</span>

<span style="font-weight: 400;">And now, with a warm cache, here are the&nbsp;new numbers:</span>

{% highlight shell %}
$ ab -n 100 localhost/shows/radio1
...
Percentage of the requests served within a certain time (ms)
50%     33
95%     47
100%    91 (longest request)
{% endhighlight %}

_<span style="font-weight: 400;">A big improvement!</span>_

<span style="font-weight: 400;">But there are, of course, some downsides: </span>

  1. <span style="font-weight: 400;">The system may waste time and disk space caching content that may not be used. This is mitigated by only caching the info for the <a href="https://en.wikipedia.org/wiki/List_of_most-listened-to_radio_programs#Current_top_stations_in_the_United_Kingdom">most popular stations</a>. </span>
  2. <span style="font-weight: 400;">Since this cache is only warmed nightly, despite new content continuously being&nbsp;made available throughout the day, some newer content may result in a cache miss. I think this is tolerable because the cache misses now happen less often. </span>
  3. <span style="font-weight: 400;">A</span>t the moment, this cache initialisation is a pretty resource-intensive task &#8211; essentially, deliberately&nbsp;inducing peak time usage. This is OK for now, since the cache warming happens during a quiet period at night, so has minimal impact on other users of the system.

That&#8217;s all for now. More to come!