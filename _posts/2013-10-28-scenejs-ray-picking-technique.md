---
layout: post
title: Pssst...How Fast Ray Picking Works in SceneJS
description: "A quick writeup on how I'm using the colour buffer to help find ray-intersects and avoid expensive intersection calculations"
modified: 2013-05-31
category: articles
comments: true
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

Normally a ray-pick is done with expensive computations to find intersections of rays with meshes and so forth.
SceneJS, however, uses a fast GPU-assisted technique that employs the colour buffer to help find the ray-intersection point, which
 avoids those sorts of computations altogether.
 <br><br>
I couldn't find anybody else doing this in WebGL or OpenGL ES (maybe I should have looked harder?), so at first I thought
it must be too good to be true. However, despite a small amount of numeric inaccuracy when the front and back clip planes
are far apart, it seems work well enough, so I went with it.

## The technique
I'll assume that you've read about [2D picking]({{site.url}}/articles/scenejs-picking)
and [ray picking]({{site.url}}/articles/scenejs-ray-picking), which describe how you can wrap your objects with ```name```
nodes to assign them logical pick names.
<br><br>
The steps of my technique are:

1. User ray-picks canvas at coordinates (X, Y).
2. Do a render pass to a hidden frame buffer, in which the objects within each ```name``` are rendered in a colour that
uniquely maps to that ```name```'s value.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the name value. Now we have the pick name.
4. Do a second render pass to another hidden frame buffer, this time rendering just the picked geometry, with each pixel
colour being the clip-space Z-value packed into an RGBA value.
5. Read the colour from the framebuffer at the canvas coordinates and unpack it to the clip-space Z value. Now we have the
clip-space Z, which will be in the range of [0..1], with near clip plane at 0 and far clip plane at 1.
6. Transform the canvas coordinates to clip-space. Make a ray from clip space (X,Y,0) to (X,Y,1) and transform that ray
into world-space by the inverse view and projection matrices.
7. Linearly interpolate along ray by the value of our clip-space Z, to find the world-space coordinate (X,Y,Z).
8. Voila, we have the picked name, canvas (X,Y) and world-space (X,Y,Z) for the pick hit.

#### Packing clip-space Z in GLSL

{% highlight glsl %}
vec4 packDepth(const in float depth) {
    const vec4 bitShift = vec4(256.0*256.0*256.0, 256.0*256.0, 256.0, 1.0);
    const vec4 bitMask  = vec4(0.0, 1.0/256.0, 1.0/256.0, 1.0/256.0);
    vec4 res = fract(depth * bitShift);
    res -= res.xxyz * bitMask;
    return res;
}
{% endhighlight %}

#### Unpacking clip-space Z in JavaScript

{% highlight javascript %}
function unpackDepth(depthZ) {
    var vec = [depthZ[0] / 256.0, depthZ[1] / 256.0, depthZ[2] / 256.0, depthZ[3] / 256.0];
    var bitShift = [1.0 / (256.0 * 256.0 * 256.0), 1.0 / (256.0 * 256.0), 1.0 / 256.0, 1.0];
    return SceneJS_math_dotVector4(vec, bitShift);
};
{% endhighlight %}

#### Calculating clip-space Z in the fragment shader
Step (4) requires that we have the view-space position in the fragment shader, which we pass through from the vertex shader. It also requires us to feed the locations of the near and far clipping planes into the fragment shader (which we take from the scene's camera node). Using these, we calculate the clip-space depth like so:

{% highlight glsl %}
float depth = (uZNear - vViewVertex.z) / (uZFar - uZNear);
gl_FragColor = packDepth(depth);
{% endhighlight %}

* SceneJS internally caches the hidden frame buffers to avoid re-rendering them. This means that when we do a subsequent pick, as long as a re-render is not neccessary after objects have moved or changed appearance, we just re-read the buffers without repeating any rendering passes.
* For picking, many WebGL frameworks will save time by doing a picking render of only a 1x1 viewport at the canvas coordinates. SceneJS renders the entire view for picking so that it can cache the pick framebuffers as just mentioned. This is an optimisation geared towards fast mouse-over picking effects in model viewing apps, such as highlighting and tooltips.

## Drawbacks to this technique

* The technique described here trades accuracy for speed. Packing and unpacking the clip-space Z to and from a colour value is lossy. Hopefully in future it will be possible to instead read the WebGL depth buffer, which will preserve much more precision.
* Another limitation is that this technique does not find any topological information on the pick hit: it only finds the name and a world-space coordinate. When picking a mesh for example, it does not report the actual face that was picked.
