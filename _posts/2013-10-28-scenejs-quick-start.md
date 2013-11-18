---
layout: post
title: SceneJS Quick Start
description: "Get cracking with SceneJS in two minutes"
modified: 2013-05-31
category: articles
comments: true
tags: [scenejs, tutorial, definition]
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

[SceneJS](http://scenejs.org) is an extensible WebGL-based engine for doing high-performance 3D visualisation in the browser
without using plugins.
<br><br>Let's start easy, with an example that shows the essential concepts of this engine.

# First example
We're going to render this spinning Utah teapot:
<br/><br/>

[![SceneJS First Example]({{ site.url }}/images/scenejs/firstExample.png)](http://scenejs.org/examples.html?page=firstExample)

[Run this example](http://scenejs.org/examples.html?page=firstExample).

### Download and unzip SceneJS and plugins

Download and unzip the [SceneJS ZIP archive](http://scenejs.org/api/latest/scenejs.zip). Then we'll have these files:

{% highlight html %}
.
├── scenejs.js
├── plugins
└── firstExample.html
{% endhighlight %}

We have the SceneJS library, the SceneJS plugins bundle, and an example HTML page which renders our teapot.
  <br><br>
Now let's look at the essential bits of that ```firstExample.html``` page.

### Link to the SceneJS library

Within the &lt;head&gt; tag of the page, we link to the SceneJS library:

{% highlight html %}
<script src="./scenejs.js"></script>
{% endhighlight %}

### Point SceneJS to the plugins

Within the &lt;script&gt; tag, the first thing we do is configure SceneJS to find those plugins:

{% highlight javascript %}
SceneJS.setConfigs({
    pluginPath: "./plugins"
});
{% endhighlight %}

### Create a scene

Then we go wild and create our spinning blue teapot:

{% highlight javascript %}
var scene = SceneJS.createScene({
    nodes:[
        {
            type:"material",
            color: { r: 0.3, g: 0.3, b: 1.0 },

            nodes:[
                {
                    type: "rotate",
                    id: "myRotate",
                    y: 1.0, angle: 0,

                    nodes: [

                        // Teapot primitive, implemented by plugin file
                        // ./plugins/node/prims/teapot.js
                        {
                            type:"prims/teapot",
                            id: "myTeapot"
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

### Spin the teapot

Finally, we start the teapot spinning:

{% highlight javascript %}
scene.getNode("myRotate", function(myRotate) {

    var angle = 0;

    scene.on("tick",
        function() {
            myRotate.setAngle(angle += 0.5);
        });
});
{% endhighlight %}

### Replace the teapot

Scenes can be build incrementally. Let's add another teapot

{% highlight javascript %}
scene.getNode("myTeapot", function(myTeapot) {

    myTeapot.destroy();

    scene.getNode("myRotate", function(myRotate) {
        myRotate.addNode({
            type: "prims/torus"
        });
    })
});
{% endhighlight %}


# What's next?

* [Adding and Removing Nodes](/articles/scenejs-creating-a-scene-and-adding-nodes)