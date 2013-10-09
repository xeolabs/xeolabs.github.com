---
layout: post
title: SceneJS Detail Culling
description: "Optimising your scenes with detail culling"
tagline: "Optimising your scenes with detail culling"
modified: 2013-05-31
category: articles
tags: [scenejs, tutorial, culling]
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

[![](http://scenejs.org/images/lod1.png)](http://scenejs.org/examples.html?page=frustumDetailCulling) | [![](http://scenejs.org/images/lod2.png)](http://scenejs.org/examples.html?page=frustumDetailCulling) | [![](http://scenejs.org/images/lod3.png)](http://scenejs.org/examples.html?page=frustumDetailCulling) | [![](http://scenejs.org/images/lod4.png)](http://scenejs.org/examples.html?page=frustumDetailCulling)
----|----|----|---
Distant bodies culled | Bodies come closer | Detail level increases | Further detail increase

[Click here to run this example](http://scenejs.org/examples.html?page=frustumDetailCulling)

# Overview
SceneJS supports detail culling via the ```frustum/lod``` node, which sets up an axis-aligned bounding box around its child nodes. Each child provides a different level of detail for the object within the boundary, and the ```frustum/lod``` enables one of its children at any instant in order to show the appropriate detail level for the current projected 2D canvas size of the boundary.

The ```frustum/body``` node can be configured to disable all its children when the boundary falls outside the view frustum. This is an optional configuration because the node might potentially be used to switch detail for non-visible objects, such as sound sources, which could influence the scene even when they fall outside the frustum. Note that the node disables all children at the lowest level of detail, effectively working as a sort of distance culling.

Internally, culling is distributed across multiple culling engines, each running in its own web worker. This attempts to make culling responsive, while preventing it from starving the main rendering thread.

# Creating a detail-switched object

The ```frustum/lod``` node encloses its child nodes in an axis-aligned  World-space bounding box and enables the appropriate child for the box's current projected 2D canvas size.

As shown in the example below, we configure the node with a 2D size threshold for each child node, given in property ```sizes```. When the boundary's projected size falls below the first of these thresholds, then no child will be enabled. When the projected size is between the first and second thresholds, the first child is enabled, then when between the second and third thresholds, the second child is enabled, and so on.

You can have an unlimited number of child nodes and size thresholds.

By default, all of the children are disabled when the bounding box falls outside the view frustum, but as mentioned above, we can use the optional ```frustumCull``` property to disable that behaviour and cull the children solely on the projected box size, which actually works regardless of which direction the box is in with regard to the viewing direction. This is useful for disabling distant sound effects etc.

##Example:
{% highlight javascript %}
someNode.addNode({
   type: "frustum/lod",

   // World-space axis-aligned bounding box
   min: [xPos - 8, yPos - 3, zPos - 5],
   max: [xPos + 8, yPos + 6, zPos + 5],

   // Size thresholds, one for each child node, in ascending
   // order for size, unlimited number
   sizes: [
      50, 250, 350 // More as required...
   ],

   // Option to show the boundary as a wireframe box for debugging - default false
   showBoundary: false,

   // Option to disable all child nodes when the bounding box
   // falls outside the view frustum - true by default
   frustumCull: true,

   // A child node for each size, in ascending order of
   // detail, unlimited number
   nodes: [

      // Detail level #1 (lowest)
      // A light blue box

      {
         type: "material",
         color: {
            r: 0.6,
            g: 0.6,
            b: 1.0
         },

         nodes: [

            {
               type: "translate",
               x: xPos,
               y: yPos,
               z: zPos,

               nodes: [{
                  type: "scale",
                  x: 4,
                  y: 3,
                  z: 4,
                  nodes: [

                     // Box primitive, implemented by plugin at
                     // http://scenejs.org/api/latest/plugins/node/prims/box.js
                     {
                        type: "prims/box"
                     }
                  ]
               }]
            }
         ]
      },

      // Detail level #2 (middle)
      // A medium-blue sphere

      {
         type: "material",
         color: {
            r: 0.5,
            g: 0.5,
            b: 1.0
         },

         nodes: [{
            type: "translate",
            x: xPos,
            y: yPos,
            z: zPos,

            nodes: [{
               type: "scale",
               x: 1.0,
               y: 0.7,
               z: 1.0,

               nodes: [

                  // Sphere primitive, implemented by plugin at
                  // http://scenejs.org/api/latest/plugins/node/prims/sphere.js
                  {
                     type: "prims/sphere",
                     radius: 5,
                     latitudeBands: 16, // A fairly low-rez sphere
                     longitudeBands: 16
                  }
               ]
            }]
         }]
      },

      // Detail level #3 (highest)
      // A dark blue teapot

      {
         type: "material",
         color: {
            r: 0.3,
            g: 0.3,
            b: 1.0
         },

         nodes: [{
            type: "translate",
            x: xPos,
            y: yPos - 2.5,
            z: zPos,

            nodes: [{
               type: "scale",
               x: 2.5,
               y: 2.5,
               z: 2.5,

               nodes: [

                  // Teapot primitive, implemented by plugin at
                  // http://scenejs.org/api/latest/plugins/node/prims/teapot.js
                  {
                     type: "prims/teapot"
                  }
               ]
            }]
         }]
      }

      // Mode children as required..
   ]
});
{% endhighlight %}

# Limitations

* Boundaries can't be transformed yet - they are fixed in World-space.
* Boundaries don't automatically fit their child nodes. That's because they might enclose things for which we might not be able
to automatically determine the boundaries of, such as sound. It's up to the scene definer (human, generator or plugin) to
  keep track of the boundaries of things, which seemed a reasonable tradeoff to reduce performance and complexity.

# Implementation

Here's some details on the internals, just in case you're curious how it works:
![](http://scenejs.org/images/frustumCulling.png)

* The ```frustum/lod``` node is implementated by this [custom node plugin](http://scenejs.org/api/latest/plugins/node/frustum/lod.js).
* For performance, culling is distributed across multiple [culling engines](http://scenejs.org/api/latest/plugins/lib/frustum/frustumCullEngine.js), each running within its own [worker thread](http://scenejs.org/api/latest/plugins/lib/frustum/frustumCullWorker.js). Each of those workers is proxied by a [system](http://scenejs.org/api/latest/plugins/lib/frustum/frustumCullSystem.js) in which the ```frustum/lod``` nodes create frustum collision bodies, and those systems are managed in a load-balanced [pool](http://scenejs.org/api/latest/plugins/lib/frustum/frustumCullSystemPool.js).
* The ```frustum/lod``` node actually wraps a more basic [```frustum/body```](http://scenejs.org/api/latest/plugins/node/frustum/body.js) node type, which creates the collision body and subscribes to its frustum intersection status and projected 2D boundary size. The ```frustum/lod``` node does the job of translating those updates into child node enables/disables. This strategy allows us to reuse the ```frustum/body``` for internal workings of new types of culling node in future.


# Roadmap

* Allow ```frustum/cull``` boundaries to be defined in Model-space, where they are transformed by parent transform nodes
* Other types of boundary shape, such as spherical
* Each culling engine simply iterates over a list of bodies - improve their performance by organising the bodies in KD-trees
* [Issues tagged 'culling'](https://github.com/xeolabs/scenejs/issues?direction=desc&labels=Culling&page=1&sort=created&state=open)