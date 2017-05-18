---
id: 123
title: remote cpu monitoring using node-kstat
date: 2011-05-03T15:20:03+00:00
author: mharsch
layout: post
guid: http://blog.harschsystems.com/?p=123
permalink: /2011/05/03/remote-cpu-monitoring-using-node-kstat/
categories:
  - Uncategorized
---
The [node-kstat add-on module](https://github.com/bcantrill/node-kstat) for node.js allows you to poll specific system kstats from within node, and get back a nice array of objects with keys corresponding to the various kstat fields (like what you get from the kstat(1) command). Bryan&#8217;s repository also includes an [example of the mpstat(1) command](https://github.com/bcantrill/node-kstat/blob/master/examples/mpstat.js) written in JavaScript that utilizes the node-kstat add-on. Given the ease with which you can implement web services using node (and supporting libraries), I thought I&#8217;d try exposing the kstat facility as a RESTful web service, and re-tool the mpstat example to run in the browser as a client.

**Part 1: Exposing a kstat web service**

Thanks to the [express framework](http://expressjs.com/) for node, this turned out to be the easiest part of the exercise. I wanted to expose roughly the same API as the node-kstat Reader() constructor method (which takes kstat class, module, name, and instance &#8211; all optional), but I also wanted to allow multiple kstats to be specified (within a single module). The reason for this is that mpstat(1) consumes 2 kstats (cpu::sys and cpu::vm), and I didn&#8217;t want to have to make multiple HTTP requests for each round of output from the mpstat client. I settled on the following API for my web service:
	  
`http://server.example.com:port/kstats/module/instance/name`

The &#8216;module&#8217; field is required and must match a single kstat module. The &#8216;`instance`&#8216; field may be an integer of a single instance, or the &#8216;*&#8217; wildcard to indicate all instances. The &#8216;`name`&#8216; field may be one or more kstat names separated by semicolons. So, the URI needed to gather the 2 kstats consumed by mpstat ends up being:
	  
`http://server.example.com:port/kstats/cpu/*/vm;sys`

The web service responds with JSON. The response contains an object for each of the named kstats (vm and sys), each of which contains an array with one element per CPU instance.

Here&#8217;s all the code for the kstat web service:

    
    var express = require('express');
    var kstat = require('kstat');
    
    var app = module.exports = express.createServer();
    
    // Routes
    
    app.get('/kstats/:module/:instance/:name', function(req, res){
    
            var filter = {};
    
            filter["module"] = req.params.module;
    
            if (req.params.instance ==! '*')
                    filter["instance"] = parseInt(req.params.instance, 10);
    
            //handle multiple semicolon-separated values for stat 'name'
            var stats = req.params.name.split(';');
    
            var results = {};
    
            var reader = {};
    
            for (var i=0; i < stats.length; i++) {
                    var stat = stats[i];
    
                    filter["name"] = stat;
    
                    reader[stat] = new kstat.Reader(filter);
    
                    results[stat] = reader[stat].read();
            }
    
            // Set response header to enable cross-site requests
            res.header('Access-Control-Allow-Origin', '*');
    
            res.send(results);
    
    });
    
    // Only listen on $ node app.js
    
    if (!module.parent) {
      app.listen(3000);
      console.log("kstat server listening on port %d", app.address().port);
    }
    

**Part 2: porting mpstat.js to the browser**

I really didn&#8217;t have to change much to the original server-side mpstat.js to get it working in the browser as a client to the kstat web service. The big change was to make it event-driven so I could use the Ajax model for making HTTP requests and responding to HTTP response events. To do this, I added an update_kstats() function for the HTTP response handler to call. Rather than having the mpstat object push output, I added a method to retrieve the current output from the object&#8217;s internal state.

The modified mpstat.js code (renamed mpstat_init.js) can be viewed [here](http://pastebin.com/raw.php?i=TCV28Tak).

The Ajax page that includes the above mpstat_init.js can be viewed [here](http://pastebin.com/raw.php?i=Kfu5SE9L).

[<img src="http://blog.harschsystems.com/wp-content/uploads/2011/05/mpstat_command.png?w=287&h=300" alt="" title="mpstat_command" width="287" height="300" class="aligncenter size-medium wp-image-125" srcset="http://blog.harschsystems.com/wp-content/uploads/2011/05/mpstat_command.png 685w, http://blog.harschsystems.com/wp-content/uploads/2011/05/mpstat_command-288x300.png 288w" sizes="(max-width: 287px) 100vw, 287px" />](http://blog.harschsystems.com/wp-content/uploads/2011/05/mpstat_command.png)

(Edit: Live Demo no longer available)

**Part 3: adding some flare**

If you&#8217;re like most people, you like [using your visual cortex](http://dtrace.org/resources/bmc/cec_analytics.pdf) to process data. Since we are running JavaScript in the browser anyway, why not take a page from the [cloud analytics](http://dtrace.org/blogs/dap/2011/03/01/welcome-to-cloud-analytics/) book, and use [flot](http://code.google.com/p/flot/) to graph some of the data we&#8217;re getting from mpstat? In the example below, I&#8217;m graphing individual CPU utilization (usr + sys columns) per-cpu, and I&#8217;m color-coding the mpstat output with the graph. The only change needed to the mpstat module was to add a method for retrieving the current output state in object form rather than lines of text (so it could be fed to the graphing library).
  
[<img src="http://blog.harschsystems.com/wp-content/uploads/2011/05/mpstat_graph.png?w=216&h=300" alt="" title="mpstat_graph" width="216" height="300" class="aligncenter size-medium wp-image-124" />](http://blog.harschsystems.com/wp-content/uploads/2011/05/mpstat_graph.png)

Source for the Ajax page (which uses the flot library) can be found [here](http://pastebin.com/raw.php?i=mrymU5k9).

(Edit: Live Demo no longer available)
  
(Edit: 3/23/13 Source for the demo is available in a self-contained express app hosted here: <https://github.com/mharsch/mpstat-browser>.)