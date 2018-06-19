---
layout: post
title: SolidComponents™ Online CAD Viewer
client: Cervican  AB, Halmstad,  Sweden
description: Built the online 3D CAD viewer for the <b>SolidComponents™</b> product catalog.  
thumbnail: solidcomponents/solidcomponents-mac.png
tech: "WebGL, 3DXML, xeogl"
modified: 2017-06-31
category: portfolio
location: "Halmstad, Sweden"
website: https://www.solidcomponents.com/
websiteReadible: https://solidcomponents.com
tags: [webgl, SolidWorks, CAD, xeogl]
---

# The Client



 
 Read about how component suppliers improve sales by taking advantage of SolidComponentstm digital product catalogues with CAD-support.

# Requirements 

Interactively view 3D models exported from SolidWworks

Viewer seamlessly integrated into SolidComponents web page

Allow to distribute with SSolidWorks integration software - a JavaSCript library called "SolidComponents Toolbox", that 
customers use to integrate SolidWork's CAD support into their own web pages.

The client had already made several viewers for STL and OBJ files using THREE and other WebGL libraries, but had 
experienced difficulty controlling the way that smooth shading was applied across the surfaces of meshes

- Needed  smooth shading, but not smooth across hard edges
- Auto conversion of face-aligned normals to vertex aligned was not an option

- I solved with a custom smoothing algorithm (TODO)
- Requirement: view models exported from SolidWorks

# First Attempt:  STL files

- First attempt by importing STL using this shading algorithm
- Smoothing algorithm:  
Triangles in STL are disjoint, where each triangle has its own separate vertex positions, normals and (optionally) colors. 
This means that you can have gaps between triangles. Normals for each triangle are perpendicular to the triangle's surface, 
which gives the model a faceted appearance by default. The smoothNormals flag causes the STLModel to recalculate its normals, 
so that each normal's direction is the average of the orientations of the triangles adjacent to its vertex. When smoothing, 
each vertex normal is set to the average of the orientations of all other triangles that have a vertex at the same position, 
excluding those triangles whose direction deviates from the direction of the vertice's triangle by a threshold given in 
smoothNormalsAngleThreshold. This makes smoothing robust for hard edges.

- Differentiate objects on vertex colors
- STL materials were not expressive enough
- Not possible to preserve colors as modeled in  SolidWworks
- STLModel them open sourced as part of xeogl extras

# OBJ

- not expressive enough

#glTF

- export plugin available from SIMLab, but not core part of SolidWworks, so not enough control.

# Second attempt: loading 3DXML

- 3DXML is native SolidWworks format
- Built a custom component that extends the xeogl native xeogl.Model component to parse 3DXML 4.2 & 4.3 schema
- Smothing with hard edges supported in 3DXML,  where hard edges lie between  separate meshes with smooth per-vertex normals
- Edges and Wireframe representations required 
- Solidworks 2017 export had an edges representation bundled with models
- SolidWorks 2010 export had no edges representation
- xeogl has baked-in solution, automatically generating wireframe views using ghost and highlight materials
- xeogl  CameraControl has all required interaction



<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->


