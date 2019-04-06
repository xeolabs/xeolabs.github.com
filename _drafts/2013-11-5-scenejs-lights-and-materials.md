---
layout: post
title: Lights and Materials in SceneJS
description: "All about texturing in SceneJS"
modified: 2013-05-31
category: articles
comments: false
tags: [scenejs, tutorial, texture]
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

In this tutorial I'll show you how define light in SceneJS and how your geometries reflect it. I'll assume that you've already read the
[Quick Start](/articles/scenejs-quick-start) tutorial, which describes how geometry nodes "inherit" state from other nodes
above the the scene graph.

# The shading model

Four types of scene graph node work to together to define the surface appearance of the geometries in their subgraphs:

   * **Lights** define light sources
   * **Materials** define how geometry reflects our light sources (as well as glow and opacity)
   * **Textures** apply patterns
   * **Flags** selectively enable or disable various aspects of shading and texture

In this particular tutorial I'll show you everything here except for ```texture```, which is such a big topic that I'll cover
it in other dedicated tutorials.

### Lights

Light sources come in three types:

 * **Point** lights cast illumination outward in every direction from a single position. These are useful for simulating omnidirectional
light sources such as lightbulbs etc.
* **Directional** have a direction, but no specific position. They represent an extremely distant light source, such as the sun or the moon.
 Rays cast from directional lights run parallel, in a single direction.
* **Ambient** lights cast rays in every direction, and are used to elevate the overall level of diffuse illumination in a scene. Unlike
point and directional lights, ambient lights have no position or direction.

Point and directional lights can be defined in either of two coordinate spaces:

* In **world** space, their positions or directions will be fixed within the world, so that as you move your eye or
direction of view, they will appear to move around you.
* In **view** space (default) they will behave as if attached to your head, so that as you move your eye or direction of view,
they move in synch.

Point and directional lights can also be configured to contribute to specular and/or diffuse lighting.

* When **specular**, they will provide light that bounces acutely off surfaces, to provide sharp glinty highlights.
* When **diffuse**, they will provide light that diffuses more when it bounces off surfaces, for a softer diffuse shading.

Point lights can also have an attenuation factor, which causes their intensity to reduce over distance. Three types
of attenuation are supported:

* **constant**
* **linear**
* **quadratic**

### Materials

As I mentioned above, the exact effect that our ```lights``` nodes have on the surfaces of our geometries will be governed by
the ```material``` and ```flags``` nodes that are also present above those geometries.

A ```material``` node has these properties

* ```specular``` governs the amount of specular light the surface reflects
* ```color``` is the basic color of the geometry
* ```specularColor``` controls the intensity of the glow for the shiny spots, small values represent soft glows, whereas high values define smaller and sharper highlights
* ```shine```
* ```emit``` - the amount of
* ```alpha``` - in conjunction with a ```transparent``` flag, this property controls the transparency of the associated
geometry. If a value of 0.0 is specified then the related geometry is completely transparent, a value of 1.0 means that the
geometry is completely transparent.

### Flags


# Default lights and material
Recall that SceneJS provides sensible defaults for everything in your scene, except geometries and textures.  Here's what
you get when you don't specify any ```lights``` and ```material``` for your geometry:<br><br>
[![Point World space lights]({{ site.url }}/images/scenejs/defaultLightAndMaterial.jpg)](http://scenejs.org/examples.html?page=defaultLightAndMaterial)
[Run this demo](http://scenejs.org/examples.html?page=defaultLightAndMaterial)<br><br>
SceneJS basically provides:

 * two directional white light sources in World-space, one contributing specular and diffuse light, the other contributing just specular,
 * a gray ambient light source, and
 * a gray material that reflects both specular and diffuse light.

It's a pretty workable set of defaults, providing a base that we can modify as required.
<br><br>
Here's all the code needed to create the scene shown above:
 {% highlight javascript %}
 var scene = SceneJS.createScene({
     nodes:[
         {
             type:"cameras/orbit", // Mouse-orbited camera for fun
             yaw:30, pitch:-30, zoom:10, zoomSensitivity:5,

             nodes:[
                 {
                     type:"geometry/teapot"
                 }
             ]
         }
     ]
 });
 {% endhighlight %}

As I show you around the lighting and materials system, I'm going to use this scene as a base, adding stuff to override
defaults as I show you various features.

# A point light

Let's take a look at point lights. I'll override the default lights with a single reddish point light, defined in World-space, that
contributes both specular and diffuse light, with an intensity that attenuates quadratically with distance.<br><br>
[![Point World space lights]({{ site.url }}/images/scenejs/pointLightWorld.png)](http://scenejs.org/examples.html?page=pointLightWorld)
[Run this demo](http://scenejs.org/examples.html?page=pointLightWorld)<br><br>
Gone is that nice ambient grey background glow. Here's the scene definition:

{% highlight javascript %}
var scene = SceneJS.createScene({
    nodes:[
        {
            type:"cameras/orbit",
            yaw:30, pitch:-30, zoom:10, zoomSensitivity:5,

            nodes:[

                // Override default lights with a single point
                // light source in World-space.

                // Since our light is in World-space, it will move
                // relative to our changing viewpoint, as if it
                // were a fixture in the scene.
                {
                    type:"lights",
                    lights:[
                        {
                            mode:"point",
                            color:{ r:1.0, g:0.8, b:0.8 },
                            diffuse:true,
                            specular:true,
                            pos:{ x:2.0, y:2.5, z:2.0 },
                            constantAttenuation:0.0, // [0..1]
                            linearAttenuation:0.0, // [0..1]
                            quadraticAttenuation:0.02, // [0..1]
                            space:"world"
                        }
                    ],

                    nodes:[
                        {
                            type:"geometry/teapot"
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

Things to note:

* The ```lights``` node provides a single point light in World space
* World space means that the light is fixed in the 3D world, so we can move and turn towards and away from it
* The source provides reddish diffuse and specular illumination
* The intensity of the source will attenuate quadratically over distance
* Although I don't show it in the code, I've added a sphere to show the position of the light source

## Updating lights

# Multiple lights

A ```lights``` node can have as many light sources as you like (well sort of: each light source uses a GLSL ```var```, so the
maximum number of sources is really limited by how many ```vars``` are available, which should be at the bare minimum six).
<br><br>
Let's add another point light source:<br><br>
[![Point World space lights]({{ site.url }}/images/scenejs/pointLightWorld.png)](http://scenejs.org/examples.html?page=pointLightWorldMultiple)
[Run this demo](http://scenejs.org/examples.html?page=pointLightWorldMultiple)<br><br>

{% highlight javascript %}
var scene = SceneJS.createScene({
    nodes:[
        {
            type:"cameras/orbit",
            yaw:30, pitch:-30, zoom:10, zoomSensitivity:5,

            nodes:[

                // Override default lights with a single point
                // light source in World-space.

                // Since our light is in World-space, it will move
                // relative to our changing viewpoint, as if it
                // were a fixture in the scene.
                {
                    type:"lights",
                    lights:[
                        {
                            mode:"point",
                            color:{ r:1.0, g:0.8, b:0.8 },
                            diffuse:true,
                            specular:true,
                            pos:{ x:2.0, y:2.5, z:2.0 },
                            constantAttenuation:0.0, // [0..1]
                            linearAttenuation:0.0, // [0..1]
                            quadraticAttenuation:0.02, // [0..1]
                            space:"world"
                        },
                        {
                            mode:"point",
                            color:{ r:0.7, g:1.0, b:0.7},
                            diffuse:true,
                            specular:true,
                            pos:{ x:-1.5, y:2.5, z:2.0 },
                            constantAttenuation:0.0, // [0..1]
                            linearAttenuation:0.0, // [0..1]
                            quadraticAttenuation:0.02, // [0..1]
                            space:"world"
                        }
                    ],

                    nodes:[
                        {
                            type:"geometry/teapot"
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

Things to note:

* The ```lights``` node provides a single point light in World space
* World space means that the light is fixed in the 3D world, so we can move and turn towards and away from it
* The source provides reddish diffuse and specular illumination
* The intensity of the source will attenuate quadratically over distance

### A directional light

[![Point View space lights]({{ site.url }}/images/scenejs/pointLightView.jpg)](http://scenejs.org/examples.html?page=videoAlphaMap)
[Run this demo](http://scenejs.org/examples.html?page=videoAlphaMap)

{% highlight javascript %}
var scene = SceneJS.createScene({
    nodes:[
        {
            type:"cameras/orbit",
            yaw:30, pitch:-30, zoom:10, zoomSensitivity:5,

            nodes:[
                {
                    type:"lights",
                    lights:[
                        {
                            mode:"point",
                            color:{ r:0.8, g:1.0, b:0.8 },
                            diffuse:true,
                            specular:true,
                            pos:{ x:100.0, y:-50.0, z:100.0 },
                            constantAttenuation:0.2, // [0..1]
                            linearAttenuation:0, // [0..1]
                            quadraticAttenuation:0, // [0..1]
                            space:"view"
                        }
                    ],
                    nodes:[
                        {
                            type:"geometry/teapot"
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

### Directional lights

#### World-space
[![Directional World space lights]({{ site.url }}/images/scenejs/dirLightsWorld.png)](http://scenejs.org/examples.html?page=videoAlphaMap)
[Run this demo](http://scenejs.org/examples.html?page=videoAlphaMap)

{% highlight javascript %}
SceneJS.createScene({
    nodes:[
        {
            type:"cameras/orbit",
            yaw:30, pitch:-30, zoom:10, zoomSensitivity:5,
            nodes:[
                {
                    type:"lights",
                    lights:[
                        {
                            mode:"dir",
                            color:{ r:1.0, g:1.0, b:1.0 },
                            diffuse:true,
                            specular:true,
                            dir:{ x:1.0, y:-0.5, z:-1.0 },
                            space:"world"
                        }
                    ],
                    nodes:[
                        {
                            type:"geometry/teapot"
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

#### View-space
[![Directional View space lights]({{ site.url }}/images/scenejs/dirLightsView.jpg)](http://scenejs.org/examples.html?page=videoAlphaMap)
[Run this demo](http://scenejs.org/examples.html?page=videoAlphaMap)

{% highlight javascript %}
var scene = SceneJS.createScene({
    nodes:[
        {
            type:"cameras/orbit",
            yaw:30, pitch:-30, zoom:10, zoomSensitivity:5,
            nodes:[
                {
                    type:"lights",
                    lights:[
                        {
                            mode:"dir",
                            color:{ r:1.0, g:1.0, b:1.0 },
                            diffuse:true,
                            specular:true,
                            dir:{ x:1.0, y:-0.5, z:-1.0 },
                            space:"view"
                        }
                    ],
                    nodes:[
                        {
                            type:"geometry/teapot"
                        }
                    ]
                }
            ]
        }
    ]
});
{% endhighlight %}

### Ambient lights


### Material

### Flags
