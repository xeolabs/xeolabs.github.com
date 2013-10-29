---
layout: post
title: What's new in SceneJS 3.2
description: "New file import formats, frustum culling, pick-zoom-rotate camera, more functionality for custom nodes and more "
modified: 2013-05-31
category: articles
comments: true
tags: [scenejs, release]
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

### Summary

[SceneJS](http://scenejs.org) V3.2 adds a bunch of new node types for visibility and detail culling, plus import from OBJ, DX2 and MD2
[custom node plugins](https://github.com/xeolabs/scenejs#custom-node-types).

### Visibility and Detail Culling

The view frustum is the pyramid-shaped area of view that extends in front of your eye, and visibility culling is an
 optimization technique in which elements of the scene are only rendered when they intersect that area. SceneJS now
 supports that using multiple worker threads, which is wickedly fast. It also supports detail
 culling, where you can provide multiple detail versions of scene objects, so the appropriate level of detail is
 rendered for the object's size on the canvas. The idea is that you don't render more detail than will actually be shown
 on the canvas; if a city was in the distance, for example, then you might want to just render a crude outline instead of
 every window and streetlamp.

 * [Read the tutorial]({{ site.url}}/articles/scenejs-detail-culling)
<br><br>

[![]({{ site.url }}/images/scenejs/frustumCulling.jpg)](http://scenejs.org/examples.html?page=frustumDetailCulling) | [![]({{ site.url }}/images/scenejs/visibilityCulling.png)](http://scenejs.org/examples.html?page=frustumVisibilityCulling)
----|----
[Detail culling demo](http://scenejs.org/examples.html?page=frustumDetailCulling) | [Visibility culling demo](http://scenejs.org/examples.html?page=frustumVisibilityCulling)


### Import meshes from .OBJ, .MD2 and .3DS

**OBJ** is an lightweight geometry definition file format first developed by Wavefront Technologies which contains mesh
 data — namely, the position of each vertex, the UV position of each texture coordinate vertex, normals, and the faces
 that make each polygon defined as a list of vertices, and texture vertices.
 **MD2** is a model format used by id Software's id Tech 2 engine in games such as Quake II and Soldier of Fortune.
The format is primarily used for animated player models and uses keyframes for mesh positions, which SceneJS interpolates
 within a vertex shader to create smooth animation.
 **3DS** is one of the file formats used by the Autodesk 3ds Max 3D modeling, animation and rendering software, and has
become a de facto industry standard for transferring models between 3D programs, or for storing models for 3D resource catalogs.
<br><br>
SceneJS imports these formats using the [K3D.JS](http://k3d.ivank.net/) library, which is very fast because it loads the
files into typed arrays, which it then parses.
<br><br>
Importing geometry from these formats is as easy as:
{% highlight javascript %}
SceneJS.createScene({
    nodes:[
        {
            type:"translate", y:-30, z:-200,

            nodes:[
                {
                    type:"texture",
                    layers:[
                        {
                            src:"foo/bar/raptor.jpg"
                        }
                    ],
                    nodes:[

                        // Import Wavefront .OBJ mesh
                        {
                            type:"import/obj",
                            src:"foo/bar/raptor.obj"
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

Try some demos:
<br>

[![]({{ site.url }}/images/scenejs/raptorOBJ.png)](http://scenejs.org/examples.html?page=importObj) | [![]({{ site.url }}/images/scenejs/lionMD5.png)](http://scenejs.org/examples.html?page=importMd2) | [![]({{ site.url }}/images/scenejs/lexus3DS.png)](http://scenejs.org/examples.html?page=import3ds)
----|----|----
[.OBJ import demo](http://scenejs.org/examples.html?page=importObj) | [.MD2 import demo](http://scenejs.org/examples.html?page=importMd2) | [.3DS import demo](http://scenejs.org/examples.html?page=import3ds)


### Pick-Fly-Orbit Camera

The folks over at [BIMSurfer.org](http://bimsurfer.org) are busy porting their platform over to SceneJS 3.X and needed a
 camera that zooms to where you click then orbits that point, so I made a plugin. This is still in beta, but working pretty
    well so far. You can pick and orbit anything in the rendered view - SceneJS

{% highlight javascript %}
SceneJS.createScene({
    nodes:[
        {
            type:"cameras/pickFlyOrbit",
            yaw:-40,
            pitch:-20,
            maxPitch:-10,
            minPitch:-80,
            zoom:800,
            eye:{ x: 0, y:150, z: -1000 },
            look:{ x: 0, y:150, z: 0 },
            zoomSensitivity:20.0,
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


[![]({{ site.url }}/images/scenejs/pickFlyOrbitCity.png)](http://scenejs.org/examples/pages/demos/pickFlyOrbitCity.html) | [![]({{ site.url }}/images/scenejs/pickFlyOrbitTerrain.png)](http://scenejs.org/examples/pages/demos/pickFlyOrbitTerrain.html)
----|----
[Pick-fly-orbit demo #1](http://scenejs.org/examples/pages/demos/pickFlyOrbitCity.html) | [Pick-fly-orbit demo #2](http://scenejs.org/examples/pages/demos/pickFlyOrbitTerrain.html)


### More Features

* Nodes can signal task progress (eg. data loading) - [demo](http://scenejs.org/examples.html?page=customNodeDefWithTasks)
* Nodes can publish data through the API - [demo](http://scenejs.org/examples.html?page=customNodeDefWithPublications)
* Built-in RequireJS so that nodes can load 3rd-party dependencies - [demo](http://scenejs.org/examples.html?page=customNodeDefWithRequire)

