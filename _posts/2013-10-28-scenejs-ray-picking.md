---
layout: post
title: Fast Ray Picking in SceneJS
description: "How to do a stupidly fast ray-pick on anything in the rendered view"
modified: 2013-05-31
category: articles
comments: false
tags: [scenejs, tutorial, picking, interaction]
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

SceneJS ray-picking lets you pick a point on the canves and get the precise 3D World-space coordinates of whatever that
intersects within the 3D view.
<br><br>
It's very fast, even for really large numbers of objects. While it uses the standard [colour coding approach](http://antongerdelan.net/opengl/colourpicking.html) to find the picked
object, it also uses the depth buffer to help it find the 3D intersection coordinates, thus managing to avoid expensive
ray-intersect computations. If you're curious how it works, I did [a writeup on the technique here](/articles/scenejs-ray-picking-technique).

[![]({{ site.url }}/images/scenejs/rayPicking.jpg)](http://scenejs.org/examples.html?page=rayPicking) | [![]({{ site.url }}/images/scenejs/pickFlyOrbitCity.png)](http://scenejs.org/examples/pages/demos/pickFlyOrbitCity.html) | [![]({{ site.url }}/images/scenejs/pickFlyOrbitTerrain.png)](http://scenejs.org/examples/pages/demos/pickFlyOrbitTerrain.html)
----|----
[Ray pick performance demo](http://scenejs.org/examples.html?page=rayPicking) | [Pick-fly-orbit demo #1](http://scenejs.org/examples/pages/demos/pickFlyOrbitCity.html)  | [Pick-fly-orbit demo #2](http://scenejs.org/examples/pages/demos/pickFlyOrbitTerrain.html)

## Usage

I'll assume that you know all about [basic 2D picking]({{site.url}}/articles/scenejs-picking) in SceneJS. Ray picking
is done in pretty much the same way, except that you supply an additional ```rayPick``` flag when you do the pick, and get
back an additional ```worldPos``` property in the result, which gives you the intersection coordinates:
{% highlight javascript %}
// Register a pick handler
myScene.on("pick",
    function (hit) {

        // The 'name' property on the first name node on the path
        // from the picked teapot up to the scene root, eg "blueTeapot"
        var name = hit.name;

        // Name path, a '.' delimited concatenation of the 'name' properties
        // of all name nodes on the path down from teh scene root to the
        // picked teapot.
        var path = hit.path;

        // Two-element array containing the X,Y canvas coordinates at which we picked
        var canvasPos = hit.canvasPos;

        // Three-element array containing the X,Y,Z World-space coordinates at which we picked
        var worldPos = hit.worldPos;
    });
{% endhighlight %}

And here's the ray-pick call:
{% highlight javascript %}
myScene.pick(45, 150, { rayPick: true });  // Note the rayPick flag
{% endhighlight %}


