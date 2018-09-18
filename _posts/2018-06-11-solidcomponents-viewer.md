---
layout: post
title: SolidComponents™ Online CAD Viewer
client: Cervican  AB, Halmstad,  Sweden
description: Developed the 3D CAD viewer within the <b>SolidComponents™</b> online product catalog.  
thumbnail: solidcomponents/solidcomponents-mac.png
tech: "WebGL, 3DXML, xeogl"
modified: 2018-07-31
category: portfolio
location: "Halmstad, Sweden"
article: true
website: https://www.solidcomponents.com/
websiteReadible: https://solidcomponents.com
tags: [webgl, 3DXML, SolidWorks, CAD, xeogl]
---

 ![]({{ site.url }}/images/gear.gif)
 
## The Client

**[SolidComponents™](https://www.solidcomponents.com)**, based in Halmstad, Sweden, provides an online catalog of 
engineering components.   

## Requirements
 
- Create a WebGL-based viewer for users to interactively view 3D CAD models of products from the client's online catalog
- Load files exported from **[SolidWorks](https://www.solidworks.com/)**
- Realistic, physically-based materials
- Wireframe and transparent viewing modes 

<!-- SolidComponents had already made some experimental viewers on THREE.js to load STL and OBJ files exported from SolidWorks, but  -->
<!-- was having difficulty getting the final appearance of the models right using those file formats. -->
 <!-- <br><br>One of the the trickiest things  -->
<!-- was correctly smooth shading the models, since the face-aligned STL normals are useless for smooth shading, and the  -->
<!-- vertex-aligned OBJ normals often caused hard edges to appear incorrectly smooth.   -->

## Solution

I implemented the SolidComponents CAD viewer as a wrapper class around **[xeogl](http://xeogl.org)**, an open source 
WebGL library I created for 3D visualization.
<br><br>
![]({{ site.url }}/images/solidcomponents/uml.png)

#### 3DXML

I chose **[3DXML](https://en.wikipedia.org/wiki/3DXML)** as the viewer's file format, since it's the native SolidWorks format and represents the model exactly 
as it appears in SolidWorks.    

#### Wireframe

For wireframe views, I'm relying on xeogl to auto-generate wireframe meshes from 3DXML's triangle meshes. 

For each geometry, xeogl creates a second wireframe mesh that contains the edges between adjacent triangles whose surface 
normals deviate from each other by a given threshold, ie. the "hard" edges. This technique eliminates the "inner" edges, 
which are edges shared by triangles that are part of the same faces. 

![]({{ site.url }}/images/solidcomponents/innerEdges.png) | ![]({{ site.url }}/images/solidcomponents/innerEdgesRemoved.png)
----|----
Wireframe with inner edges | Inner edges eliminated

(work-in-progress)

<!-- #### STL models -->

<!-- Initially, I experimented with loading the **[STL](https://en.wikipedia.org/wiki/STL_(file_format))** format, which is also exported by SolidWorks. -->
<!-- <br><br> -->
<!-- ![]({{ site.url }}/images/solidcomponents/gearWhiteBG.png) -->

<!-- Triangles in STL are disjoint, where each triangle has its own separate vertex positions, normals and  -->
<!-- (optionally) colors. This means that you can have gaps between triangles. Normals for each triangle are perpendicular to the triangle's surface, which gives the model a  -->
<!-- faceted appearance by default. The smoothNormals flag causes the STLModel to recalculate its normals, so that each normal's  -->
<!-- direction is the average of the orientations of the triangles adjacent to its vertex. When smoothing, each vertex normal  -->
<!-- is set to the average of the orientations of all other triangles that have a vertex at the same position, excluding those  -->
<!-- triangles whose direction deviates from the direction of the vertice's triangle by a threshold given in smoothNormalsAngleThreshold.  -->
<!-- This makes smoothing robust for hard edges.  -->


<!-- Normals in STL are face-aligned for a  -->
<!-- flat-shaded appearance, so I ignored them and relied on xeogl to auto-generate them per-vertex. To handle hard edges  -->
<!-- correctly, I extended xeogl's auto-generation to -->

<!-- #### OBJ Models -->

<!-- The client had previously made an experimental viewer on THREE.js that loaded OBJ exported by SolidWorks. However, OBJ contained  -->
<!-- vertex normals that rendered every edge as smooth-shaded, even when it was "hard" and should have rendered with a  -->
<!-- crease. -->
<!-- <br><br> -->
  <!--  -->


<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->


