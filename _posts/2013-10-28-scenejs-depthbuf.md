---
layout: post
title: Using the SceneJS Depth Buffer
description: "Using the depthbuf node to configure the depth buffer for subgraphs"
modified: 2013-05-31
category: articles
comments: true
tags: [scenejs, tutorial, depthbuf]
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

In this tutorial we'll check out the SceneJS ```depthbuf``` node, which configures the depth buffer for the
 nodes in its subgraph.

## Depth what?

When an object is rendered, the Z-depth of each pixel is stored in the *depth buffer*, which is a two-dimensional
array (x-y) with one element for each pixel. If another object is to be rendered at the same pixel, the buffer
compares the two depths and overrides the current pixel if the new object is closer to the viewpoint. The chosen
depth is then saved to the buffer, replacing the old one. In short, the buffer ensures corrrect depth perception,
where a close object hides a farther one.

* [Wikipedia article](http://en.wikipedia.org/wiki/Z-buffering)
* [Learning WebGL](http://learningwebgl.com/blog/?p=859)

## Example

Let's create a teapot wrapped in a ```depthbuf``` that has default attribute values:
{% highlight javascript %}
 // Create scene
 var scene = SceneJS.createScene({
     nodes: [

         // Mouse-orbited camera, implemented by plugin at
         // http://scenejs.org/api/latest/plugins/node/cameras/orbit.js
         {
             type: "cameras/orbit",
             yaw: 30,
             pitch: -30,
             zoom: 10,
             zoomSensitivity: 0.1,

             nodes: [{
                 type: "material",
                 color: {
                     r: 0.6,
                     g: 0.6,
                     b: 0.9
                 },
                 nodes: [

                     // Our depthbuf node, with an ID so we can find it in the scene
                     {
                         type: "depthbuf",
                         id: "myDepthbuf",
                         enabled: true, // Default
                         clearDepth: 1.0, // Default - clamped to [0..1]

                         // Default - also "equal","lequal","greater","notequal" and "gequal"
                         depthFunc: "less",

                         nodes: [

                             // Teapot primitive, implemented by plugin at
                             // http://scenejs.org/api/latest/plugins/node/prims/teapot.js
                             {
                                 type: "prims/teapot"
                             }
                         ]
                     }
                 ]
             }]
         }
     ]
 });
{% endhighlight %}

And there's your teapot, with depth buffer working with default values:
<br><br>
![depthbuf Defaults]({{ site.url }}/images/scenejs/depthbufDefaults.png)
<br><br>

### Setting clearDepth

Now let's get that ```depthbuf``` node and set its ```clearDepth``` to a point in normalised View-space that
cuts through the teapot:

{% highlight javascript %}
scene.getNode("myDepthbuf",
    function (myDepthbuf) {
        myDepthbuf.setClearDepth(.989);
    });
{% endhighlight %}

Now all pixels
<br><br>
![depthbuf Defaults]({{ site.url }}/images/scenejs/depthbufEnabledWithClearDepth.png)
<br><br>

### Setting depthFunc

The ```depthFunc``` attribute decides how each in
Now let's get that ```depthbuf``` node and set its ```clearDepth``` to a point in normalised View-space that
cuts through the teapot:

{% highlight javascript %}
scene.getNode("myDepthbuf",
    function (myDepthbuf) {
        myDepthbuf.setClearDepth(.989);
    });
{% endhighlight %}

Now all pixels
<br><br>
![depthbuf Defaults]({{ site.url }}/images/scenejs/depthbufEnabledWithClearDepth.png)
<br><br>
