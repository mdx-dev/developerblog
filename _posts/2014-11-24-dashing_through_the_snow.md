---
layout: post
author: toddman0430
title: "Dashing through the snow."
description: "Exceptionally Handsome and Interactive."
tags: [dashing.io, sinatra, hotness, gridster,  batman.js, newrelic, rufus scheduler]
comments: true
---

[Dashing](http://dashing.io/) is a Sinatra-based framework that facilitates rapid development of dashboards with pre-configured 'widgets' which display metrics that are virtually real-time thanks to 
the Rufus-scheduler and Batman.js bindings. The out-of-the-box controls have a polished look and feel
and include gauges, lists, graphs and more. The dead-simple API makes creating custom widgets virtually painless. Dashing also leverages Gridster.js, which allows the user to rearrange the orientation of widgets and persist that placement per user. As it's been open-sourced by Shopify on Github, there are a number of custom widgets that contributors have generously created that consume the NewRelic and SideKiq API's to mention a couple. Data is retrieved by worker processes at a pre determined interval and is bound (using batman) to each widget template.

### A Dashing Challenge
My task with Dashing morphed from a simple representation of the status of each of our client systems (there are many) to the ability to drill down into more granular metrics that would not just be client-specific but also specific to the multiple environments per each (e.g., Production, Stage, UAT). What this would necessitate is a click-event handling on each widget of a 'parent' dashboard to one that is client-specific. My searches did not uncover a similar use case, therefore I needed to perform a deeper dive into the proverbial plumbing of the framework to determine what approach would be most appropriate give the requirement.

### The Options
At first glance one might want to add a route to handle params that would indicate which client data should be rendered in the drilldown (or child) dashboard. The primary issue was ensuring that only client-specific data was sent from the job workers to the UI. These workers use Rufus-scheduler, which is an in-process, in-memory scheduler. Quite simply, these run independent of the routing mechanism of Sinatra. One option to simulate session state was to use an in-memory persistence layer such as redis with namespaced keys that could be accessed by each worker. Another would be mass querying for all client (and environment) data and inside the iteration per client (and environment) bind to a client-specific data-id in the .erb. Although the latter implies a large dataset to filter through, the fact that the scheduled job intervals more than accommodated processing outweighed the additional prospective point of failure that redis introduced. As the dashboard scales (more clients added), optimization and therefore a true persistence layer would be revisited.

### The Implementation
To ensure that the user is able to 'drill into' a particular client dashboard by selecting a widget we needed a handler. I first set the default dashboard in the config.ru as follows:
```ruby
set :default_dashboard, 'summary'
```
-inside the configure block.
I then created a collection of list items for the client environment widgets, each having a data-link attribute that will pass in a query string param for each client. Wiring up an event handler was then very straightforward:
```javascript
$(function () {
  $('li').live('click', function(e){
    var widget = $(this).find('.widget');
      var url = widget[0].attributes[1].value
      var win = window.open(url, '_blank');
    win.focus();   
  });
});
```
Keeping with DRY principles and non-intensive addition of future clients, I added scripting to append the client name to the data-id of each widget, thereby ensuring that the job could target it with the appropriate data:
```javascript
  $.urlParam = function(name, url) {
    if (!url) {
     url = window.location.href;
    }
    var results = new RegExp('[\\?&]' + name + '=([^&#]*)').exec(url);
    if (!results) { 
        return undefined;
    }
    return results[1] || undefined;
  }

    $("div").each(function () {
      var originalSrc = $(this).attr('data-id');
      if (originalSrc) {
        $(this).attr('data-id', originalSrc.concat("_" + (decodeURIComponent($.urlParam('client', window.location.href)))));  
      }  
    });
```
Data retrieval in the job now becomes a matter of iterating through those clients and updating the 
send_event method as follows:
```ruby
send_event("etl_last_run_#{client}", { items: items }) 
```

#### Conclusion
Dashing provides an efficient means of fashioning attractive and lightweight dashboards. Its current architecture allows for extension and customization; however to accommodate a heirarchical model with multiple dashboards sharing state may be a little elusive. Whichever option you choose should be predicated on the amount of data retrieved and weighed against such factors as number of consumers, scheduled interval and responsiveness of data endpoints. The persistence layer possibility may come to fruition in our case, and will then be the subject of another blog post. Happy Holidays!

