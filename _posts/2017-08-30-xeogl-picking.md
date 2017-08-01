---
layout: post
title: GPU-Assisted Picking in xeogl
description: "How picking works in xeogl"
modified: 2017-30-08
category: articles
comments: true
tags: [xeogl, picking, webgl]
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

**[xeogl](http://xeogl.org)** is a WebGL-based engine for 3D visualization. In this article, I'll describe xeogl's
GPU-assisted picking system, which uses a multi-layer color-indexing rendering algorithm to ray-pick objects in linear
time complexity.

## Background

TODO:
https://www.hindawi.com/journals/ijcgt/2009/730894/
http://people.csail.mit.edu/ericchan/bib/pdf/p215-hanrahan.pdf

## Picking methods in xeogl

xeogl supports four methods of picking, which are as follows:

 1. pick entity at canvas coordinates,
 2. pick point on entity surface at canvas coordinates,
 1. pick entity with World-space ray, and
 2. pick point on entity surface with World-space ray.

Methods 1 and 3 just return the picked entity.
<br><br>Methods 2 and 4 return the picked entity
 along with additional information about the point that was picked on its surface. That includes:

 * the **entity**
 * the **triangle** that contains the position,
 * the **barycentric coordinates** of the position within the triangle,
 * the **World space coordinates** of the position,
 * the **View space coordinates** of the position,
 * the **normal vector** at the the position, and
 * the **UV coordinates** at the position.

Note that we'll only get a normal vector if the entity's geometry has normals, and we'll only get UV coordinates if the
geometry has UVs. xeogl finds the values for these by interpolating within the values for the triangle vertices using
the barycentric coordinates.

### 1. Pick entity at canvas coordinates

This picking method is the simplest: we pick the closest entity behind the given canvas coordinates. This is equivalent to
firing a ray through the canvas, down the negative Z-axis, to find the first entity it hits.
<br>
<br>
[![]({{ site.url }}/images/xeogl/pickingCanvasEntity.gif)](http://xeogl.org/examples/#picking_canvas_pickEntity)
   <br>[Run demo](http://xeogl.org/examples/#picking_canvas_pickEntity)
<br>
<br>
To show how to use it, we'll start by loading a [GLTFModel](http://xeogl.org/docs/classes/GLTFModel.html):

 {% highlight javascript %}
 var gearbox = new xeogl.GLTFModel({
     src: "models/gltf/gearbox/gearbox_assy.gltf"
 }); {% endhighlight %}

 With our model loaded and sitting in the middle of our canvas, we'll try picking an entity at some canvas coordinates:

 {% highlight javascript %}
 var hit = gearbox.scene.pick({
     canvasPos: [500,400]
 }); {% endhighlight %}

 If we hit an entity, then we'll get a result containing a reference to it:

 {% highlight javascript %}
 if (hit) { // Picked an entity
     var entity = hit.entity;
 } {% endhighlight %}

Internally, xeogl performs the following steps for this picking method:

1. User picks at canvas coordinates.
2. Render the scene to an auxiliary frame buffer, filling each entity with a unique colour that is the RBGA-encoded
index of its position within xeogl's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the entity in the display list.
4. Return a hit result containing the entity.

### 2. Pick point on entity surface at canvas coordinates

Like the previous picking method, this one also picks the closest entity behind the given canvas coordinates, but also
gets information about the point on the entity's surface that lies right behind those canvas coordinates.
 <br>
 <br>
[![]({{ site.url }}/images/xeogl/pickingCanvasEntitySurface.gif)](http://xeogl.org/examples/#picking_canvas_pickSurface)
   <br>[Run demo](http://xeogl.org/examples/#picking_canvas_pickSurface)
<br>
<br>
Reusing the scene from the previous example, we'll pick again at the canvas coordinates, but this time
 we'll supply a ````pickSurface```` flag to indicate that we want information about the point we're picking on the surface:

{% highlight javascript %}
var hit = gearbox.scene.pick({
    canvasPos: [500,400],
    pickSurface: true,    // <<----- Indicates that we want to pick on surface
});{% endhighlight %}

Here's what we get in the pick result this time:

{% highlight javascript %}
if (hit) { // Picked an entity

    var entity = hit.entity;        // Entity we picked
    var primitive = hit.primitive;  // Type of primitive we picked, usually "triangles"
    var primIndex = hit.primIndex;  // Index of triangle's first element within Entity's Geometry indices array
    var indices = hit.indices;      // Value of each of the triangle's vertex indices (a three element array)
    var localPos = hit.localPos;    // Local-space position
    var worldPos = hit.worldPos;    // World-space position
    var viewPos = hit.viewPos;      // View-space position
    var bary = hit.bary;            // Barycentric coordinate within the triangle
    var normal = hit.normal;        // Interpolated normal vector within the triangle
    var uv = hit.uv;                // Interpolated UVs within the triangle
}{% endhighlight %}

We can then get the triangle vertices like so:

{% highlight javascript %}
var entity = hit.entity;
var geometry= entity.geometry;

// Get indices for each of the vertices of the picked triangle

var primIndex = hit.primIndex;

var indexA = geometry.indices[primIndex];
var indexB = geometry.indices[primIndex + 1];
var indexC = geometry.indices[primIndex + 2];

// Get the triangle's local-space vertex positions

var ax = geometry.position[indexA];
var ay = geometry.position[indexA + 1];
var az = geometry.position[indexA + 2];

var bx = geometry.position[indexB];
var by = geometry.position[indexB + 1];
var bz = geometry.position[indexB + 2;

var cx = geometry.position[indexC];
var cy = geometry.position[indexC + 1];
var cz = geometry.position[indexC + 2];

// Another way to get the indices
var indexABC = hit.indices;{% endhighlight %}

Internally, xeogl performs the following steps for this picking method:

1. User ray-picks at given canvas coordinates.
2. Render the scene to an auxiliary frame buffer, filling each entity with a unique colour that is the RBGA-encoded
index of its position within xeogl's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to an entity in the display list.
4. Clear the framebuffer, then render the triangles of the picked entity to it, filling each triangle with a unique color that
is the RBGA-encoded index of triangle's first element within the entity's geometry indices.
5. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the picked triangle.
6. Now that we have the entity and the triangle, make a ray in clip-space, originating at the eye position and passing through
the near projection plane at a position corresponding to where we picked, then unproject that ray to get a ray in the entity's
local coordinate space.
7. Find the intersection of the ray with the triangle in local space.
8. Find the barycentric coordinates of the local-space intersection, then use those to interpolate within the triangle
to find the normal vector and UV coordinates at that position.
9. Return a hit result containing the picked entity, the triangle, and the ray-triangle intersection info (see code example above).

For step (4) we lazy-compute extra vertex position and color arrays for the entity geometry, so that we can render each
triangle with a unique flat color. It's actually very fast, so you'd hardly notice it, even with big meshes - [here's
the math function](https://gist.github.com/xeolabs/6e53681b88e00773075e97460a5e7c72).

### 3. Pick entity with World-space ray

For this picking method, xeogl fires a ray through the scene, in World-space, to pick the first entity it hits. I originally added
this to support the 3D stylus for the [zSpace integration project](http://xeolabs.com/articles/xeogl-and-zspace).
<br>
<br>
[![]({{ site.url }}/images/xeogl/pickingRaycastEntity.gif)](http://xeogl.org/examples/#picking_raycast_pickEntity)
   <br>[Run demo](http://xeogl.org/examples/#picking_raycast_pickEntity)
<br>
<br>
To show how it's done, we'll fire a ray through the scene in World-space, to pick the first entity hit by the ray:

{% highlight javascript %}
var hit = scene.pick({
    origin: [0,0,-5],   // Ray origin
    direction: [0,0,1], // Ray direction
    pickSurface: false   // <<------ Don't want to pick a point on the surface
});

if (hit) { // Picked an entity with the ray
     var entity = hit.entity;   // Entity we picked
}{% endhighlight %}

xeogl performs the following steps for this picking method:

1. User picks using a ray in World-space.
2. Position the camera to look along the ray.
3. Render the scene to an auxiliary frame buffer, filling each entity with a unique colour that is the RBGA-encoded
index of its position within xeogl's internal display list.
4. Restore the camera to its previous position.
5. Read the colour from the framebuffer at the canvas coordinates, map the colour back to an entity in the display list.
4. Return a hit result containing the entity.

### 4. Pick point on entity surface with world-space ray

Like the previous picking method, this one also involves firing a ray through the scene in World-space, to pick
 an entity, but this time we're also getting some geometric information about the intersection of the ray with the
 entity surface.
<br>
<br>
[![]({{ site.url }}/images/xeogl/pickingRaycastEntitySurface.gif)](http://xeogl.org/examples/#picking_raycast_pickSurface)
   <br>[Run demo](http://xeogl.org/examples/#picking_raycast_pickSurface)
<br>
<br>
Lets fire a ray through the scene in World-space, to pick a 3D position on the surface of the first entity hit by the ray:

{% highlight javascript %}
var hit = scene.pick({
    origin: [0,0,-5],   // Ray origin
    direction: [0,0,1], // Ray direction
    pickSurface: true   // <<--------- Indicates that we want to pick on surface
});

if (hit) { // Picked an entity with the ray

    // Hit result contains the same properties as the second example
    
     var entity = hit.entity;        // Entity we picked
     var primitive = hit.primitive;  // Type of primitive we picked, usually "triangles"
     var primIndex = hit.primIndex;  // Triangle's first index within geometry indices
     var indices = hit.indices;      // Triangle's vertex indices (a three element array)
     var localPos = hit.localPos;    // Local-space position within the triangle
     var worldPos = hit.worldPos;    // World-space position within the triangle
     var viewPos = hit.viewPos;      // View-space position within the triangle
     var bary = hit.bary;            // Barycentric coordinate within the triangle
     var normal = hit.normal;        // Interpolated normal vector within the triangle
     var uv = hit.uv;                // Interpolated UVs within the triangle
}{% endhighlight %}

Internally, xeogl performs the following steps for this picking method:
 
1. User picks using a ray in World-space.
2. Position the camera to look along the ray.
3. Render the scene to an auxiliary frame buffer, filling each entity with a unique colour that is the RBGA-encoded
index of its position within xeogl's internal display list.
4. Read the colour from the framebuffer at the canvas coordinates, map the colour back to an entity in the display list.
5. Clear the framebuffer, then render the triangles of the picked entity to it, filling each triangle with a unique color that
is the RBGA-encoded index of the triangle's first element within the entity's geometry indices.
6. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the picked triangle.
7. Restore the camera to its previous position.
8. Now that we have the entity and the triangle, make a ray in clip-space, originating at the eye position and passing through
the near projection plane at a position corresponding to where we picked, then unproject that ray to get a ray in the entity's
local coordinate space.
9. Find the intersection of the ray with the triangle in local space.
10. Find the barycentric coordinates of the local-space intersection, then use those to interpolate within the triangle
to find the normal vector and UV coordinates at that position.
11. Return a hit result containing the picked entity, the triangle, and the ray-triangle intersection info (see code example above).

# Conclusion

# References

* P. Hanrahan and P. Haeberli, “Direct WYSIWYG painting and texturing on 3D shapes,” ACM SIGGRAPH Computer Graphics, vol. 24, no. 4, pp. 215–223, 1990. View at Publisher · View at Google Scholar
