---
layout: post
title: GPU-Assisted Picking in xeogl
description: "A multi-layer color-indexing rendering algorithm which ray-picks 3D scene objects in linear time"
thumbnail: xeogl/pickingRaycastEntity.gif
modified: 2017-30-08
category: articles
comments: false
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
GPU-assisted picking system, which uses a multi-layer color-indexing rendering algorithm to ray-pick scene entities
in linear time complexity.
<br><br>
![]({{ site.url }}/images/xeogl/pickingTeapots.png)

## Introduction

The color-indexed "WYSIWYG" picking method, which takes advantage of graphics hardware to render scene objects into an auxiliary frame buffer,
was first proposed by Robin Forrest in the mid-1980s and used in 3D painting by Hanrahan and Haeberli [[Hanrahan1990](#hanrahan1990)]. In their
implementation, each triangle is assigned a unique color which is used as an identifier. Given the cursor's canvas coordinates and a map of IDs
to triangles, the picked position on the surface can be found by retrieving color values from the frame buffer and mapping them back to triangles.
<br><br>
Jeff Lander extended this approach to calculate the exact intersection information, ie. the barycentric coordinates
within the intersected triangle. By setting additional color values to the three triangle vertices, he calculated the barycentric
coordinates by interpolation after the rasterization stage [[Lander2000](#lander2000)].
<br>
<br>
xeogl uses a three-stage picking technique:

1. Color-indexed method to find the picked object.
2. Color-indexed method to find the picked triangle within the object.
3. Calculations in JavaScript to find the barycentric coordinates, position, normal and UV.

## Four types of picking in xeogl

xeogl supports four types of picking, which are:

 1. pick entity at canvas coordinates,
 2. pick point on entity surface at canvas coordinates,
 1. pick entity with World-space ray, and
 2. pick point on entity surface with World-space ray.

Types 1 and 3 just return the picked entity.
<br><br>Types 2 and 4 return the picked entity, as well as information about the surface intersection, which includes:

 * the triangle,
 * barycentric coordinates within the triangle,
 * World space coordinates,
 * View space coordinates,
 * normal vector, and
 * UV coordinates.

Note that we'll only get a normal if the entity's geometry has normals, and UV coordinates if the
geometry has UVs. xeogl finds the values for these by interpolating within the values for the triangle vertices using
the barycentric coordinates.

## 1. Pick entity at canvas coordinates

This type of picking is the simplest: we pick the closest entity behind the given canvas coordinates. This is equivalent to
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

 We can also listen for pick hits on individual entities:

 {% highlight javascript %}
  var entity = gearbox.entities["gearbox#someGear"];
  entity.on("picked", function(hit) {
        //...
 });{% endhighlight %}

Internally, xeogl performs the following steps for this type of picking:

1. User picks at canvas coordinates.
2. Render the scene to an auxiliary frame buffer, filling each entity with a unique colour that is the RBGA-encoded
index of its position within xeogl's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the entity in the display list.
4. Return a hit result containing the entity.

## 2. Pick point on entity surface at canvas coords

Like the previous type of picking, this one also picks the closest entity behind the given canvas coordinates, but also
gets geometric information about the point on the entity's surface that lies right behind those canvas coordinates.
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
    var localPos = hit.localPos;    // Local-space coordinates
    var worldPos = hit.worldPos;    // World-space coordinates
    var viewPos = hit.viewPos;      // View-space coordinates
    var bary = hit.bary;            // Barycentric coordinates within the triangle
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

Internally, xeogl performs the following steps for this type of picking:

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
to Find the normal vector and UV coordinates at that position.
9. Return a hit result containing the picked entity, the triangle, and the ray-triangle intersection info (see code example above).

For step (4) we lazy-compute extra vertex position and color arrays for the entity geometry, so that we can render each
triangle with a unique flat color. It's actually very fast, so you'd hardly notice it, even with big meshes - [here's
the math function](https://gist.github.com/xeolabs/6e53681b88e00773075e97460a5e7c72).

## 3. Pick entity with World-space ray

For this type of picking, xeogl fires a ray through the scene, in World-space, to pick the first entity it hits. I originally added
this to support the 3D stylus in some [xeogl demo apps for the zSpace 300](http://xeolabs.com/articles/xeogl-and-zspace).
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

Internally, xeogl performs the following steps for this type of picking:

1. User picks using a ray in World-space.
2. Position the camera to look along the ray.
3. Render the scene to an auxiliary frame buffer, filling each entity with a unique colour that is the RBGA-encoded
index of its position within xeogl's internal display list.
4. Restore the camera to its previous position.
5. Read the colour from the framebuffer at the canvas coordinates, map the colour back to an entity in the display list.
4. Return a hit result containing the entity.

## 4. Pick point on entity surface with World-space ray

Like the previous type of picking, this one also involves firing a ray through the scene in World-space, to pick
 an entity, but this time we're also getting geometric information about the intersection of the ray with the
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
     var localPos = hit.localPos;    // Local-space coordinates within the triangle
     var worldPos = hit.worldPos;    // World-space coordinates within the triangle
     var viewPos = hit.viewPos;      // View-space coordinates within the triangle
     var bary = hit.bary;            // Barycentric coordinates within the triangle
     var normal = hit.normal;        // Interpolated normal vector within the triangle
     var uv = hit.uv;                // Interpolated UVs within the triangle
}{% endhighlight %}

Internally, xeogl performs the following steps for this type of picking:
 
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

## Pros and cons

Ray-picking usually involves testing a ray for intersection with each object's bounding box,
then having picked an object, testing the ray for intersection with each of the object's triangles. This is normally done
on the CPU, with the assistance of a spatial lookup structure, such as a KD-tree. I believe this is how its done in THREE.js.
<br><br>
Compared to the usual technique, xeogl will be faster for huge numbers of objects and triangles, since it avoids these
CPU-intensive calculations. There are some disadvantages, however:

* xeogl's ray-picking technique only works for triangles so far. It should be possible to extend it to support line segments though.
* The CPU-intensive technique can find multiple objects that intersect a ray, while xeogl's technique only finds the closest
intersecting object to the ray origin.
* As mentioned when [ray-picking triangles](#pick-point-on-entity-surface-with-world-space-ray), xeogl lazy-computes extra
vertex position and color arrays for each entity we're about to ray-pick a triangle from, so that we can render each of its
triangles with a unique flat color. That happens on-the-fly, just before we attempt to pick a triangle on the entity,
 and can't be deferred to xeogl's' frame-budgeted task queue (like most computationally-intensive things in xeogl), because
 we need those arrays immediately. Therefore there can be a moment's delay when picking a complex entity for the first time,
 plus the sudden memory needs for those two extra arrays. Since those extra arrays belong to the geometry, which can be
 instanced by multiple entities, this can be mitigated if we're able to share (instance) our geometries among many entities. For example,
  if we have a scene made up mostly of cubes, where we have one cube geometry shared by all our entities, then when we ray-pick
   our entities, we're only calculating those arrays once, for the shared geometry.

## Work remaining

* When rendering to auxiliary frame buffers for ray-picking, xeogl renders the whole canvas-sized viewport. That's wasteful
in terms of GPU efficiency and memory, so I need reduce the picking framebuffer, and temporarily the picking viewport, to 1x1 size.
* As mentioned in the previous section, I also need to extend the ray-picking technique to select line segments.

## References

### Hanrahan1990

P. Hanrahan and P. Haeberli, “Direct WYSIWYG painting and texturing on 3D shapes,” ACM SIGGRAPH Computer Graphics, vol. 24, no. 4, pp. 215–223, 1990. View at Publisher · View at Google Scholar

### Lander2000

J. Lander, “Haunted trees for halloween,” Game Developer Magazine, vol. 7, no. 11, pp. 17–21, 2000. View at Google Scholar
