---
layout: post
title: SceneJS v4.0 Released
description: "More rendering capabilities!"
modified: 2014-07-24
category: articles
comments: true
tags: [scenejs, releases]
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

In addition to fast rendering of large

 * Render targets
 * Rendering pipelines
 * Postprocessing effects
 * Multipass rendering
 * Procedural textures
 * Performance optimisations from NVIDIA
 * New examples browser


## Postproccessing effects

SceneJS 4.0 lets you build effects pipelines of virtually unlimited complexity. You can render portions of your scene to
colour and depth *render targets*, which you then apply as textures, or feed into custom shaders in other parts of your scene.
In the example below, I'm doing a *depth-of-field* postprocessing effect by rendering the scene to colour and depth targets, then
piping those targets into a second stage where we render the colour target to a screen-aligned quad, while blurring each pixel
in proportion to its value in the depth buffer.

<iframe width="560" height="315" src="//www.youtube.com/embed/HNsJ7j6XZJU" frameborder="0" allowfullscreen></iframe>
<br>
[Run this demo](http://scenejs.org/examples/index.html#postprocessing_depthOfField)
<br><br>I've implemented a few postprocessing effects as node types that you can just drop into your scene:

 * [Depth-of-field](http://scenejs.org/examples/index.html#postprocessing_depthOfField_autofocus)
 * [Blur](http://scenejs.org/examples/index.html#postprocessing_blur)
 * [Scanlines](http://scenejs.org/examples/index.html#postprocessing_scanlines)
 * [Sepia](http://scenejs.org/examples/index.html#postprocessing_sepia)
 * [Technicolor](http://scenejs.org/examples/index.html#postprocessing_technicolor)

Stay tuned - I'll do a few more soon, such as a *bloom filter* and maybe *motion blur*.

## Multipass rendering

SceneJS v4.0 also supports *multi-pass* rendering, where you can configure the scene to render multiple passes per frame,
while updating scene state for each pass. This is really useful for stereoscopic effects like Anaglyph 3D, where we render
one pass to the green-blue color channels with camera looking through the left eye, and a second pass to the red channel
 with the camera looking through the right eye.

<iframe width="560" height="315" src="//www.youtube.com/embed/WQ5FOA1V40o?list=PLZKDjaM3SUtcBMa-CtxT8WCI_mCF2VGpD" frameborder="0" allowfullscreen></iframe>
<br>
[Run this demo](http://scenejs.org/examples/index.html#effects_anaglyph)
<br><br>
Oculus Rift support didn't make it into this release unfortunately (I don't actually have a Rift device, so need a bit more time to
  tweak things).

## Procedural textures

Now that we can render portions of our scenes to render targets, then apply those as textures, we can generate textures using shaders lifted straight off
GLSL sharing sites like [ShaderToy](http://shadertoy.com) and [GLSL Sandbox](http://glslsandbox.com). Check this one out, copied from GLSL Sandbox:

<iframe width="420" height="315" src="//www.youtube.com/embed/1I71UWg1Wmg?rel=0" frameborder="0" allowfullscreen></iframe>
<br>
[Run this demo](http://scenejs.org/examples/index.html#texture_procedural_color_water)
<br><br>

## Optimisations from NVIDIA

[Olli Etuaho](http://www.oletus.fi/) from NVIDIA has made some valuable optimizations:

 * [Vertex Array Objects (VAO)](http://blog.tojicode.com/2012/10/oesvertexarrayobject-extension.html)
 * [Vertex interleaving](http://blog.tojicode.com/2011/05/interleaved-array-basics.html)
 * Reduced unnecessary uniform reloading

These are primarily aimed at making SceneJS render faster on mobile, GPU-bound devices, and Olli reports as much as a
45% speedup on Tegra-based devices with some scenes! He's also done some optimisations to [THREE.js](http://threejs.org) as well, so props to Olli on that.

## Renamings


## Texture nodes


## New examples browser

And finally - a [new examples browser](http://scenejs.org/examples/index.html#scenegraph_firstExample) based on the one
used over at THREE.js. Hopefully it doesn't look too similar - I may have to style it a bit more.

## Post-release

A few things didn't make it into this release, but I'll be putting them into minor version releases over the next few weeks:

 * Normal mapping
 * Oculus Rift