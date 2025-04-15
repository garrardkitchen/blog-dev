---
title: "How to run the Apache Ignite Agent with an Ignite.NET Core Server Node"
date: 2020-07-28T13:48:46+01:00
draft: false
tags: ["apache ignite", "ignite", "gridgain", "data grid", "Ignite.NET"]
---

I've recently been researching into [Apache Ignite](https://apacheignite-net.readme.io/docs/getting-started).  Apache Ignite is an in-memory, memory-centric, distributed database, caching and processing platform for transactional, analytical, and streaming workloads.  

So why the post?  Well, with using .NET Core, I have run into one or two challenges that I have had to work through.  One of which involves the Agent.  I feel it is important to share with you how I get beyond this issue.  It may save you a lot of time if you're an Apache Ignite noob like me.  

You use the Agent when you want to execute queries, SQL DML & DDL amongst other actions, from within the Web Console app.  The Agent acts as a proxy.  The Agent must connect to both the Web Console and your Server node or Thick Client node.

As I say above, this Agent acts as a proxy between the Web Console (UI to configure clusters, execute SQL DML & DDL, and query KV stores including visuals on caches etc…) and a Server node (aka, `data node`) to execute SQL & KV stores.

With all previous efforts, I was not able connect the Agent to the data node when the data node was created using a .NET runtime.  When running the same configuration but using Java instead, it would connect without issue.  Based on being able to connect to `data node` when created via java, I was confident that I would find a way to get this to work.

After many hours of twawling the internet for answers and failed attempts, I figured out what the issue was.  The `Apache.Ignite` nuget package does not include the `ignite-rest-http` module (contains many Jars), and this is what the Agent needs to communicate with the data node.  So, what you need to do is download the entire Ignite binary package, and extract the ignite-rest-http folder.  Then you use the following code to add a list of comma-separated file names of the HTTP Jar files to the IgniteConfiguration.JvmClasspath property:

```csharp
var cfg = new IgniteConfiguration
{
   JvmClasspath = Directory.GetFiles(pathToIgniteRestHttpJars)
      .Aggregate((x, y) => x + ";" + y)
};

using (var ignite = Ignition.Start(cfg))
{
...
```

From what I could find, there's no plans on including the ignite-rest-http module in the Apache.Ignite nuget package.

I will share a GitHub repo to ease you into this shortly.