---
layout: post
title: Automatic WebGLContestLost Recovery in SceneJS
description: "How SceneJS gets back on its feet when WebGL blows a fuse"
modified: 2013-05-31
category: articles
comments: false
tags: [scenejs, tutorial]
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

[SceneJS](http://scenejs.org) can automatically recover from lost WebGL context without disruption to your scene state, and
more importantly, without needing to reload anything.<br><br>
SceneJS was the first to do this, but pretty soon this will be a standard feature of all retained-mode
WebGL frameworks.
<br><br>
Check out the example below, in which we blow the context away every five seconds while you orbit the rising bubbles using the mouse:
<br><br>
[![SceneJS First Example]({{ site.url }}/images/scenejs/webglContextLost.jpg)](http://scenejs.org/examples.html?page=webglContextLost)

[Run this example](http://scenejs.org/examples.html?page=webglContextLost).

## So, what's WebGLContextLost?

It's the devil.
<br><br>
The GPU is a shared resource, and there are times when it might be taken away from our WebGL applications, such as when
there are too many applications holding them, or when another application does something that ties up the GPU too
long.
<br><br>
In such cases the operating system or browser may decide to reset the GPU to regain control. There's
a [nice tutorial](http://www.khronos.org/webgl/wiki/HandlingContextLost) over at Khronos.org that describes how to handle
this in our applications. It describes how application processes such as asset loads can be disrupted by context loss, and
how an app will then need to reallocate its textures, VBOs, shaders etc. on a new context afterwards.
<br><br>
SceneJS retains all state and assets, such as matrices, geoemtry and textures etc in its scene graph. Then when the context
is recovered after loss, it simply reallocates those assets on the new context.