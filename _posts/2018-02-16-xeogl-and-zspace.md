---
layout: post
title: Creating WebGL Demos for zSpace using xeogl 
description: Partnered with <b>zSpace</b> to develop WebGL-based demos on the <b>zSpace 300</b> system using <b>xeogl</b>, an open source library I created for 3D visualization in the browser.
thumbnail: xeogl/zspace/zSpaceDeviceThumb.jpg
tech: "WebGL, WebVR, xeogl"
modified: 2017-19-03
category: portfolio
comments: false
location: "Sunnyvale, CA"
article: true
tags: [xeogl, webgl, zspace, AR]
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

[**zSpace**](http://zspace.com) is a technology firm based in Sunnyvale, California that creates mixed reality systems that combine elements of virtual and augmented reality in a specially-built computer system, which has a quad-buffered stereo display that works with IR-tracked active-shuttered glasses and a stylus. 

<br>[**xeogl**](http://xeogl.org) is an open-source WebGL-based 3D engine that I've been working on for the past year or so. 

<br>Last year, zSpace loaned me one of their *zSpace 300* displays so that I could put together some xeogl demos for its beta WebVR support. If you were at [GDC 2017](www.gdconf.co), you might have seen those demos running at the [Khronos booth](https://www.khronos.org/news/events/gdc-2017#gdc2017_booth).

<br>
 ![]({{ site.url }}/images/xeogl/zspace/zSpaceDevice.jpg)
 
 <br>These demos were my first foray into WebVR and were lots of fun to make. If you happen to be using a zSpace 300 right now, go ahead and click these thumbnails to run them: 
 <br>
 
 ----|----|----
 [![]({{ site.url }}/images/xeogl/zspace/zspace-geometries.png)](http://xeogl.org/examples/#effects_zspace_geometries) | [![]({{ site.url }}/images/xeogl/zspace/zspace-gearbox.png)](http://xeogl.org/examples/#effects_zspace_gearbox) | [![]({{ site.url }}/images/xeogl/zspace/zspace-saw.png)](http://xeogl.org/examples/#effects_zspace_ReciprocatingSaw) 
 ----|----|----
 [Simple geometries](http://xeogl.org/examples/#effects_zspace_geometries) | [Gearbox loaded from glTF](http://xeogl.org/examples/#effects_zspace_gearbox) | [Saw loaded from glTF](http://xeogl.org/examples/#effects_zspace_ReciprocatingSaw)
 
## zSpace support in xeogl 
 
 To support zSpace, I added two new component types to xeogl: 
 
 * [ZSpaceEffect](http://xeogl.org/docs/classes/ZSpaceEffect.html) - renders its scene in quad-buffered stereo while updating the viewing and projection transforms off the zSpace WebVR device events.
 * [ZSpaceStylusControl](http://xeogl.org/docs/classes/ZSpaceStylusControl.html) - an input control to select and drag entities with the stylus. 
 
 Let's take a quick look at how these components are used.
  
<br>First, I'll set up a 3D scene by importing a [glTF](https://github.com/KhronosGroup/glTF) model into xeogl, using a [GLTFModel](http://xeogl.org/docs/classes/GLTFModel.html) component:

 {% highlight javascript %}
 var model = new xeogl.GLTFModel({
    src: "models/gltf/2.0/Reciprocating_Saw/PBR-SpecGloss/Reciprocating_Saw.gltf",
    position: [1, 0, 0],
    rotation: [90, 0, 0]    
 });
 {% endhighlight %}

 Our model is the red reciprocating saw shown in the third thumbnail above. I've also attached a [Rotate](http://xeogl.org/docs/classes/Rotate.html) to our GLTFModel, just to tip it upright so that we can see it more clearly.
 
 <br>Now, to view the model on the zSpace display, I'll simply create a [ZSpaceEffect](http://xeogl.org/docs/classes/ZSpaceEffect.html):

 {% highlight javascript %}
 var zspaceEffect = new ZSpaceEffect();
 {% endhighlight %}

 That activates immediately, rendering whatever we have in our scene in quad-buffered stereo. At this point, we can put on our stereo glasses and see the model rendering in 3D. 

### Detecting zSpace support
 
 The [ZSpaceEffect](http://xeogl.org/docs/classes/ZSpaceEffect.html) will fire a "supported" event once it has determined whether or not your browser is running on a zSpace viewer:
 
 {% highlight javascript %}
 zspaceEffect.on("supported", function (supported) {
     if (!supported) { 
         this.error("This computer is not a ZSpace viewer! Inconceivable.");           
     }
 });
 {% endhighlight %}

### Stylus tracking
 
 Now let's track the World-space position and direction of the zSpace stylus. Also, whenever we press button 0 on the stylus while intersecting an entity, we'll ray-pick the entity with the stylus.
  
 {% highlight javascript %}
 zspaceEffect.on("stylusPos", function(pos) { 
     console.log("Stylus position: " + pos);
 });
    
 zspaceEffect.on("stylusDir", function(dir) { 
     console.log("Stylus direction: " + dir);
 });
 
 zspaceEffect.on("stylusButton0", function(down) { 
     if (down) {
         var hit = zspaceEffect.scene.pick({
             pickSurface: true,
             origin: zspaceEffect.stylusPos,
             direction: zspaceEffect.stylusDir
         });
  
         console.log("Entity selected with stylus: " + hit.entity.id);
     }
 });
 {% endhighlight %}

 note that we can also just poll the ZSpaceEffect at any time for the current state of the stylus:
 
 {% highlight javascript %}
 // Get stylus position
 var stylusPos = zspaceEffect.stylusPos;
 var stylusDir = zspaceEffect.stylusDir;
 
 // Get button states
 var button0 = zspaceEffect.stylusButton0; // Boolean
 var button1 = zspaceEffect.stylusButton1;
 var button2 = zspaceEffect.stylusButton2;
 {% endhighlight %}

### ZSpaceStylusControl
  
 In xeogl we'd normally wrap your input controls in reusable components, so I added the [ZSpaceStylusControl](http://xeogl.org/docs/classes/ZSpaceStylusControl.html) as an boilerplate/example on which you might base your own zSpace stylus control. 
   
 <br>This component:

 * hooks into the [ZSpaceEffect](http://xeogl.org/docs/classes/ZSpaceEffect.html) events we just saw,  
 * renders a 3D line segment that appears to extend from the tip of the stylus,
 * grabs intersecting entities when you push button 0, and 
 * drags grabbed entities around while button 0 is down.

 To use it, all we need to do is add it to our scene:
   
 {% highlight javascript %}
 var zspaceStylusControl = new xeogl.ZSpaceStylusControl();
 {% endhighlight %}

 At this point we can see a ray extending from the tip of our stylus, with which we can select and drag entities.
  
### Conclusion

**zSpace's support for WebVR is still in beta, so xeogl's support for zSpace is also in beta.** Therefore, I'll be changing the implementation of these xeogl components moving forwards, but their API should remain the same. Stay tuned!

<iframe src="//giphy.com/embed/mRdkHVQ1NdUWc" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="http://giphy.com/gifs/mRdkHVQ1NdUWc">via GIPHY</a></p>

