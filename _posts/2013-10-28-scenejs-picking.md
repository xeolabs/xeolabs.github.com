---
layout: post
title: Picking in SceneJS
description: "How to click on things in the 3D view"
modified: 2013-05-31
category: articles
comments: true
tags: [scenejs, tutorial, picking, interaction]
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

In this tutorial I'll show you how to

 * pick things in your scenes,
 * make things unpickable,
 * allow picking through transparent objects, and
 * organise things in named hierarchies, so that pick hits reflect the containment hierachy of the object you've picked.

Then you'll be ready to do [ray picking]({{site.url}}/articles/scenejs-ray-picking), which builds on what you'll learn in this tutorial.

## Basic Picking

As usual, we'll point SceneJS at where you keep your plugins:
{% highlight javascript %}
SceneJS.setConfigs({
     pluginPath:"./plugins"
});
{% endhighlight %}

Then we'll create a scene containing the two teapots shown below. Each teapot is wrapped in a ```name``` node, which enables it
to be picked, and assigns it a symbolic name that will identify it in pick results whenever it is picked.
<br><br>
[![Picking Example 1]({{ site.url }}/images/scenejs/basicPicking.png)](http://scenejs.org/examples.html?page=picking)
[Run this demo](http://scenejs.org/examples.html?page=picking)

### Create the scene

{% highlight javascript %}
// Define scene
var myScene = SceneJS.createScene({
    nodes:[

        // Viewing transform
        {
            type:"lookAt",
            eye:{ x:7, y:5, z:10 }, look:{ x:0, y:1, z:0 },
            nodes:[

                // Pickable red teapot with name "redTeapot"
                {
                    type:"name", name:"redTeapot",
                    nodes:[
                        {
                            type:"material",
                            color:{ r:1.0, g:0.3, b:0.3 },
                            nodes:[
                                {
                                    type:"translate", z:2,
                                    nodes:[

                                        // Teapot primitive
                                        {
                                            type:"prims/teapot"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },

                // Pickable blue teapot with name "blueTeapot"
                {
                    type:"name", name:"blueTeapot",
                    nodes:[
                        {
                            type:"material",
                            color:{ r:0.3, g:0.3, b:1.0 },
                            nodes:[
                                {
                                    type:"translate", z:-2,
                                    nodes:[

                                        // Teapot primitive
                                        {
                                            type:"prims/teapot"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

A few things to note:

* A ```name``` node will assign a symbolic name value to whatever is in its subgraph, which can include multiple geometries.
* You can share the same name value across multiple ```name``` nodes. For example, a building model might have lots of windows
scattered around the scene, each having a separate ```name``` node, but all with the value "window".
* Geometries not wrapped in ```name``` nodes will be unpickable. You'd be able to pick through those objects
as if they were not there.
* You can also wrap geometries with ```flags``` nodes to make them unpickable, even if they have ```name``` nodes around them.
That's  useful for switching their pickability on and off.
* You can nest ```name``` nodes, which will give you name paths in your pick results.

### Register a pick handler

Now we'll register a pick handler on our scene, which will be called whenever something gets picked:
{% highlight javascript %}
// Register a pick handler
myScene.on("pick",
    function (hit) {

        // The 'name' property on the first name node on the path
        // from the picked teapot up to the scene root, eg "blueTeapot"
        var name = hit.name;

        // Name path, a '.' delimited concatenation of the 'name' properties
        // of all name nodes on the path down from teh scene root to the
        // picked teapot.
        var path = hit.path;

        // Two-element array containing the X,Y canvas coordinates at which we picked
        var canvasPos = hit.canvasPos;
    });
{% endhighlight %}

We'll also register a handler that gets called when nothing is picked:

{% highlight javascript %}
myScene.on("nopick",
    function (hit) {
        alert("Nothing picked.");
    });
{% endhighlight %}

### Now pick something

Now we can attempt to pick one of our teapots, at some given canvas X and Y coordinates:

{% highlight javascript %}
myScene.pick(45, 150); // X, Y
{% endhighlight %}

Assuming that we picked the blue teapot, the "pick" handler will then return:

{% highlight javascript %}
{
    name: "blueTeapot",
    path: "blueTeapot",
    canvasPos: [45, 150]
}
{% endhighlight %}

## Disabling picking for objects

As mentioned earlier, sometimes we don't objects to be pickable, such as skies and backgrounds.
<br><br>
Here's a variation on the previous scene, in which I've made the green teapot unpickable by wrapping it
with a ```flags``` node which has a ```picking``` flag set to ```false```. I've left the ```name``` node around
it just to emphasise that it's the ```flags``` node that causes picking to be disabled.
<br><br>
[![Disable Picking]({{ site.url }}/images/scenejs/disablePicking.png)](http://scenejs.org/examples.html?page=disablePicking)
[Run this demo](http://scenejs.org/examples.html?page=disablePicking)
{% highlight javascript %}
var myScene = SceneJS.createScene({
    nodes:[
        {
            type:"lookAt", eye:{ x:7, y:5, z:10 }, look:{ x:0, y:1, z:0 },
            nodes:[

                // Pickable red teapot with name "redTeapot"
                {
                    type:"name", name:"redTeapot",
                    nodes:[
                        {
                            type:"material", color:{ r:1.0, g:0.3, b:0.3 },
                            nodes:[
                                {
                                    type:"translate", z:2,
                                    nodes:[

                                        // Teapot primitive
                                        {
                                            type:"prims/teapot"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },

                // Non-pickable green teapot
                // Though it's non-pickable, we'll give it name "greenTeapot"
                {
                    type:"name", name:"greenTeapot",
                    nodes:[

                        // This 'picking' flag disables picking for this teapot
                        {
                            type:"flags",
                            id: "myFlags",
                            flags:{ picking:false },
                            nodes:[
                                {
                                    type:"material", color:{ r:0.3, g:1.0, b:0.3 },
                                    nodes:[
                                        {
                                            type:"translate", z:-2,
                                            nodes:[

                                                // Teapot primitive
                                                {
                                                    type:"prims/teapot"
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

### Toggling pickability for an object
You can make the green teapot pickable by flipping the ```picking``` flag on that ```flags``` node:
{% highlight javascript %}
myScene.getNode("myFlags",
    function(flags) {
        flags.setPicking(true);
    });
{% endhighlight %}

## Picking through transparent objects

Another situation where we'd want to disable picking is when we want to pick through transparent objects.
<br><br>
In this scene I've made the blue transparent box unpickable, just like in the previous example, by wrapping it
with a ```flags``` node which has a ```picking``` flag set to ```false```. As before, I've left a ```name``` node around
it just to emphasise that it's the ```flags``` node that causes picking to be disabled.
<br><br>
[![Pick through transparent objects]({{ site.url }}/images/scenejs/pickThroughTransparentObject.png)](http://scenejs.org/examples.html?page=pickThroughTransparentObject)
[Run this demo](http://scenejs.org/examples.html?page=pickThroughTransparentObject)
{% highlight javascript %}
var myScene = SceneJS.createScene({
    nodes:[
        {
            type:"lookAt", eye:{ x:7, y:5, z:10 }, look:{ x:0, y:1, z:0 },
            nodes:[

                // Pickable red teapot with name "redTeapot"
                {
                    type:"name", name:"redTeapot",
                    nodes:[
                        {
                            type:"material",
                            color:{ r:1.0, g:0.3, b:0.3 },
                            nodes:[

                                // Teapot primitive
                                {
                                    type:"prims/teapot"
                                }
                            ]
                        }
                    ]
                },

                // Transparent blue box that you can pick through
                // Flags make it transparent and unpickable, while
                // material alpha sets degree of opacity
                {
                    type:"name", name:"blueBox",
                    nodes:[
                        {
                            type:"flags", flags:{ transparent:true, picking:false },
                            nodes:[

                                // Transparent material - note the 'alpha'
                                {
                                    type:"material",
                                    color:{ r:0.3, g:0.3, b:1.0 }, alpha:0.4,
                                    nodes:[
                                        {
                                            type:"translate", x:1, y:2,
                                            nodes:[

                                                // Box primitive
                                                {
                                                    type:"prims/box",
                                                    xSize:3, ySize:2, zSize:3
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}


## Pick name hierarchies

As mentioned earlier, you can nest ```name``` nodes, so that when you pick on a geometry you'll get the path of all the
 ```name``` nodes above it, tracing down from the scene root.
 <br><br>
 The objects in the scene shown below are arranged within a hierarchy of ```name`` nodes so that when you pick them, the pick
  result will have name paths like "myScene.table.leg1", and so on.
<br><br>
 [![Name paths]({{ site.url }}/images/scenejs/pickNameHierarchy.png)](http://scenejs.org/examples.html?page=namePaths)
 [Run this demo](http://scenejs.org/examples.html?page=namePaths)
<br><br>
So for example, assuming that we picked one of the table
  legs, the pick result, returned through the "pick" handler we registered earlier, would look like this:

{% highlight javascript %}
{
    name: "leg1",
    path: "myScene.table.leg1",
    canvasPos: [78, 110]
}
{% endhighlight %}

Here's a snippet of this scene that defines the table. The whole scene gets the name "myScene", the table gets the
name "table", while the table's legs get the names "leg1", "leg2", "leg3" and "leg4".

{% highlight javascript %}
//...

// Table
{
    type:"name", name:"table",

    nodes:[
        {
            type:"translate", x:0, y:0, z:0,
            nodes:[

                // Table leg #1
                {
                    type:"name", name:"leg1",
                    nodes:[
                        {
                            type:"translate", x:-2.0, y:0.0, z:-2.0,
                            nodes:[
                                {
                                    type:"scale", x:0.2, y:1.0, z:0.2,
                                    nodes:[
                                        {
                                            type:"material", color:{ r:1.0, g:0.4, b:0.4 },
                                            nodes:[

                                                // Box primitive
                                                {
                                                    type:"prims/box"
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },

                // Table leg #2
                {
                    type:"name", name:"leg2",
                    nodes:[
                        {
                            type:"translate", x:2.0, y:0.0, z:-2.0,
                            nodes:[
                                {
                                    type:"scale", x:0.2, y:1.0, z:0.2,
                                    nodes:[
                                        {
                                            type:"material", color:{ r:0.4, g:0.6, b:0.4 },
                                            nodes:[

                                                // Box primitive
                                                {
                                                    type:"prims/box"
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },

                // Table leg #3
                {
                    type:"name", name:"leg3",
                    nodes:[
                        {
                            type:"translate", x:2.0, y:0.0, z:2.0,
                            nodes:[
                                {
                                    type:"scale", x:0.2, y:1.0, z:0.2,
                                    nodes:[
                                        {
                                            type:"material", color:{ r:0.4, g:0.4, b:0.6 },
                                            nodes:[

                                                // Box primitive
                                                {
                                                    type:"prims/box"
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },

                // Table leg #4
                {
                    type:"name", name:"leg4",
                    nodes:[
                        {
                            type:"translate", x:-2.0, y:0.0, z:2.0,
                            nodes:[
                                {
                                    type:"scale", x:0.2, y:1.0, z:0.2,
                                    nodes:[
                                        {
                                            type:"material", color:{ r:0.6, g:0.4, b:0.6 },
                                            nodes:[

                                                // Box primitive
                                                {
                                                    type:"prims/box"
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },

                // Table top
                {
                    type:"name", name:"top",
                    nodes:[
                        {
                            type:"translate", x:0.0, y:1.2, z:0.0,
                            nodes:[
                                {
                                    type:"scale", x:2.7, y:0.2, z:2.7,
                                    nodes:[
                                        {
                                            type:"material", color:{ r:0.6, g:0.6, b:0.4 },
                                            nodes:[

                                                // Box primitive
                                                {
                                                    type:"prims/box"
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    ]
}
//...
 {% endhighlight %}

## What's next?

* Check out [Ray Picking](/articles/scenejs-ray-picking), which is an extention of basic picking, where we provide a
flag that causes SceneJS to return the World-space coordinates of the point we're picking on the surface of the object.