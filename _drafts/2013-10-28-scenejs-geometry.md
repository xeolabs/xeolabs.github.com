---
layout: post
title: SceneJS Geometry
description: "How to define geometry"
modified: 2013-05-31
category: articles
tags: [scenejs, tutorial, geometry]
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

A ```geometry``` node defines an element of geometry within a scene graph. It supports various primitive types,
from points to triangles, and allows us to share vertex arrays to reduce WebGL calls and redundant vertex use.

### Triangle Meshes

The following example defines a cube, with positions, normals, UV coordinates and indices for triangle primitives.
Although this geometry supports shading and texturing, no [[material]] or [[texture]] nodes are included in this first
example - see the pages for those for more info on how to shade and texture a geometry.

{% highlight javascript %}
{
     type: "geometry",

     // Optional geometry core ID. If some other geometry node with this core
     // has previously been defined in the scene graph then this geometry will just
     // re-use the geometry buffers (IE. vertex buffers etc.) that were created for that.
     coreId: "cube_5_5_5",   // Optional

     // Primitive type - "points", "lines", "line-loop", "line-strip",
     // "triangles", "triangle-strip" or "triangle-fan".
     primitive: "triangles",

     // 3D positions - eight for our cube, each one spanning
     // three array elements for X,Y and Z.
     positions : [
        5, 5, 5, -5, 5, 5,-5,-5, 5, 5,-5, 5,    // v0-v3-v4-v5 right
        5, 5, 5, 5,-5, 5, 5,-5,-5,5, 5,-5,      // v0-v3-v4-v5 right
        5, 5, 5, 5, 5,-5, -5, 5,-5, -5, 5, 5,   // v0-v5-v6-v1 top
        -5, 5, 5, -5, 5,-5, -5,-5,-5, -5,-5, 5, // v1-v6-v7-v2 left
        -5,-5,-5, 5,-5,-5, 5,-5, 5, -5,-5, 5,   // v7-v4-v3-v2 bottom
        5,-5,-5, -5,-5,-5,-5, 5,-5,  5, 5,-5    // v4-v7-v6-v5 back
     ],

     // Optional normal vectors, one for each vertex. If you omit these, then the
     // geometry will not be shaded.
     normals : [
        0, 0, -1, 0, 0, -1, 0, 0, -1,0, 0, -1,  // v0-v1-v2-v3 front
        -1, 0, 0,-1, 0, 0, -1, 0, 0,-1, 0, 0,   // v0-v3-v4-v5 right
        0, -1, 0, 0, -1, 0, 0, -1, 0, 0, -1, 0, // v0-v5-v6-v1 top
        1,  0, 0, 1, 0, 0,1, 0, 0, 1, 0, 0,     // v1-v6-v7-v2 left
        0,  1, 0, 0,1, 0, 0,1, 0, 0,1, 0,       // v7-v4-v3-v2 bottom
        0,  0,1, 0, 0,1, 0, 0,1, 0, 0,1         // v4-v7-v6-v5 back
     ],

     // Optional 2D texture coordinates corresponding to the 3D  positions
     // defined above - eight for our cube, each one spanning two array elements
     // for X and Y. If you omit these, then the geometry will not be textured.
     uv : [
         5, 5, 0, 5, 0, 0, 5, 0, // v0-v1-v2-v3 front
         0, 5, 0, 0, 5, 0, 5, 5, // v0-v3-v4-v5 right
         5, 0, 5, 5, 0, 5, 0, 0, // v0-v5-v6-v1 top
         5, 5, 0, 5, 0, 0, 5, 0, // v1-v6-v7-v2 left
         0, 0, 5, 0, 5, 5, 0, 5, // v7-v4-v3-v2 bottom
         0, 0, 5, 0, 5, 5, 0, 5  // v4-v7-v6-v5 back
     ],

     // Optional coordinates for a second UV layer - just to illustrate their availability
     uv2 : [
     ],

     // Mandatory indices - these organise the positions, normals and uv
     // texture coordinates into geometric primitives in accordance with the "primitive"
     // parameter, in this case a set of three indices for each triangle.
     // Note that each triangle in this example is specified in counter-clockwise
     // winding order. You can specify them in clockwise order if you configure the
     // renderer node's frontFace property as "cw", instead of the default "ccw".
     indices : [
           0, 1, 2, 0, 2, 3,   // Front
           4, 5, 6, 4, 6, 7,   // Right
           8, 9,10, 8,10,11,   // Top
           12,13,14, 12,14,15, // Left
           16,17,18, 16,18,19, // Bottom
           20,21,22, 20,22,23  // Back
     ]
}
{% endhighlight %}

##Vertex Colors

We can also define a colour at each vertex, to interpolate the colors across the faces. These colours will define the
**base colour** which colours defined by the interaction of [[material]] and [[light]] nodes will multiply by, then
[[texture]] nodes will then either multiply or add to the final result.


![x](http://scenejs.wikispaces.com/file/view/vertex-colors.jpg)


To create the colors in the cube above, we would add this colors array to our geometry, which consists of sets of red,
green, blue and alpha values:

{% highlight javascript %}
colors : [

    // v0-v1-v2-v3 front
    1.0, 0.0, 0.0, 1.0, 0.0, 1.0, 0.0, 1.0, 0.0, 0.0, 1.0, 1.0, 0.0, 0.0, 0.0, 1.0,

    // v0-v3-v4-v5 right
    1.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 1.0,

    // v0-v5-v6-v1 top
    1.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 1.0,

    // v1-v6-v7-v2 left
    1.0, 0.0, 0.0, 1.0, 1.0, 1.0, 0.0, 1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 1.0,

    // v7-v4-v3-v2 bottom
    0.0, 0.0, 0.0, 1.0, 1.0, 0.0, 1.0, 1.0, 0.0, 0.0, 1.0, 1.0, 0.0, 0.0, 1.0, 1.0,

    // v4-v7-v6-v5 back
    1.0, 0.0, 1.0, 1.0, 0.0, 1.0, 1.0, 1.0, 1.0, 1.0, 0.0, 1.0, 1.0, 1.0, 1.0, 1.0
]
{% endhighlight %}

## Vertex Sharing

SceneJS allows a mesh to be composed by a hierarchy of geometry nodes, in which the root geometry defines arrays for
positions and uv coordinates, while child geometry's collectively define the primitive index arrays (for faces, lines etc). The syntax is: when a geometry defines positions but no indices, then sub-geometry nodes are expected to supply the indices.

This is a useful technique for optimising scene performance (as used in games such as Quake 3), where it reduces the
number of expensive WebGL buffer re-binds for each frame.

It also allows us to apply different [[material]]s, [[texture]]s, transforms etc. to different portions of the same
mesh. We'll demonstrate how to do that with an example, where we apply two different [[texture]]s to separate portions of a cube-shaped mesh, like this:

![Composite Mesh](http://scenejs.wikispaces.com/file/view/composite-mesh.jpg)

We'll define the same cube as before, but this time composed of three geometry nodes in a hierarchy; the root geometry
defines the position, normal and UV coordinate arrays, while child geometry nodes each define a portion of the face index
list. Furthermore, the child geometry's are each wrapped in a different texture node, so that the portions have different
textures applied. In this example, both child-geometry's are wrapped in the same material.

First, define the root geometry with the position and coordinate arrays:

{% highlight javascript %}
{
    type: "geometry",

    // The vertices - eight for our cube, each
    // one spanning three array elements for X,Y and Z
    positions : [

        5, 5, 5, -5, 5, 5,-5,-5, 5, 5,-5, 5,    // v0-v3-v4-v5 right
        5, 5, 5, 5,-5, 5, 5,-5,-5,5, 5,-5,      // v0-v3-v4-v5 right
        5, 5, 5, 5, 5,-5, -5, 5,-5, -5, 5, 5,   // v0-v5-v6-v1 top
        -5, 5, 5, -5, 5,-5, -5,-5,-5, -5,-5, 5, // v1-v6-v7-v2 left
        -5,-5,-5, 5,-5,-5, 5,-5, 5, -5,-5, 5,   // v7-v4-v3-v2 bottom
        5,-5,-5, -5,-5,-5,-5, 5,-5,  5, 5,-5    // v4-v7-v6-v5 back
    ],

    // Normal vectors, one for each vertex
    normals : [

        0, 0, -1, 0, 0, -1, 0, 0, -1,0, 0, -1,  // v0-v1-v2-v3 front
        -1, 0, 0,-1, 0, 0, -1, 0, 0,-1, 0, 0,   // v0-v3-v4-v5 right
        0, -1, 0, 0, -1, 0, 0, -1, 0, 0, -1, 0, // v0-v5-v6-v1 top
        1,  0, 0, 1, 0, 0,1, 0, 0, 1, 0, 0,     // v1-v6-v7-v2 left
        0,  1, 0, 0,1, 0, 0,1, 0, 0,1, 0,       // v7-v4-v3-v2 bottom
        0,  0,1, 0, 0,1, 0, 0,1, 0, 0,1         // v4-v7-v6-v5 back
    ],

    // 2D texture coordinates corresponding to the
    // 3D positions defined above - eight for our cube, each
    // one spaining two array elements for X and Y
    uv : [

        5, 5, 0, 5, 0, 0, 5, 0, // v0-v1-v2-v3 front
        0, 5, 0, 0, 5, 0, 5, 5, // v0-v3-v4-v5 right
        5, 0, 5, 5, 0, 5, 0, 0, // v0-v5-v6-v1 top
        5, 5, 0, 5, 0, 0, 5, 0, // v1-v6-v7-v2 left
        0, 0, 5, 0, 5, 5, 0, 5, // v7-v4-v3-v2 bottom
        0, 0, 5, 0, 5, 5, 0, 5  // v4-v7-v6-v5 back
    ],
{% endhighlight %}

Next, we'll define the two child geometry's with their shared [[material]] and individual [[texture]]s - note how their
indices arrays address different portions of the root geometry's position and coordinate arrays:

{% highlight javascript %}
    nodes: [
        {
            type: "material",
            baseColor:      { r: 1.0, g: 1.0, b: 1.0 },
            specularColor:  { r: 0.9, g: 0.9, b: 0.9 },
            specular:       0.9,
            shine:          6.0,

            nodes: [
                {
                    type: "texture",
                    src:"images/BrickWall.jpg",
                    applyTo:"baseColor",

                    nodes: [
                        {
                            type: "geometry",

                            /* Indices for first three faces
                             */
                            indices : [
                                0, 1, 2, 0, 2, 3, // Front
                                4, 5, 6, 4, 6, 7, // Right
                                8, 9,10, 8,10,11  // Top
                            ]
                        }
                    ]
                },

                {
                    type: "texture",
                    src:"images/general-zod.jpg",
                    applyTo:"baseColor",

                    nodes: [
                        {
                            type: "geometry",

                            /* Indices for remaining three faces
                             */
                            indices : [
                                12,13,14, 12,14,15, // Left
                                16,17,18, 16,18,19, // Bottom
                                20,21,22, 20,22,23  // Back
                            ]
                        }
                    ]
                }
            ]
        }
    ]
}
{% endhighlight %}
