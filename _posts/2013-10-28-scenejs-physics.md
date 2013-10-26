---
layout: post
title: Rigid-Body Physics in SceneJS
description: "Creating rigid-body physical simulations"
tagline: "Creating multi-threaded physical simulations"
modified: 2013-05-31
category: articles
tags: [scenejs, tutorial, physics, simulation]
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

[![](http://scenejs.org/images/physics100Bodies.png)](http://scenejs.org/examples.html?page=physicsBouncingSpheres) | [![](http://scenejs.org/images/physics400Bodies.png)](http://scenejs.org/examples.html?page=physicsBouncingSpheresMultiSystems) | [![](http://scenejs.org/images/physicsBoxes.png)](http://scenejs.org/examples.html?page=physicsBouncingBoxes)
----|----|---
100 bodies in a single system - [Run demo](http://scenejs.org/examples.html?page=physicsBouncingSpheres)  | 400 bodies in four systems - [Run demo](http://scenejs.org/examples.html?page=physicsBouncingSpheres)  | 30 Falling boxes - [Run demo](http://scenejs.org/examples.html?page=physicsBouncingBoxes)


# Overview
SceneJS supports rigid-body physics through a collection of special node types which may be inserted into scene graphs to define physical behaviours (ie. coordinate system transformations) for child nodes. Physical simulations may be run across multiple systems, with each system running within an engine on a separate worker thread. This means that when one physics system is overloaded, it will not starve the rendering thread and other physics systems. The physics engines are pluggable - they are [JigLibJS](https://github.com/supereggbert/JigLibJS) by default, but may be easily replaced with alternatives such as [AmmoJS](https://github.com/kripken/ammo.js), [CannonJS](http://schteppe.github.io/cannon.js/), or even with a proxy to a high-performance server-based engine.

![](http://scenejs.org/images/physics.png)

# Creating a physics body

A ```physics/body``` node creates a body in a physics system and causes its child nodes to translate and rotate  according to the body's movements and interactions with other bodies.
The ```shape``` parameter on this node specifies the shape of the body boundary. We'll describe the different shapes that are available, starting with the ```box``` shape, with which we'll introduce the general parameters of the node. Then we'll describe the other shape types, describing only the parameters specific to each of those shape types.

## Box-shaped body

Example:
{% highlight javascript %}
someNode.addNode({
   type: "physics/body",

   // This optional "systemId" param selects which system to create body in,
   // allowing multiple systems per scene.  The system is created if not yet
   // existing. The body is created in the nameless default system when this
   // param is omitted.
   systemId: "mySystem",

   // Available shapes are "box", "sphere", "plane" etc.
   shape: "box",

   // Center of body
   pos: [-10, 2, 100],

   // Dimension parameters will be shape-specific
   size: [3,1, 2],

   // Mass
   mass: 0.4,

   // The coefficient of restitution (COR) of two colliding objects is a
   // fractional value representing the ratio of speeds after and before
   // an impact, taken along the line of the impact.
   restitution: 0.9,

   // Friction
   friction: 0.3,

   // Velocity vector
   velocity: [0.2,0.3,-0.2],

   // Specify whether or not this body is able to move
   // Unmoving bodies would be things like walls and floors
   movable: true,

   // Child nodes
   nodes: [

      // Child nodes can be anything, but must fit within the dimensions
      // of the physics body. In this case, we have a child box primitive,
      // implemented by the plugin at
      // http://scenejs.org/api/latest/plugins/node/prims/box.js
      {
         type: "prims/box",
         size: [3, 1, 2]
      }
   ]
});
{% endhighlight %}

* There is one default physics system per scene
* Each system has its own physics engine (eg. JigLibJS) that runs in its own Web worker
* Using an optional ```systemId``` property on the ```physics/body``` node, multiple physics systems may be created per scene
* Note that vectors (size, pos, velocity) on the ```physics/body``` and ``prims/box``` are arrays - we're deprecating object formats for those sort of params.
* [```physics/body``` node implementation](http://scenejs.org/api/latest/plugins/node/physics/body.js)

## Sphere-shaped body

Example:
{% highlight javascript %}
someNode.addNode({
   type: "physics/body",

   // Optional physics system ID - fall back on scene's default system
   systemId: "mySystem",

   // Spherical body shape
   shape: "sphere",

   // Center of body
   pos: [-10, 2, 100],

   // Sphere radius
   radius: 1.0,

   // Mass
   mass: 0.4,

   // The coefficient of restitution
   restitution: 0.9,

   // Friction
   friction: 0.3,

   // Velocity vector
   velocity: [0.2,0.3,-0.2],

   // Body is moving
   movable: true,

   // Child nodes
   nodes: [

      // Child sphere primitive, implemented by the plugin at
      // http://scenejs.org/api/latest/plugins/node/prims/sphere.js
      {
         type: "prims/sphere",
         radius: 1
      }
   ]
});
{% endhighlight %}

## Plane-shaped body

Example:
{% highlight javascript %}
someNode.addNode({
   type: "physics/body",

   // Optional physics system ID - fall back on scene's default system
   systemId: "mySystem",

   // Planar body shape
   shape: "plane",

   // Position
   pos:[0, 0, 0],

   // Points upwards
   dir:[0, 1, 0],

   // Let's make it stationary
   velocity:[0, 0, 0],

   // Mass not relevant for unmoving object
   mass:0.0,

   // The coefficient of restitution
   restitution:0.9,

   friction:0.3,

   // Let's make it stationary
   movable:false,

   // Child nodes
   nodes:[

        // Blue color for ground plane
        {
            type:"material",
            color:{ r:0.8, g:0.8, b:1.0 },
            nodes:[

                // Grid ground plane geometry, implemented by plugin at
                // http://scenejs.org/api/latest/plugins/node/prims/grid.js
                {
                    type:"prims/grid",
                    size:{ x:1000, z:1000 },
                    xSegments:200,
                    zSegments:200
                }
            ]
        }
    ]
});
{% endhighlight %}

More body types coming up.

# Body extents
You must ensure that the extents of the ```physics/body``` enclose the space occupied by its child nodes. Ideally, we would do that automatically, expanding and contracting the extents as child nodes are added and removed.

However, the reasons we don't (re)calculated them automatically are:
* It's complex, where there are too many cases to consider (at least at this stage), and we need to keep the JavaScript simple
* The child nodes might be moving, and thus the boundary would be specified for the extents of their movement, (which again, could be hard to compute in advance, or synchronise the body extents with)
* Whatever creates the ```physics/body``` node (eg. a loader, or another custom node type) is likely to be in a position of calculate the extents of child nodes anyway

# Physics materials
Although you can specify properties like ```restitution```, ```friction``` and ```mass``` on individual ```physics/body``` nodes, sometimes you'll want to share those properties among a group of ```physics/body``` nodes.

Just as we can have a parent ```material``` node wrapping multiple child ```geometry``` nodes, we can have a ```physics/material``` node wrapping multiple child ```physics/body``` nodes, so that the children inherit properties from the parent:

{% highlight javascript %}
someNode.addNode({
   type: "physics/material",

   mass:0.0,
   restitution:0.9,
   friction:0.3,

   nodes: [
       {
           type: "physics/body",
           shape: "plane",
           //...
       },
       {
           type: "physics/body",
           shape: "sphere",
           //...
       },
       //...
    ]
});
{% endhighlight %}

Note that spatial properties like ```pos```, ```dir```, ```velocity``` etc. are not supported on a ```physics/material``` node, because those only make sense on an actual physical body.

* [```physics/material``` node implementation](http://scenejs.org/api/latest/plugins/node/physics/material.js)

# Configuring a physics system

The ```physics/system``` node configures a physics system and may appear anywhere within the scene graph.

{% highlight javascript %}
var physicsSystem = someNode.addNode({
      type: "physics/system",
      systemId: "mySystem",  // Optional - selects scene's default system when absent
      gravity: [0, -9.8, 0, 0],
      solver:"ACCUMULATED" // (default) or "FAST", "NORMAL"
})
{% endhighlight %}
We can reconfigure the system any time:
{% highlight javascript %}
physicsSystem.setConfigs({
     gravity: [0, -4.0, 0, 0],
     solver:"FAST"
});
{% endhighlight %}
* When the ```systemId``` is given, the ```physics/system``` node will configure that system, creating it first if not existing yet. When ```systemId``` is absent, the node will configure the scene's default physics system.
* ```physics/system``` nodes are optional - if you create ```physics/body``` nodes for a ```systemId``` (or the scene's
 default system) for which no ```physics/system``` node exists in the scene, then that system will just use its default configuration.
* [```physics/system``` node implementation](http://scenejs.org/api/latest/plugins/node/physics/system.js)

# Physics primitives

There is a collection of convenience nodes, each of which wraps a child ```physics/body```, which in turn wraps a child geometry primitive node, and configures the extents of the ```physics/body``` to enclose the primitive.  These allow you to just create, for example, a teapot with physics behaviours, in one shot:

## Teapot primitive
{% highlight javascript %}
someNode.addNode({
    type: "physics/teapot",
    pos: [-100.0, 30.4, 234.1],
    velocity: [0.2,0.3,-0.2],
    mass: 5,
    //...
});
{% endhighlight %}
* [```physics/teapot``` node implementation](http://scenejs.org/api/latest/plugins/node/physics/teapot.js)

## Box primitive
{% highlight javascript %}
someNode.addNode({
    type: "physics/box",
    pos: [-100.0, 30.4, 234.1],
    velocity: [0.2,0.3,-0.2],
    size: [3.0, 4.0, 2.0],
    mass: 5,
    //...
});
{% endhighlight %}
* [```physics/box``` node implementation](http://scenejs.org/api/latest/plugins/node/physics/box.js)

## Sphere primitive
{% highlight javascript %}
someNode.addNode({
    type: "physics/sphere",
    pos: [-100.0, 30.4, 234.1],
    velocity: [0.2,0.3,-0.2],
    radius: 1,
    latitudeBands: 30,
    longitudeBands: 30,
    wire: false,
    mass: 5,
    //...
});
{% endhighlight %}

* [```physics/sphere``` node implementation](http://scenejs.org/api/latest/plugins/node/physics/sphere.js)

## Plane primitive
{% highlight javascript %}
someNode.addNode({
    type: "physics/plane",
    pos: [-100.0, 30.4, 234.1],
    velocity: [0.2,0.3,-0.2],
    direction: [0,1,0,
    width: 10,
    height: 10,
    widthSegments: 10,
    heightSegments: 10
    wire: false,
    mass: 5,
    //...
});
{% endhighlight %}
* [```physics/plane``` node implementation](http://scenejs.org/api/latest/plugins/node/physics/plane.js)

[More to come..]

# Switching physics engines

**In development**

JigLibJS is the default physics engine, however other supported engines may be selected with a ```physicsEngine``` SceneJS global config. The config will be "jiglibjs" by default, but might also be "cannonjs" or "ammojs" if those are available:

{% highlight javascript %}
SceneJS,setConfigs({
    physicsEngine: "jiglibjs"
});
{% endhighlight %}

Each supported physics engine will be a RequireJS module  within the plugin support library directory, and will export this interface:

{% highlight javascript %}
{

     /** Configure the engine. This can be done at any time.
      * @param configs Key-value map of configs
      */
     setConfigs: function(params) {
     },

     /** Creates a body in the engine and returns an ID for it
      * @params params Body properties - possibly generic across all engines
      * @params callback Callback which return an updated position and a rotation
      *                            matrix each the body is updated
      * @return ID of the new body
      */
     createBody : function (params, callback) {
     };

     /** Updates physics properties an existing body in the engine. Note this
      * does not change the shape of the phycics body, just things  like mass,
      * restitution, velocity, position etc.
      * @params bodyId Body ID
      * @params params Body properties
      */
     updateBody : function (bodyid, params) {
     },

     /** Removes a body from the engine
      */
     removeBody : function (bodyId) {
     },

      /**
       * Integrates this physics system
       */
      integrate : function () {
      }
}
{% endhighlight %}

* [Worker implementation for JigLibJS](http://scenejs.org/api/latest/plugins/lib/physics/worker.js)

# Roadmap

* Ability to enable or disable physics systems so they don't keep running when their bodies are culled from the view (ie. by frustum culling etc)
* An engine (pluggable as shown above) that proxies a high-performance server-side physics system (ie. written in C/C++, not JavaScript)
* [Issues tagged 'physics'](https://github.com/xeolabs/scenejs/issues?direction=desc&labels=Physics&page=1&sort=created&state=open)