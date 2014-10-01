---
layout: post
title: Reflective Surfaces in SceneJS
description: "Using cube maps to make objects reflective"
modified: 2014-07-24
category: articles
comments: true
tags: [scenejs, tutorial, reflection]
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

**Reflection mapping** or **environment mapping** is an efficient image-based lighting technique (introduced in [SceneJS v4.0](/articles/scenejs4-release)) for approximating the
appearance of a reflective surface by means of a precomputed texture image.
The texture image contains a view of the distant environment surrounding the rendered objects, and is projected onto the six
faces of a cube, which we can imagine as surrounding our objects. For each point on the objects, SceneJS will then
determine the colour by calculating the reflection vector at that point and mapping it to the texel in the environment map.
<br><br>
In this tutorial I'll show you how to

 * make objects reflective,
 * control surface reflectivity
 * vary reflectivity across a surface using a **specular control map**,
 * control reflection map intensity,
 * switch reflection on and off for objects, selectively, and
 * layer multiple reflections onto the same objects.

## Making objects reflective

As usual, we'll point SceneJS at where you keep your plugins:
{% highlight javascript %}
SceneJS.setConfigs({
     pluginPath:"./plugins"
});
{% endhighlight %}

Then we'll create a scene containing the reflective teapot shown below. The teapot is wrapped in a ```reflect``` node, which defines
our environment cube map.
<br><br>
[![Reflection Example 1]({{ site.url }}/images/scenejs/reflection.jpg)](http://scenejs.org/examples.html?page=cubeMap)
[Run this demo](http://scenejs.org/examples.html?page=cubeMap)

{% highlight javascript %}
SceneJS.createScene({
  nodes: [

    // The reflection cube map
    // Images taken from: http://hristo.oskov.com/projects/cs418/mp3/
    {
      type: "reflect",
      src: [
        "../../../textures/reflection/london/pos-x.png",    // +X axis
        "../../../textures/reflection/london/neg-x.png",    // -X axis
        "../../../textures/reflection/london/pos-y.png",    // +Y axis
        "../../../textures/reflection/london/neg-y.png",    // -Y axis
        "../../../textures/reflection/london/pos-z.png",    // +Z axis
        "../../../textures/reflection/london/neg-z.png"     // -Z axis
      ],

      // 100% texture intensity (default)
      intensity: 1.0,

      nodes: [

        // Specular material
        {
          type: "material",
          color: {
            r: 1,
            g: 0.9,
            b: 0
          },

          // Mirror-like reflection when specular == 1.0
          // No reflection at all when specular == 0.0
          specular: 0.8,

          nodes: [

            // Teapot primitive implemented by plugin at
            // http://scenejs.org/api/latest/plugins/node/geometry/teapot.js
            {
              type: "geometry/teapot"
            }
          ]
        }
      ]
    }
  ]
});
{% endhighlight %}
These are the images we're using for each face of our ```reflect``` node's cubemap:

----|----|----|----|----|----
![]({{ site.url }}/images/scenejs/reflection/london/pos-x.png) | ![]({{ site.url }}/images/scenejs/reflection/london/neg-x.png) | ![]({{ site.url }}/images/scenejs/reflection/london/pos-y.png)| ![]({{ site.url }}/images/scenejs/reflection/london/neg-y.png)| ![]({{ site.url }}/images/scenejs/reflection/london/pos-z.png)| ![]({{ site.url }}/images/scenejs/reflection/london/neg-z.png)
----|----|----|----|----|----
pos-x.png | neg-x.png | pos-y.png | neg-y.png | pos-z.png | neg-z.png

The amount of environment that's actually reflected by the teapot is the product of the ```intensity``` property on
the ```reflect``` node multiplied by the ```specular``` property on the ```material``` node. When that product is 0, then
the surface will reflect none of the environment, and when its 1.0, the surface will have mirror-like reflectivity. One
thing to note in the example above is that the product is ```0.8```, which allows some of the yellow material ```color``` to
show through, creating a gold-like surface.
<br><br>
The ```specular``` property on the ```material``` node also controls the amount of specular light reflected by the
surface from ```light``` light nodes. This allows you to have a shiny object with a dull reflection, by specifying a
low value for ```intensity``` and a high value for ```specular```.
<br><br>
The geometry must have normal vectors, but does not require UV coordinates. The ```reflect``` node is designed so that you
can just slap it around some portion your scene graph and all the geometries within that will magically get reflective
surfaces, just as long as those geometries have normal vectors and ```material``` nodes with non-zero ```specular``` factors.
<br><br>


## Reflection map intensity

Let's look at the effect of varying ```reflect``` node ```intensity``` while keeping ```material``` node ```specular``` fixed.
The example below contain four objects, each wrapped with a ```reflect``` node with a different ```intensity``` value.

<br><br>
[![Reflection Example 1]({{ site.url }}/images/scenejs/reflectionIntensity.jpg)](http://scenejs.org/examples.html?page=reflectionIntensity)
[Run this demo](http://scenejs.org/examples.html?page=reflectionIntensity)

* From left to right, the ```intensity``` factors are 1.0, 0.4, 0.2 and 0.05, respectively.
* Note how all the tanks have different amounts of relection, but the same amount of specularity.
* Click *Source code* on the example for source code.

## Material specularity

Now let's look at the effect of varying ```material``` node ```specular``` while keeping ```reflect``` node ```intensity``` fixed.
The example below shows three boxes that share the same ```reflect```, but each have their own ```material``` node with a unique ```specular``` factor.

<br><br>
[![Reflection Example 1]({{ site.url }}/images/scenejs/surfaceReflectivity.jpg)](http://scenejs.org/examples.html?page=reflectionWithVaryingSpecularity)
[Run this demo](http://scenejs.org/examples.html?page=reflectionWithVaryingSpecularity)

* From left to right, the ```specular``` factors are 1.0, 0.5 and 0.0, respectively.
* See how the specularity varies, along with the amount of reflection.
* Click *Source code* on the example for source code.

## Varying reflectivity across a surface

Often we want to control reflectivity across a surface using a control texture. In SceneJS, a **specular map** is a texture
that is applied to the ```specular``` property of a geometry's ```material```, to vary the
amount of specular reflection across the geometry surface. The amount of specular reflection at each fragment on the
  surface is multiplied by the value of the *red* channel of the corresponding pixel in the texture image. We can also use
  a greyscale image for these textures, where only the red channel will be actually be used.
  <br><br>
  As mentioned earlier, when ```reflect``` nodes are involved, the ```material``` node's ```specular``` factor contributes to the degree of surface
  reflectivity, so texturing that factor with a map will vary the reflectivity across the surface.
  <br><br>
  The demo below shows three boxes with specular map textures, where each texture image has an increasing darkness of the
  diamond shapes it contains.
  <br><br>
[![Reflection Example 2]({{ site.url }}/images/scenejs/reflectionControlMap.jpg)](http://scenejs.org/examples.html?page=reflectionWithControlMap)
[Run this demo](http://scenejs.org/examples.html?page=reflectionWithControlMap)

* Note how as the diamonds get darker, the value of the **red** channel in the pixels will be lower. Therefore, the specularity
at the corresponding surface fragment will be lower after multiplying by it, so the reflectivity at that point will be lower also.
* Since the ```material``` node's ```color``` property is light blue, as the reflectivity decreases, the blue color begins to contribute
more to the surface color, while the ```reflect``` node's texture contributes less.
* See [this example](http://scenejs.org/examples.html?page=specularMap) for a basic example of how specular maps are normally done in SceneJS.

Shown below are the images we're using for the specular control maps for each box:

![]({{ site.url }}/images/scenejs/reflection/reflectionSpecularMap1.jpg) | ![]({{ site.url }}/images/scenejs/reflection/reflectionSpecularMap2.jpg) | ![]({{ site.url }}/images/scenejs/reflection/reflectionSpecularMap3.jpg)


## Switching reflection on and off

Usually you'll want to apply reflection only to certain objects in your scene. Do that with a ```reflection``` flag
on a ```flags``` node, as shown in the example below:
<br><br>
[![Reflection Example 3]({{ site.url }}/images/scenejs/reflection.jpg)](http://scenejs.org/examples.html?page=reflectionFlag)
[Run this demo](http://scenejs.org/examples.html?page=reflectionFlag)

{% highlight javascript %}
 var scene = SceneJS.createScene({
   nodes: [

     // Mouse-orbited camera, implemented by plugin at
     // http://scenejs.org/api/latest/plugins/node/cameras/orbit.js
     {
       type: "cameras/orbit",
       yaw: 30,
       pitch: -30,
       zoom: 10,
       zoomSensitivity: 5,
       nodes: [

         // Flags node which enables or disables reflection on our teapot
         {
           type: "flags",
           id: "myFlags",
           flags: {
             reflection: true // (default is true)
           },

           nodes: [

             // The reflection cube map
             // Images taken from: http://hristo.oskov.com/projects/cs418/mp3/
             {
               type: "reflect",
               src: [
                 "../../textures/reflection/london/pos-x.png",
                 "../../textures/reflection/london/neg-x.png",
                 "../../textures/reflection/london/pos-y.png",
                 "../../textures/reflection/london/neg-y.png",
                 "../../textures/reflection/london/pos-z.png",
                 "../../textures/reflection/london/neg-z.png"
               ],

               // 100% texture intensity
               intensity: 1.0,

               nodes: [

                 // Specular material
                 {
                   type: "material",
                   color: {
                     r: 1,
                     g: 0.9,
                     b: 0
                   },

                   // Mirror-like reflection when specular == 1.0
                   // No reflection at all when specular == 0.0
                   specular: 0.8,

                   nodes: [

                     // Teapot primitive implemented by plugin at
                     // http://scenejs.org/api/latest/plugins/node/geometry/teapot.js
                     {
                       type: "geometry/teapot"
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

Then we can toggle reflectivity on and off for the teapot:

{% highlight javascript %}
myScene.getNode("myFlags",
    function(flags) {
        flags.setTransparent(false);
    });
{% endhighlight %}

* The ```geometry/teapot``` in our scene "inherits" the ```flags``` node, where the ```flags``` can be anywhere on the path from
 the ```geometry/teapot``` up to the scene root. Only the first ```flags``` on the path up to the root will be applied, overriding
  any other higher ```flags``` for the ```geometry/teapot```.
* By default, the ```flags``` node ```reflection``` property is ```true```.

## Multiple reflections

Like ```texture``` nodes, multiple reflections can be layered onto the same object, as shown in the example below:
<br><br>
[![Reflection Example 4]({{ site.url }}/images/scenejs/reflectionLayered.jpg)](http://scenejs.org/examples/index.html#reflections_multipleCombined)
[Run this demo](http://scenejs.org/examples/index.html#reflections_multipleCombined)

{% highlight javascript %}
SceneJS.createScene({
    nodes: [

        // Orbiting camera node, implemented by plugin at
        // http://scenejs.org/api/latest/plugins/node/cameras/orbit.js
        {
            type: "cameras/orbit",
            yaw: 30,
            pitch: -30,
            zoom: 10,
            zoomSensitivity: 1.0,

            nodes: [

                // London reflection
                // Images taken from: http://hristo.oskov.com/projects/cs418/mp3/
                {
                    type: "reflect",
                    src: [
                        "textures/reflection/london/pos-x.png",
                        "textures/reflection/london/neg-x.png",
                        "textures/reflection/london/pos-y.png",
                        "textures/reflection/london/neg-y.png",
                        "textures/reflection/london/pos-z.png",
                        "textures/reflection/london/neg-z.png"
                    ],
                    intensity: 1.0,

                    nodes: [

                        // Cloudy sky reflection
                        {
                            type: "reflect",
                            src: [
                                "textures/reflection/clouds/a.png",
                                "textures/reflection/clouds/b.png",
                                "textures/reflection/clouds/top.png",
                                "textures/reflection/clouds/bottom.png",
                                "textures/reflection/clouds/c.png",
                                "textures/reflection/clouds/d.png"
                            ],
                            intensity: 1.0,

                            nodes: [

                                // Specular material
                                {
                                    type: "material",
                                    color: { r: 1, g: 1.0, b: 1.0 },

                                    // Mirror-like reflection when specular == 1.0
                                    // No reflection at all when specular == 0.0
                                    specular: 0.5,

                                    nodes: [

                                        // Teapot primitive implemented by plugin at
                                        // http://scenejs.org/api/latest/plugins/node/geometry/teapot.js
                                        {
                                            type: "geometry/teapot"
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

* Each of the nested ```reflect``` nodes in our scene has the same value for ```intensity```, so they both contribute the same amount to the ```teapot```.

## Reflections on textured objects:

Finally, just to show how the ```reflect``` node is designed so that you can slap around almost anything you already
have in your scene, let's finish off with an example in which we've taken a raptor imported with an ```import/obj``` node
and wrapped it with a ```reflect``` to make it shiny:
<br><br>
[![Reflection Example 3]({{ site.url }}/images/scenejs/reflectionToOBJ.jpg)](http://scenejs.org/examples.html?page=reflectionWithOBJImport)
[Run this demo](http://scenejs.org/examples.html?page=reflectionWithOBJImport)

# Conclusion

In this tutorial I've shown you how to use the ```reflect``` node to make geometry
   appear to reflect its environment. Reflections are designed so that you can wrap them around any subgraph to make objects
    with specular materials reflective, without modifying the contents of the subgraph. The amount of reflectivity is governed by the
    combination of the ```reflect``` node's ```intensity``` property with the ```specular``` property on the ```material``` nodes around the
     geometries in the subgraph. You can vary the amount of reflection using the ```reflect``` node's ```intensity``` property,
     which may avoid needing to adjust the ```specular``` properties on those ```material``` nodes.
<br><br>
