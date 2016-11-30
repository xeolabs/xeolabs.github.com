---
layout: post
title: GPU-Assisted Picking in xeogl
description: "How picking works in xeogl"
modified: 2016-19-09
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

**[xeogl](http://xeogl.org)** is a WebGL-based engine for 3D visualization on the Web. In this 
article, I'm going to describe xeogl's GPU-assisted **picking** system, which allows you to efficiently select entities 
in complex 3D scenes.  
 
# Overview

xeogl supports four basic types of picking, which I'll describe below:
 
 1. Picking an [Entity](http://xeogl.org/docs/classes/Entity.html) at canvas coordinates
 2. Picking a position on the surface of an Entity, at canvas coordinates
 3. Picking an Entity with a World-space ray
 4. Picking a position on the surface of an Entity with a World-space ray
 
When we pick just the Entity (types 1 and 3), the result contains just the Entity, nothing else. When we pick the **surface** 
of an Entity (types 2 and 4), the result will also contain information about the **position** on the surface that we picked, 
such as the triangle, barycentric coordinates within the triangle, interpolated position, normal vector, UV etc., which is
super useful for fun things like drawing on entities, attaching decals, and manipulating entities with stylus input 
devices (like I'm doing for [zSpace support](http://xeogl.org/docs/classes/ZSpaceEffect.html)).

<br>

# Picking entities at canvas coordinates 

The most basic type of picking involves finding the closest Entity at the given canvas coordinates. This is equivalent to 
firing a ray through the canvas, down the negative Z-axis, to find the closest intersecting Entity. However, this type of 
  picking only finds the Entity and does not return any information about the ray intersection.
<br>  
[![](http://xeogl.org/assets/images/picking2DCanvas.gif)](http://xeogl.org/examples/#interaction_picking) 
   <br>[Click to run](http://xeogl.org/examples/#interaction_picking) 
<br><br>
To start with, let's create a [Model](http://xeogl.org/docs/classes/GLTFModel.html) 
component that loads a glTF model of a gearbox. We'll use this scene in all our examples.
 
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
 
 Now let's attempt to pick the closest Entity at the given canvas coordinates:  
 
 {% highlight javascript %}
 var hit = gearbox.scene.pick({  
     canvasPos: [500,400]
 });
 {% endhighlight %}
 
 If we hit an Entity, then xeogl will return a hit result containing a reference to the Entity:
  
 {% highlight javascript %}
 if (hit) { // Picked an Entity
     var entity = hit.entity;     
 }
 {% endhighlight %}

Internally, xeogl performs the following steps for this type of picking:
 
1. User picks at given canvas coordinates.
2. Do a render pass to a hidden frame buffer, rendering each Entity with a unique colour. Each colour is the RBGA-encoded 
index of the Entity's position within xeogl's internal display list.
3. Read the colour from the framebuffer at the canvas coordinates, map the colour back to the Entity. 

<br>

# Picking entity surfaces at canvas coordinates 

As with the previous example, this type of picking fires a ray through the canvas, down the negative Z-axis, 
 to pick the first Entity that intersects the ray. However, this time we'll get some geometric information about the 
 intersection.
 <br>  
[![](http://xeogl.org/assets/images/picking3DCanvas.gif)](http://xeogl.org/examples/#interaction_picking_triangles_normalAndUV) 
   <br>[Click to run](http://xeogl.org/examples/#interaction_picking_triangles_normalAndUV) 
<br><br>
Reusing the scene that we created for the previous example, we'll now fire a ray through the canvas coordinates, this time 
 supplying a ````pickSurface```` flag, causing it to pick a 3D **position** on the surface of the Entity:  

{% highlight javascript %}
var hit = gearbox.scene.pick({   
    canvasPos: [500,400],
    pickSurface: true,    // <<--------- Indicates that we want to pick on surface
});
{% endhighlight %}

This time, in addition to the Entity, the hit result will contain information about the position that we picked on the Entity's surface:   

{% highlight javascript %}
if (hit) { // Picked an Entity

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

 * For step (4) we lazy-compute some extra vertex position and color arrays for the Entity so that we can render each 
 triangle with a unique flat color. It's actually very fast, so you'd hardly notice it, even with big meshes - [here's 
 the math function](https://gist.github.com/xeolabs/6e53681b88e00773075e97460a5e7c72).   
<br>

# Picking entities with a World-space ray

World-space ray casting involves fire an arbitrarily-positioned ray through the scene in World-space, to pick the first 
Entity that intersects the ray. This is usually done when we want to interact with a scene using a 3D stylus input device.

 <br>  
[![](http://xeogl.org/assets/images/picking3DRayEntity.gif)](http://xeogl.org/examples/#interaction_picking_triangles_normalAndUV) 
   <br>[Click to run](http://xeogl.org/examples/#interaction_picking_triangles_normalAndUV) 
<br><br>
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

<br>

# Picking entity surfaces with a World-space ray

World-space ray casting involves fire an arbitrarily-positioned ray through the scene in World-space, to pick 
 the first Entity that intersects the ray. This is usually done when we want to interact with a 
  scene using a 3D stylus input device.<br><br>
[![]({{ site.url }}/images/xeogl/worldRayPicking.gif)](http://xeogl.org/examples/#interaction_picking_raycasting_triangles) 
<br>[Click to run](http://xeogl.org/examples/#interaction_picking_raycasting_triangles) 
<br><br>
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
