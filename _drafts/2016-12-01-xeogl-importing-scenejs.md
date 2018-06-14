---
layout: post
title: Importing SceneJS content into xeogl
description: "On the joys of data-driven scene definitions and recursive-descent parsing"
modified: 2016-19-09
category: articles
comments: false
tags: [xeogl, scenejs, picking, webgl]
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

**[xeogl](http://xeogl.org)** is a WebGL-based engine for 3D visualization. In this article, I'll describe how xeogl's 
GPU-assisted picking lets us efficiently select things in complex scenes.  
 
## Four flavours of picking

In order of sophistication, they are:
 
 1. [picking an entity at canvas coordinates](./#picking-an-entity-at-canvas-coordinates), 
 2. [picking a position on an entity at canvas coordinates](./#picking-a-position-on-an-entity-at-canvas-coordinates), 
 3. [picking an entity with a World-space ray](./#picking-an-entity-with-a-world-space-ray), and
 4. [picking a position on an entity with a World-space ray](./#picking-a-position-on-an-entity-with-a-world-space-ray).
 
When we just pick an entity (flavours 1 and 3), the result returns the entity and nothing else. 
<br><br>
However, when we pick a position on the surface of an entity (flavours 2 and 4), the result also returns what we picked 
at that position, which includes: 

 * the entity
 * the triangle within the entity's geometry, 
 * the barycentric coordinates within the triangle, 
 * the World-space coordinates at the position, 
 * the View-space coordinates at the position,
 * the normal vector at the the position, and 
 * the UV coordinates at the position.
 
This information is useful for things like drawing on Entities, attaching labels, making the camera look at the position 
you clicked on, and many other fun things. xeogl can do this really fast, using shaders to do most of the work.

## Picking an entity at canvas coordinates 

This one's the simplest - we select the closest entity at the given canvas coordinates. This is equivalent to 
firing a ray through the canvas, down the negative Z-axis, to find the closest intersecting entity. 
<br>
<br>
[![](http://xeogl.org/assets/images/picking2DCanvas.gif)](http://xeogl.org/examples/#interaction_picking) 
   <br>[Click to run](http://xeogl.org/examples/#interaction_picking) 
<br>
<br>
To demonstrate, we'll create a [GLTFModel](http://xeogl.org/docs/classes/GLTFModel.html) 
component that loads a glTF model, then point the default camera at it:
 
 {% highlight javascript %}
 var gearbox = new xeogl.GLTFModel({
     src: "models/gltf/gearbox/gearbox_assy.gltf"
 });
 
 // Our gearbox is not centered at the World-space origin,
 // so we need to move the camera to arrange it in view
 var view = gearbox.scene.camera.view;
 view.eye = [184.21, 10.54, -7.03];
 view.look = [159.20, 17.02, 3.21];
 view.up = [-0.15, 0.97, 0.13];
 {% endhighlight %}
 
 Now, with the model loaded and sitting in the middle of our canvas, try picking something:   
 
 {% highlight javascript %}
 var hit = gearbox.scene.pick({  
     canvasPos: [500,400]
 });
 {% endhighlight %}
 
 If we hit an entity, then xeogl will return a *hit result* containing a reference to it:
  
 {% highlight javascript %}
 if (hit) { // Picked an Entity
     var entity = hit.entity;     
 }
 {% endhighlight %}

Internally, xeogl performs the following steps for this type of picking:
 
1. User picks at canvas coordinates.
2. Render the scene to a hidden frame buffer, drawing each Entity with a unique colour that is the RBGA-encoded 
index of the Entity's position within xeogl's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the Entity. 
 
## Picking a position on an entity at canvas coordinates 

Like the previous flavour, this involves finding the closest entity at the given canvas coordinates, but this time we're 
also getting information about the position on the entity's surface that lies right behind those canvas coordinates. In other words, 
 we are *unprojecting* the canvas coordinates.
 <br>
 <br>  
[![](http://i.giphy.com/3o6ZsZ8dQPxc5uQ5wc.gif)](http://xeogl.org/examples/#interaction_picking_triangles_normalAndUV) 
   <br>[Click to run](http://xeogl.org/examples/#interaction_picking_raycasting_pickSurface) 
<br>
<br>
Reusing the scene from the previous example, we'll fire a ray through the canvas coordinates, this time 
 supplying a ````pickSurface```` flag:  

{% highlight javascript %}
var hit = gearbox.scene.pick({   
    canvasPos: [500,400],
    pickSurface: true,    // <<----- Indicates that we want to pick on surface
});
{% endhighlight %}

Here's what's in the hit result this time:   

{% highlight javascript %}
if (hit) { // Picked an Entity

    var entity = hit.entity;        // Entity we picked
    var primitive = hit.primitive;  // Type of primitive we picked, usually "triangles"
    var primIndex = hit.primIndex;  // Index of triangle's first element within the Entity's Geometry's indices array
    var indices = hit.indices;      // Value of each of the triangle's vertex indices (a three element array)
    var localPos = hit.localPos;    // Local-space position within the triangle
    var worldPos = hit.worldPos;    // World-space position within the triangle
    var viewPos = hit.viewPos;      // View-space position within the triangle
    var bary = hit.bary;            // Barycentric coordinate within the triangle
    var normal = hit.normal;        // Interpolated normal vector within the triangle
    var uv = hit.uv;                // Interpolated UVs within the triangle
}
{% endhighlight %}

{% highlight javascript %}
var entity = hit.entity;
var geometry= entity.geometry;

// Get indices for each of the vertices of the picked triangle
var primIndex = hit.primIndex;
var indexA = geometry.indices[primIndex];
var indexB = geometry.indices[primIndex + 1];
var indexC = geometry.indices[primIndex + 2];

// Another way to get the indices
var indexABC = hit.indices;

{% endhighlight %}

xeogl performs the following steps for this type of picking:
 
1. User ray-picks at given canvas coordinates.
2. Do a render pass to a hidden frame buffer, rendering each Entity with a unique colour. Each colour is the RBGA-encoded 
index of the Entity within xeogl's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the Entity. 
4. Clear the framebuffer, and render a second render pass, this time rendering only the triangles of the picked Entity, 
each with a unique color. Each colour is the RBGA-encoded index of the triangle within the Entity's geometry.
5. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the triangle.
6. Now that we have the Entity and the triangle, make a ray in clip-space from the eye position that passes through 
the near projection plane, then unproject that ray to get a ray in the Entity's local coordinate space.
7. Find the intersection of the ray with the triangle in local space.
8. Find the barycentric coordinates of the local-space intersection, then use those to interpolate within the triangle 
to find the normal vector and UV coordinates at that position.  

For step (4) we lazy-compute some extra vertex position and color arrays for the Entity so that we can render each 
triangle with a unique flat color. It's actually very fast, so you'd hardly notice it, even with big meshes - [here's 
the math function](https://gist.github.com/xeolabs/6e53681b88e00773075e97460a5e7c72).
   
## Picking an entity with a World-space ray

This involves firing a ray through the scene, in World-space, to pick the first entity that intersects it. This is 
really useful when we want to select an entity with a 3D pointing device.
<br>
<br>
[![](http://i.giphy.com/26gJyaa8wBeuvtgac.gif)](http://xeogl.org/examples/#interaction_picking_raycasting) 
   <br>[Click to run](http://xeogl.org/examples/#interaction_picking_raycasting) 
<br>
<br>
Fire a ray through the scene in World-space, to pick the first Entity hit by the ray:

{% highlight javascript %}
var hit = scene.pick({
    origin: [0,0,-5],   // Ray origin
    direction: [0,0,1], // Ray direction
    pickSurface: false   // <<------ Don't want to pick a point on the surface
});

if (hit) { // Picked an Entity with the ray    
     var entity = hit.entity;   // Entity we picked    
}
{% endhighlight %}

xeogl performs the following steps for this type of picking: 
 
1. User ray-picks at given canvas coordinates.
2. Do a render pass to a hidden frame buffer, rendering each Entity with a unique colour. Each colour is the RBGA-encoded 
index of the Entity within xeogl's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the Entity. 
4. Clear the framebuffer, and render a second render pass, this time rendering only the triangles of the picked Entity, 
each with a unique color. Each colour is the RBGA-encoded index of the triangle within the Entity's geometry.
5. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the triangle.
6. Now that we have the Entity and the triangle, make a ray in clip-space from the eye position that passes through 
the near projection plane, then unproject that ray to get a ray in the Entity's local coordinate space.
7. Find the intersection of the ray with the triangle in local space.
8. Find the barycentric coordinates of the local-space intersection, then use those to interpolate within the triangle 
to find the normal vector and UV coordinates at that position.  

## Picking a position on an entity with a World-space ray

Like the previous picking flavour, this also involves firing a ray through the scene in World-space, to pick 
 an Entity, but this time we're also getting information on the ray intersection. This is also useful when we want to select an entity with a 3D pointing device, 
 particularly when we want to draw on the entity's surface.
<br>
<br>
[![](http://i.giphy.com/3o6ZsZ8dQPxc5uQ5wc.gif)](http://xeogl.org/examples/#interaction_picking_raycasting_pickSurface) 
<br>[Run this example](http://xeogl.org/examples/#interaction_picking_raycasting_pickSurface) 
<br>
<br>
Fire a ray through the scene in World-space, to pick a 3D position on the surface of the first Entity hit by the ray:

{% highlight javascript %}
var hit = scene.pick({
    origin: [0,0,-5],   // Ray origin
    direction: [0,0,1], // Ray direction
    pickSurface: true   // <<--------- Indicates that we want to pick on surface
});

if (hit) { // Picked an Entity with the ray

    // Hit result contains the same properties as the previous example    
    
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
}
{% endhighlight %}

xeogl performs the following steps for this type of picking: 
 
1. User ray-picks at given canvas coordinates.
2. Do a render pass to a hidden frame buffer, rendering each Entity with a unique colour. Each colour is the RBGA-encoded 
index of the Entity within xeogl's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the Entity. 
4. Clear the framebuffer, and render a second render pass, this time rendering only the triangles of the picked Entity, 
each with a unique color. Each colour is the RBGA-encoded index of the triangle within the Entity's geometry.
5. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the triangle.
6. Now that we have the Entity and the triangle, make a ray in clip-space from the eye position that passes through 
the near projection plane, then unproject that ray to get a ray in the Entity's local coordinate space.
7. Find the intersection of the ray with the triangle in local space.
8. Find the barycentric coordinates of the local-space intersection, then use those to interpolate within the triangle 
to find the normal vector and UV coordinates at that position.  

<br>

# Conclusion

# Acknowledgements
