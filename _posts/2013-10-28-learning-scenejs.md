---
layout: post
title: SceneJS Tutorials
description: "3D web programming using SceneJS"
tagline: "3D web programming using SceneJS"
thumbnail: scenejs/reflectionLayered.jpg
modified: 2013-05-31
category: articles

tags: [scenejs, tutorial, picking]
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
<br><br>
[![]({{ site.url }}/images/scenejs/pickFlyOrbitTerrainSlim.jpg)](http://scenejs.org/examples/index.html#cameras_pickFlyOrbit_terrain)

## Introduction
[SceneJS](http://scenejs.org) is an extensible WebGL-based engine for doing high-performance 3D visualisation in the browser,
without using plugins. Behind its friendly API, it's optimised to death with serious trickery like
scene compilation and state sorting, so will happily scale up to thousands of objects while eating draw calls for breakfast.
You can also extend it with your own scene node types, which is some powerful extensibility.
<br><br>
This is where I'll post tutorials to show you how to use SceneJS to do common tasks. Use them as a
companion to the [live API demos](http://scenejs.org/examples.html).
<br><br>
Before we begin, do be sure that your browser [supports WebGL](http://get.webgl.org/).
<br><br>
Have fun!

### Creating 3D Scenes
* [Quick Start](/articles/scenejs-quick-start)
* [Adding and Removing Nodes](/articles/scenejs-creating-a-scene-and-adding-nodes)

### Interaction
* [Picking](/articles/scenejs-picking)
* [Ray Picking](/articles/scenejs-ray-picking)
* [Pick-Fly-Orbit Camera](/articles/scenejs-pickflyorbit-camera)

### Effects
* [Transparency](/articles/scenejs-transparency)
* [Reflections](/articles/scenejs-reflection)

### Texturing
* [Alpha Mapping](/articles/scenejs-alpha-mapping)

### Importing
* [Importing .OBJ, .MD2 and .3DS](/articles/scenejs-obj-md2-3ds)

### Physics
* [Rigid-Body Physics](/articles/scenejs-physics)

### Optimisation
* [Visibility and Detail Culling](/articles/scenejs-detail-culling)

### Extending SceneJS
* [Creating your own Scene Node Types](/articles/scenejs-node-types)

### Appendix
* [Pssst...How SceneJS Does Fast Ray Picking](/articles/scenejs-ray-picking-technique)
* [Automatic Recovery from Lost WebGL Context](/articles/scenejs-webgl-context-lost)
