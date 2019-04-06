---
layout: post
title: Importing .OBJ, .MD2 and .3DS into SceneJS
description: "Importing meshes from three popular formats"
modified: 2013-05-31
category: articles
comments: false
tags: [scenejs, tutorial, importing]
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

I've made three new plugins for SceneJS, to import mesh geometry from these file formats:

* **OBJ** - a lightweight geometry definition file format first developed by Wavefront Technologies which contains mesh
 data — namely, the position of each vertex, the UV position of each texture coordinate vertex, normals, and the faces
 that make each polygon defined as a list of vertices, and texture vertices.

* **MD2** - a model format used by id Software's id Tech 2 engine in games such as Quake II and Soldier of Fortune.
This format is primarily used for animated player models and uses keyframes for mesh positions, which SceneJS interpolates
 within a vertex shader to create smooth animation.

* **3DS** - one of the file formats used by the Autodesk 3ds Max 3D modeling, animation and rendering software. This format has
become a de facto industry standard for transferring models between 3D programs, or for storing models for 3D resource catalogs.

[![]({{ site.url }}/images/scenejs/raptorOBJ.png)](http://scenejs.org/examples.html?page=importObj) | [![]({{ site.url }}/images/scenejs/lionMD5.png)](http://scenejs.org/examples.html?page=importMd2) | [![]({{ site.url }}/images/scenejs/lexus3DS.png)](http://scenejs.org/examples.html?page=import3ds)
----|----|----
[Run .OBJ demo](http://scenejs.org/examples.html?page=importObj) | [Run .MD2 demo](http://scenejs.org/examples.html?page=importMd2) | [Run .3DS demo](http://scenejs.org/examples.html?page=import3ds)


### Usage
The plugins provide new scene node types, and like all plugins, are automatically loaded into the engine as you need them.
<br><br>As usual, we'll start by pointing SceneJS at where you keep your plugins:
{% highlight javascript %}
SceneJS.setConfigs({
     pluginPath:"./plugins"
});
{% endhighlight %}

..then importing an .OBJ is as easy as:
{% highlight javascript %}
SceneJS.createScene({
    nodes:[
        {
            type: "translate", y:-30, z:-200,

            nodes:[
                {
                    type: "texture",
                    src: "foo/bar/raptor.jpg",

                    nodes:[

                        // Import Wavefront .OBJ mesh
                        {
                            type: "import/obj",
                            src: "foo/bar/raptor.obj"
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

Importing meshes from the other formats is similar. To import .MD2, your node would be:
{% highlight javascript %}
{
    type: "import/md2", src: "foo/bar/lion.md2"
}
{% endhighlight %}

And to import .3DS:
{% highlight javascript %}
{
    type: "import/3ds", src: "foo/bar/lexus.3ds"
}
{% endhighlight %}


### Shouts out to:

* [K3D.JS](http://k3d.ivank.net/) - SceneJS uses this library for parsing. It's nice and fast because it loads the
files into typed arrays, which it then parses.
<br><br>