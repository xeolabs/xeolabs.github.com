---
layout: post
title: Adding and Removing Scene Nodes
description: "How to create a scene then update, add and remove nodes"
modified: 2013-05-31
category: articles
tags: [scenejs, tutorial]
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

In this article we're going to go a little deeper than we did in the [quick start](quick start) to demonstrate how to
add and remove scene nodes. Although you can define a whole scene one chunk of JSON, SceneJS actually provides you with
node objects that you can create, call methods on, destroy and rearrange.

# Overview
A scene graph is a data structure that arranges the logical and spatial representation of a graphical scene as a collection
of nodes in a graph, typically a tree. Scene graphs can provide a convenient abstraction on top of low-level graphics APIs
(such as WebGL) that encapsulates optimisations and API best practices, leaving the developer free to concentrate on scene content.

A key feature of most scene graphs is state inheritance in which child nodes inherit the states set up by parents
(e.g. coordinate spaces, appearance attributes etc).

For SceneJS, a benefit of scene graphs is modularity, where JSON subtrees can be complete reusable components, and brevity,
where state may be reused by wrapping it around many child nodes.

# Creating a Scene Graph
The snippet below is an example of a 3D scene created with SceneJS. The scene graph is a directed acyclic graph expressed
in JSON, in this case defining a scene containing a blue teapot and two boxes sharing the same textured appearance.
Geometry nodes are normally at the leaves, where they inherit the state defined by higher nodes, in this case the material
and rotation transform.

{% highlight javascript %}
var scene = SceneJS.createScene({
    nodes: [{
        type: "material",
        id: "myMaterial",
        color: {
            r: 0.5,
            g: 0.5,
            b: 1.0
        },

        nodes: [{
            type: "texture",
            layers: [{
                src: "../../textures/superman.jpg"
            }],

            nodes: [{
                type: "translate",
                id: "firstBoxPos",
                x: -3.0,
                y: 1.0,
                z: 5.0,

                nodes: [{
                    type: "prims/box"
                }]
            }, {
                type: "translate",
                y: 1.0,
                z: 5.0,

                nodes: [{
                    type: "prims/box"
                }]
            }]
        }, {
            type: "prims/teapot"
        }]
    }]
});
{% endhighlight %}

SceneJS parses that description to create the 3D scene shown below:
<br/><br/>

[![](http://scenejs.org/images/secondExample.jpg)](http://scenejs.org/examples.html?page=secondExample)

[Run this example here](http://scenejs.org/examples.html?page=secondExample).

# Updating Nodes
We can animate one of those ```translate``` nodes like this:

{% highlight javascript %}
scene.getNode("firstBoxPos",
    function (firstBoxPos) {

        var y = 1.0;
        var inc = 0.02;

        scene.on("tick", function () {
            firstBoxPos.setY(y);
            y += inc;

            if (y > 5 || y < 1.0) {
                inc *= -1.0;
            }
        });
    });
{% endhighlight %}

# Adding Nodes
Adding nodes to the scene is as simple as:

{% highlight javascript %}
scene.getNode("myMaterial",
    function(myMaterial) {

        myMaterial.addNode({
            type: "translate",
                x: 5,
                nodes: [
                    {
                        type: "prims/torus"
                    }
                ]
            }
        });
    });
{% endhighlight %}

# Removing Nodes
Remove nodes like this:

{% highlight javascript %}
scene.getNode("myMaterial",
    function(myMaterial) {

        myMaterial.destroy();
    });
{% endhighlight %}
