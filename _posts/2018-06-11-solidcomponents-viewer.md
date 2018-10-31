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

In 2018, I worked remotely with **[SolidComponents™](https://www.solidcomponents.com)** in Halmstad, Sweden, to develop 
the WebGL-based 3D CAD viewer within their online engineering components catalog.
<br><br>
We developed the CAD viewer on **[xeogl](http://xeogl.org)**, an open source WebGL library I created for developing 
3D model visualization applications in Web browsers without using plugins.
<br><br>
In this article, I'll describe some of the features that I implemented within the CAD viewer.

 ![]({{ site.url }}/images/gear.gif)
 
# Contents

  * [The Client](#the-client)
  * [Requirements](#requirements)
  * [Solution](#solution)
      - [3DXML](#3dxml)
      - [Wireframe](#wireframe)
 
# The Client

**[SolidComponents™](https://www.solidcomponents.com)** is a company based in Halmstad, Sweden that provides an online catalog of 
engineering components. For each component, the catalog provides an image, a table of attributes, and a downloadable 
[3DXML](https://en.wikipedia.org/wiki/3DXML) CAD model that was originally created in SolidWorks.

# Requirements
 
- Create a WebGL-based viewer for users to interactively view the 3D CAD model of each product in the catalog
- Render using realistic, physically-based materials
- Render wireframe and transparent views 

<!-- SolidComponents had already made some experimental viewers on THREE.js to load STL and OBJ files exported from SolidWorks, but  -->
<!-- was having difficulty getting the final appearance of the models right using those file formats. -->
 <!-- <br><br>One of the the trickiest things  -->
<!-- was correctly smooth shading the models, since the face-aligned STL normals are useless for smooth shading, and the  -->
<!-- vertex-aligned OBJ normals often caused hard edges to appear incorrectly smooth.   -->

# Solution

### Using xeogl

I implemented the SolidComponents CAD viewer as a wrapper class around **[xeogl](http://xeogl.org)**, an open source 
WebGL library I created for 3D visualization.
<br><br>
![]({{ site.url }}/images/solidcomponents/uml.png)

### 3DXML

I chose to make the viewer load the downloadable 3DXML model itself, rather than load some other format from SolidWorks, such 
as STL or OBJ. This way, we're guaranteed to have a rendition that closely matches the way the 3DXML model looks in 
SolidWorks. Also, STL and OBJ were not able to express the same materials that we would see in the 3DXML. 
<br><br>
A nice feature of 3DXML is that all the files that comprise the model (ie. scene, geometry and material descriptors) are 
packed into a single ZIP archive that can be loaded in a single HTTP request. Internally, the viewer then unpacks the files within 
the ZIP archive, converts the XML to JSON, and then parses the JSON using a recursive descent parser, to create the various 
3D objects within the xeogl scene graph.

### Wireframe

Often, a 3DXML file contains multiple representations of a model, such as solid-shaded and wireframe. In this 
case, however, we could only rely on it to contain a solid triangle mesh representation. Therefore, for wireframe views, 
I'm relying on xeogl to auto-generate the wireframe representation from the triangle mesh. 

 > For each geometry, xeogl creates a second wireframe mesh that contains the edges between adjacent triangles whose surface 
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



