---
layout: post
title: Importing a Mesh from .OBJ into SceneJS
description: "Using the `import/obj` node to load a mesh from Wavefront OBJ into your scene"
modified: 2013-05-31
category: articles
comments: true
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

[SceneJS](http://scenejs.org) has a new ```import/obj``` node, which lets you import meshes into your scene from Wavefront OBJ files.
In this tutorial I'll describe how I took a duck off the Web, conditioned it in Blender, then made this scene with it:

[![SceneJS OBJ Duck Import]({{ site.url }}/images/scenejs/duckOBJ.jpg)](http://scenejs.org/examples.html?page=importObj)

[Click here to run this example](http://scenejs.org/examples.html?page=importObj).

# First example
We're going to create this duck:
<br/><br/>


### Step 1. Link to the API
Include the [SceneJS library](http://scenejs.org/api/latest/scenejs.js) in the &lt;head&gt; tag of your web page:

{% highlight html %}
<script src="http://scenejs.org/api/latest/scenejs.js"></script>
{% endhighlight %}

### Step 2. Create a scene
We'll make a spinning blue teapot:

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
                    y: 1.0,
                    angle: 0,

                    nodes: [

                        // Teapot primitive, implemented by plugin at
                        // http://scenejs.org/api/latest/plugins/node/prims/teapot.js
                        {
                            type:"prims/teapot"
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

And voilà, there's your blue teapot!

### Step 3. Spin the teapot
Now let's start the teapot spinning:

{% highlight javascript %}
scene.getNode("myRotate", function(myRotate) {

    var angle = 0;

    scene.on("tick",
        function() {
            myRotate.setAngle(angle += 0.5);
        });
});
{% endhighlight %}

# Plugins
To keep the core library small, SceneJS dynamically loads it's non-core functionality from a directory of
plugins. In the example above, the <code>prims/teapot</code> node is a custom node type instantiated
from a <a href="https://github.com/xeolabs/scenejs/tree/V3.1/api/latest/plugins/node/prims/teapot.js">prims/teapot</a>
plugin, which SceneJS loaded on-demand from <a href="https://github.com/xeolabs/scenejs/tree/V3.1/api/latest/plugins">the
plugins directory</a> within its repository on GitHub.

## Serving your own plugins
If you'd rather serve the plugins yourself, then just copy that directory to your server and configure SceneJS to load
them from there, like this:

{% highlight javascript %}
SceneJS.setConfigs({
    pluginPath: "./foo/myPluginsDir"
});
{% endhighlight %}

Want to write your own plugins? Sweet! Please [read more about the plugin API here]({{ site.url}}/articles/scenejs-node-types)
