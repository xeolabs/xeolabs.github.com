---
layout: post
title: SceneJS Pick-Fly-Orbit Camera
description: "Camera node which flies to whatever you pick and orbits it"
modified: 2013-05-31
category: articles
comments: true
tags: [scenejs, tutorial, picking, camera]
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

The folks over at [BIMSurfer.org](http://bimsurfer.org) needed a
 camera that flies to and orbits whatever you click on, so I made a plugin.<br><br>Try a couple of demos:

[![]({{ site.url }}/images/scenejs/pickFlyOrbitCity.png)](http://scenejs.org/examples/pages/demos/pickFlyOrbitCity.html) | [![]({{ site.url }}/images/scenejs/pickFlyOrbitTerrain.png)](http://scenejs.org/examples/index.html#cameras_pickFlyOrbit_terrain)
----|----
[Click to run](http://scenejs.org/examples/pages/demos/pickFlyOrbitCity.html) | [Click to run](http://scenejs.org/examples/pages/demos/pickFlyOrbitTerrain.html)

### Usage
The plugin provides a new scene node type and gets loaded as you need it. So assuming that you've
pointed SceneJS to where you keep your plugins:

{% highlight javascript %}
SceneJS.setConfigs({
    pluginPath: "./foo/myPluginsDir"
});
{% endhighlight %}

Then just wrap the node around your scene:

{% highlight javascript %}
SceneJS.createScene({
    nodes:[
        {
            type:"cameras/pickFlyOrbit",
            look:{ x: 0, y:150, z: 0 },
            yaw:-40,
            pitch:-20,
            zoom:800,
            zoomSensitivity:-20.0,
            showCursor: true,
            cursorSize:6.0,

            nodes:[
                {
                    type:"objects/buildings/city",
                    width:600
                }
            ]
        }
    ]
});
{% endhighlight %}

And then everything's pick-fly-orbitable.<br><br>The node's properties are:

* ```look``` - initial point-of-interest (POI) in World-coordinates
* ```yaw```, ```pitch``` - initial degrees of rotation of eye position about POI on Y and X axis, respectively
* ```zoom``` - initial distance of eye from POI
* ```zoomSensitivity``` - sensivity of mouse wheel when updating zoom. The sign of this determines which direction the wheel drives the zoom.
* ```showCursor``` - set true to show the POI with a sphere
* ```cursorSize``` - radius of the cursor sphere

### Making scene objects unpickable

See how you can't pick the sky in the live demos above? That's a good thing, because otherwise you'd be able to orbit
a point on the skybox, which would be rather a Truman Show moment!<br><br>Learn how to make objects unpickable in
 [this API example](http://scenejs.org/examples.html?page=disablePicking&showCode=true).

### Read More
* [Picking in SceneJS]({{site.url}}/articles/scenejs-picking)
* [Fast Ray Picking in SceneJS]({{site.url}}/articles/scenejs-ray-picking)
* [Creating your own Scene Node Types](/articles/scenejs-node-types)