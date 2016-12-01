---
layout: post
title: xeogl in BIMSurfer
description: "Supercharging your BIM"
tagline: "Supercharging your BIM"
modified: 2016-19-09
category: articles
comments: true
tags: [xeogl, bimsurfer, webgl]
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

Last week I integrated the latest build of [xeogl](http://xeogl.org) into [BIMSurfer](http://bimsurfer.org).<br><br>xeogl is an 
 open source WebGL-based 3D visualization engine I've been working on, and BIMSurfer is an open source tool 
 for Web-based visualization and evaluation of Building Information Models (BIM). Here's my first blog post on these adventures.
 <br>

<iframe width="560" height="315" src="https://www.youtube.com/embed/tCHwEA2HqU8" frameborder="0" allowfullscreen></iframe>

## History

 First, a little history. BIMSurfer's WebGL-based 3D viewer has been through a few evolutions, so it's been an tag-team effort in which each evolution 
 has built upon lessons learned from the previous one. 
 <br><br>
 [Rehno Lindeque](https://twitter.com/RehnoLindeque) kicked things off 
 with the first version in 2011, which he implemented as a facade object wrapping a [SceneJS](http://scenejs.org) scene graph.
  [Robert Zach](robert.zach@tuwien.ac.at), [Kaltenriner Christoph](mailchriska@gmail.com) 
 and [Leichtfried Michael](leichtfried.michael@gmail.com) from TU Vienna then added more enhancements in 2012. 
 <br><br>
 [Bryan de Nijs](https://twitter.com/bryandenijs) wrote the next version in 2014, again on SceneJS, but this time as a 
 collection of pluggable components that wrapped portions of the underlying SceneJS scene graph.
 <br><br>
 Bryan handed the viewer over to me in 2015. Looking at Bryan's design, I realised that, for applications like BIMSurfer, 
  the underlying WebGL engine should ideally have an object-component-based API. And so this is how xeogl came about - initially 
  as a bunch of components wrapping SceneJS, then later as a completely new component-based WebGL engine, stripped down 
  to the bare essentials for fast CAD-like visualization.   
 <br>
 Finally, behind the scenes there is [BIMServer](http://bimserver.org/), which serves IFC models to BIMSurfer as 
 binary streams over WebSockets. The principle developers of BIMServer are [Thomas Krijnen](https://github.com/aothms) (who also handles various UI elements), [Ruben de Laat](https://github.com/rubendel) and [Leon van Berlo](https://github.com/berlotti) (who is also overall project leader).

## BIMSurfer requirements

BIMSurfer has certain requirements that have shaped xeogl, such as: 
     
 * fast rendering of [large numbers of independently transformed geometries](http://xeogl.org/examples/#profiling_statistics) 
 * efficient queries for boundaries of things, so that we can [fly the camera to look at them](http://xeogl.org/examples/#animation_CameraFlightAnimation_AABB)
 * extensible camera animation and control components, like [xeogl.BIMCameraControl](http://xeogl.org/examples/#interaction_BIMCameraControl)

### Responsiveness

To keep applications like BIMSurfer from locking up while loading monstrous amounts of content, xeogl defers all scene updates (eg. creation of 
scene objects, generation of geometries, shader allocations, matrix updates etc) to a FIFO task queue, then executes 
a few tasks from the queue before it renders each frame. 
<br><br>
By amortizing the cost of updates across many frames like this, xeogl also prevents WebGL from freaking out and thinking 
that it's running on a potato, which may cause it to lock FPS down to a lower rate.
<br><br>In the screen capture above, we're loading a test model containing around 3500 objects. While those are loading, 
we can still show and hide things, rotate the model, and so forth. In future we may end up choosing to intentionally 
throttle the FPS while loading, just to get the initial loading out of the way as quickly as possible, but this technique 
is still potentially useful for dynamically streamed content.   
                             
<br>
I learned this technique from the excellent article *Rendering Optimizations in 
the Turbulenz Engine* by [David Galeano](https://twitter.com/davidgaleano), which you can read 
in [WebGL Insights](https://www.amazon.com/WebGL-Insights-Patrick-Cozzi/dp/1498716075). 
  
## Future extension

Using an open source 3D engine within an app like BIMSurfer has the advantage that all enhancements made to the engine 
 are then automatically available to the app. 
<br><br>
xeogl has a growing library of generic plugin components that we can easily drop into our apps 
 to add more functionality. I'm not sure exactly how many of these currently overlap with BIMSurfer's roadmap, but 
 a few possibilities might be: 

 * Loading glTF BIM models using the [xeogl.GLTFModel](http://xeogl.org/docs/classes/GLTFModel.html) component
 * WebVR and Cardboard effect components (in progress)
 * Rendering on zSpace VR displays using the [xeogl.ZSpaceEffect](http://xeogl.org/docs/classes/ZSpaceEffect.html) component
 * Walkthroughs with camera waypoints using the [xeogl.CameraPathAnimation](http://xeogl.org/docs/classes/CameraPathAnimation.html) component
 
Some key features are planned for xeogl's rendering core, which will be great to have in BIMSurfer:

 * [order-independent transparency](https://en.wikipedia.org/wiki/Order-independent_transparency), 
 * [view frustum culling](http://www.lighthouse3d.com/tutorials/view-frustum-culling/), and
 * [shadow mapping](https://en.wikipedia.org/wiki/Shadow_mapping)
 
Right now, however, the plain old CAD-ish geometric features are xeogl's priority.

## Get involved!

The best way contribute to BIMSurfer is to fork the [repository at GitHub](https://github.com/opensourcebim/BIMsurfer).
We also use the [community forum](support.opensourcebim.org) from the open source BIM collective. Feel free to interact 
with other developers to help answer your questions. Then prepare a new branch with a new feature and post a pull request. 
We will review your code and commit it into the official BIM Surfer code base. Alternatively you can also attach a patch 
into our [issue tracker](https://github.com/opensourcebim/BIMsurfer/issues). 

 

 

 



 
 
 
 
     
 





