---
layout: post
title: Normal Mapping Morphs in SceneJS
description: "How to make morphing meshes look lumpy"
modified: 2016-3-16
category: articles
comments: true
tags: [scenejs, tutorial, textures, morphing]
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

I just added support for [normal maps](https://en.wikipedia.org/wiki/Normal_mapping) on morphing geometry 
in [SceneJS](http://scenejs.org). Normal mapping is an implementation of bump mapping, 
which fakes the lighting of bumps on a mesh by applying a texture that effectively perturbs the normal vectors that the 
fragment shader interpolates across the mesh. Click the GIF below to try it out.
<br><br>

[![SceneJS Morphing Normals]({{ site.url }}/images/scenejs/morphingNormals.gif)](http://scenejs.org/examples/index.html#geometry_morphTargets_normalMap)
<br>
 
## Implementation
The actual perturbation happens in tangent space within the fragment shader. For static meshes, I'm precomputing the 
local-space tangent for each mesh vertex and passing those into the shader as an array attribute. Then within the vertex 
shader, I'm building the tangent space (TBN) matrix, multiplying the View-space eye position by that, then passing the 
tangent-space eye position through to the fragment shader as a ````varying````. Within the vertex shader I'm also 
transforming each local-space vertex tangent by the Model and View matrices and likewise passing that through to the 
fragment shader as a ````varying````.
<br><br> 
Then over in the fragment shader, I'm computing the bitangent from the cross-product of the tangent and the 
View-space normal. Then from there, I'm (re)computing the per-fragment TBN matrix from the tangent, the bitangent and 
the View-space normal, which I then use to transform each fragment -&gt; light vector into tangent space in preparation 
for the Lambertian shading calculation.   
    
I'm precomputing the tangents so that I don't have to compute them on-the-fly in the fragment shader, which I'm thinking 
might be a bit heavy on mobile devices. 

SceneJS morphGeometry nodes work by defining a sequence of target arrays for geometry vertex positions and 
normals, which we then interpolate between within the vertex shader to obtain the vertex position and normal. As we 
update a morphGeometry with the current animation time, it finds the pair of positions arrays and normals arrays that 
enclose the time and loads those into the shader, where interpolation involves a ````mix```` between the element for each 
vertex.  
 
So to support tangents without computing them on-the-fly in the shader, I'm simply pre-computing an array of vertex 
tangents for each morph target  
    

[Run this example](http://scenejs.org/examples/index.html#geometry_morphTargets_normalMap).

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
                        // ./plugins/node/geometry/teapot.js
                        {
                            type:"geometry/teapot",
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

Scenes can be built incrementally. Let's add another teapot

{% highlight javascript %}
scene.getNode("myTeapot", function(myTeapot) {

    myTeapot.destroy();

    scene.getNode("myRotate", function(myRotate) {
        myRotate.addNode({
            type: "geometry/torus"
        });
    })
});
{% endhighlight %}


# What's next?

* [Adding and Removing Nodes](/articles/scenejs-creating-a-scene-and-adding-nodes)


