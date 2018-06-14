---
layout: post
title: Migrating from SceneJS 2.0 to 3.1
description: "Updating your apps to the V3.1 API"
modified: 2013-05-31
category: articles
comments: false
tags: [scenejs, tutorial, migrating]
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

**UNDER CONSTRUCTION**
<br><br>
This will be a work-in-progress over the next few weeks, primarily to help out the [BIMSurfer](http://bimsurfer.org) crew migrate
their IFC viewer over to the latest SceneJS API. When it's done, we should have a useful migration document. Check back soon.

# Scene graph construction

Create a scene graph like this:

{% highlight javascript %}
var myScene = SceneJS.createScene({
    id: "myScene,
    canvasId: "myCanvas",
    nodes: [
    ]
});
{% endhighlight %}

* The scene starts immediately - no need to call ```myScene.start()```.
* ```canvasId``` is optional - when you omit it, SceneJS will create a canvas for the scene, expanded to fit the browser window
* [Basic scene example](http://scenejs.org/examples.html?page=firstExample)

# Materials

The ```material``` node's ```baseColor``` attribute has been deprecated - now use ```color``` instead, ie:

{% highlight javascript %}
var myMaterial = myScene.addNode({
    type: "material",
    emit: 0,
    color:      { r: 0.5, g: 0.5, b: 0.6 },
    specularColor:  { r: 0.9, g: 0.9, b: 0.9 },
    specular:       1.0,
    shine:          70.0,
    nodes: [
         //...
    ]
});
{% endhighlight %}


# Lights

### Previously in V2.X
In V2.X, each light source was:

 * a separate node,
 * which you could nest,
 * and wrap in modelling transforms to move them around.

{% highlight javascript %}
myScene.addNode({
    type: "node",
    nodes: [
        {
            type: "light",
            id: "mylight",
            mode:                   "dir",
            color:                  { r: 1.0, g: 1.0, b: 1.0 },
            diffuse:                true,
            specular:               true,
            dir:                    { x: 1.0, y: -0.5, z: -1.0 }
        },
        {
            type: "light",
            mode:                   "dir",
            color:                  { r: 1.0, g: 1.0, b: 1.0 },
            diffuse:                true,
            specular:               true,
            dir:                    { x: 1.0, y: -1.0, z: -1.0 }
        },
        {
            type: "node",
            nodes: [
                {
                    type: "translate",
                    x: 100, y: 200, z: 300,
                    nodes: [
                        {
                            type: "light",
                            mode:                   "dir",
                            color:                  { r: 1.0, g: 1.0, b: 1.0 },
                            diffuse:                true,
                            specular:               true,
                            dir:                    { x: 1.0, y: -1.0, z: -1.0 }
                        }
                    ]
                },
                //..
            ]
        }
        //...
    ]
});
{% endhighlight %}

### Now in V3.X
In V3.X, light nodes have been simplified as shown below, where a single ```lights``` node defines multiple light
sources for its subgraph. The ```lights``` node:

* cannot be nested - a child ```lights``` node overrides the effect of parent ```lights```
* wrapping in modelling transforms has no effect


{% highlight javascript %}
myScene.addNode({
    type: "node",
    nodes: [
        {
            type: "lights",
            id: "myLights",
            lights: [
                {
                    mode:                   "dir",
                    color:                  { r: 1.0, g: 1.0, b: 1.0 },
                    diffuse:                true,
                    specular:               true,
                    dir:                    { x: 1.0, y: -0.5, z: -1.0 }
                },
                {
                    type: "light",
                    mode:                   "dir",
                    color:                  { r: 1.0, g: 1.0, b: 1.0 },
                    diffuse:                true,
                    specular:               true,
                    dir:                    { x: 1.0, y: -1.0, z: -1.0 }
                },
                //..
            ]
        }
        //...
    ]
});
{% endhighlight %}

These restrictions greatly simplify the engine and provide improved performance.

* [Lighting examples](http://scenejs.org/examples.html?tags=lighting)

### Ambient lighting now supported

If you define a light source of type "ambient" it will provide ambient lighting throughout the entire scene.

* [Ambient lighting example](http://scenejs.org/examples.html?page=ambientLight&showCode=true)

# Geometry

All of the geometry primitive nodes are now provided as plugins. In V2.X, you would create a box primitive like this:

{% highlight javascript %}
var myMaterial = myScene.addNode({
    type: "box",
    //...
});
{% endhighlight %}

Now in V3.X you create it using a plugin:

{% highlight javascript %}
var myMaterial = myScene.addNode({
    type: "geometry/box",
    //...
});
{% endhighlight %}

* [Read more about the plugin API here]({{ site.url}}/articles/scenejs-node-types)
* [Live examples tagged "prims"](http://scenejs.org/examples.html?tags=prims)

# Defaults
V3.X provides many more defaults and training wheelss You can leave out just about any kind of non-content node, such
as ```lookAt```, ```camera```, ```lights```, ```material``` etc. and it will provide internal defaults for those.
This means that you can create a scene containing nothing but a ```geometry``` node and that appear nicely aligned in the
 view volume, with perspective projection, illuminated with default light source and coloured white. This is good for examples,
    allowing us to specify only the nodes we want to demonstrate in those example scenes.

# Attribute name changes
TODO

# Removed nodes

* ```billboard```
* TODO

# Deprecations

### The 'renderer' node
The ```renderer``` node configures misceleaneous WebGL capabilities for its subgraph, such as line width, color buffer, depth buffer etc.
  It will be replaced by a collection of more specific nodes, eg. ```depthBuf``` and ```colorbuf```.
