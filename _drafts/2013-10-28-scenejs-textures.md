---
layout: post
title: Defining Surface Appearance in SceneJS
description: "All about texturing in SceneJS"
modified: 2013-05-31
category: articles
comments: true
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

 In this tutorial I'll show you how define surface appearance for objects in SceneJS. Four types of node work to
 together to define appearance for the objects in their subgraphs:
   * ```lights``` nodes define light sources, which can be positional, directional and ambient, with specular and diffuse components
   * ```material``` nodes specify how object surfaces reflect light
   * ```texture``` nodes pattern the light or material color as it is applied to the surface, sourcing their patterns from images or video streams.
   * ```flags``` configure SceneJS rendering modes for light and texture for the objects.

In this tutorial we'll look at everything except for ```texture```, which is such a big topic that it's covered in several
other tutorials.

### Lights

### Material

### Flags

As will be shown below, texture nodes can have multiple layers, where each layer is a separate texture that may be independently transformed and applied to a different target.

A texture layer may be applied to a target [[material]] attribute to affect the way that is applied to geometry nodes. The [[material]] properties that may be texture targets:

* **baseColor** for colour mapping,
* **specular** for specular mapping,
* **alpha** for transparency mapping
* **emit** for glow mapping

A texture can also apply directly to [[geometry]] properties. The [[geometry]] properties that may be texture targets:

* **normals** for bump mapping

##Colour Mapping

To demonstrate how to do basic colour-mapping using a texture, we'll create the cube below. We'll apply one texture layer, an image of General Zod, to the diffuse ("baseColor") material property of a cube geometry:

![Colour mapping](http://scenejs.wikispaces.com/file/view/zod-texture.jpg)

To show the available attributes on a texture layer, we'll we'll specify all of the attributes, setting most of them to their default values. Only the layer's image URL is mandatory. We'll also rotate, translate and scale the layer a bit in the Animation section further down the page.

{% highlight javascript %}
{
    type: "material",
    baseColor: { r: 1.0, g: 1.0, b: 1.0 },

    nodes: [
        {
            type: "texture",
            layers: [
                {
                    // Only the image URI is mandatory:

                    uri:"http://scenejs.org/library/textures/misc/zod.jpg",

                    // Optional params:

                    applyTo: "baseColor",                  // Options so far are
                                                           //      “baseColor” (default),
                                                           //      “specular” for specular mapping
                                                           //      "alpha" for transparency mapping
                                                           //      "emit" for glow mapping
                                                           //      and "normals" for bump mapping

                    minFilter: "linear",                   // Options are ”nearest”,
                                                           //      “linear” (default),
                                                           //      “nearestMipMapNearest”,
                                                           //      ”nearestMipMapLinear” or
                                                           //      “linearMipMapLinear”

                    magFilter: "linear",                   // Options are “nearest” or
                                                           //      “linear” (default)

                    wrapS: "repeat",                       // Options are “clampToEdge” (default)
                                                           //      or “repeat”

                    wrapT: "repeat",                       // Options are "clampToEdge” (default)
                                                           //      or “repeat”

                    isDepth: false,                        // Options are false (default) or true
                    depthMode:"luminance"                  // (default)
                    depthCompareMode: "compareRToTexture", // (default)
                    depthCompareFunc: "lequal",            // (default)
                    flipY: false,                          // Options are true (default) or false
                    width: 1,
                    height: 1,
                    internalFormat:"lequal",               // (default)
                    sourceFormat:"alpha",                  // (default)
                    sourceType: "unsignedByte",            // (default)
                    blendMode: "multiply",                 // Options are "add" (default) or
                                                           //      "multiply"

                    // Optional transformations to apply to geometry UV coordinates
                    // before they are mapped to the texture. Note that these
                    // will have some performance overhead.

                    rotate: {            // Currently textures are 2-D, so only
                                         //      rotation about Z makes sense
                        z: 45.0
                    },

                    translate : {
                        x: 10,
                        y: 0,
                        z: 0
                    },

                    scale : {
                        x: 1,
                        y: 2,
                        z: 1
                    }
                }
            ],

            nodes: [
                {
                    type: "geometry/box"
                }
            ]
        }
    ]
}
{% endhighlight %}

##Animation

###UV Transformation

Once a texture is created, the transformation attributes of the layers may be updated to animate their textures. Other attributes are not updateable, since they would require re-creation of the textures.
* This only works on texture layers that were defined with scale, translate and/or rotate properties
* These transformations are not applied to the actual texture image. Instead, they are applied to the [[geometry]] UV coordinates as those are mapped to the texture, which can be a little misleading at first. Therefore, as we increase scale factors, for example, then the apparent size of the applied texture will decrease.

Lets adjust the scale and rotation of our texture layer:

{% highlight javascript %}
SceneJS.scene("theScene").findNode("theTexture").set("layers", {
    "0": {
        scale: {
            x: 1.2,
            y: 2.4
        },
        rotate: 55.0
    }
});
{% endhighlight %}

##Texture Layers

Now to show off multi-layer textures, we'll model the planet Earth as shown below, using colour, specular and emission maps. Notice how the shininess is only on the water, and the glowing points of the city lights on the dark side of the sphere.


![Texture layers](http://scenejs.wikispaces.com/file/view/earth-textured.jpg)


This Earth consists of a sphere [[geometry - Defining Shapes|geometry]], wrapped in a [[material - Defining How a Surface Reflects Light|material]] node to define diffuse and specular properties. The material is wrapped by a texture node that applies one layer to the material's diffuse color (baseColor) to create the land and oceans, a second layer applied to the material's specular factor to make the oceans shiny (a specular map), and a third layer applied to the material's emmisive factor to make shining city lights (an emission map).

{% highlight javascript %}
{

    nodes: [

        /*------------------------------------------------------------------
         * Texture with texture layers applied to base color and specularity
         *----------------------------------------------------------------*/
        {
            type: "texture",
            layers: [

                /*---------------------------------------------------------
                 * Underlying texture layer applied to the Earth material's
                 * baseColor to render the continents, oceans etc.
                 *--------------------------------------------------------*/
                {
                    uri: "images/earth.jpg",
                    applyTo:"baseColor"
                },

                /*---------------------------------------------------------
                 * Second texture layer applied to the Earth material's
                 * specular component to make the ocean shiney.
                 *--------------------------------------------------------*/
                {
                    uri: "images/earth-specular.gif",
                    applyTo:"specular"
                }
                ,

                /*---------------------------------------------------------
                 * Second texture layer applied to the Earth material's
                 * emission component to show lights on the dark side.
                 *--------------------------------------------------------*/
                {
                    uri: "images/earth-lights.gif",
                    applyTo:"emit"
                }
            ],

            /*---------------------------------------------------------
             * Sphere with some material
             *--------------------------------------------------------*/
            nodes: [

                {
                    type: "material",
                    specular: 5,
                    shine:100,
                    emit: 0.6,
                    baseColor: {r: 1, g: 1, b: 1},
                    nodes: [
                        {
                            type: "geometry/sphere"
                        }
                    ]
                }
            ]
        }
    ]
}
{% endhighlight %}

###Layer Blend Factor

We can specify an optional **blendFactor** on each texture layer to control the degree to which the layer contributes to the pixels already existing on the surface (ie. the material colour or effects of the previous texture layer).

This is useful for animating a fade from one texture to another.

The example below creates one layer that applies an image of Superman, followed by a second layer that applies General Zod. The second layer has a **blendFactor** of 0.4, causing it to blend 40% with the first layer:

![Texture Layer Blending](http://scenejs.wikispaces.com/file/view/textureBlend.jpg)

{% highlight javascript %}
 {
    type: "texture",

    id: "theTexture",

    layers: [
        {
            uri:"superman.jpg",
            //..
            applyTo:"baseColor",
            blendMode: "add",
            blendFactor: 1.0
        },
        {
            uri:"general-zod.jpg",
            //..
            applyTo:"baseColor",
            blendMode: "add",
            blendFactor: 0.4       // Fade in on top of superman layer
       }
    ]
}
{% endhighlight %}

Then we can animate the **blendFactor** of the second layer within [0.0..1.0] to animate the fade-in effect:

{% highlight javascript %}
var theTextNode = SceneJS.scene("theScene").findNode("theTexture");

var blendFactor = 0.7;  // Animate this per frame
theTextureNode.set("layers", {
    "1": {
        blendFactor: blendFactor
    }
});
{% endhighlight %}

##Bump Mapping

We can simulate a bumpy surface by applying a texture layer to the geometry's normal vectors. The R,G,B components of the texture image's pixels will store vectors by which the normals will be perturbed as the shader interpolates across the geometry's faces.

To show how it's done, we'll create the bumpy cube below. The checked pattern is done with one layer, which defines a colour map, while the bumps are done with a second layer that defines the bump map:

![Applied bump map](http://scenejs.wikispaces.com/file/view/bump-map.jpg)

![Bump map texture](http://scenejs.wikispaces.com/file/view/normal_map.jpg)

![Color map pattern](http://scenejs.wikispaces.com/file/view/pattern.jpg)

Here's the implementation - note that we only specify attributes for the layers where we need to override the default values:

{% highlight javascript %}
{
    type: "material",
    baseColor:      { r: 1.0, g: 1.0, b: 1.0 },
    specularColor:  { r: 1.0, g: 1.0, b: 1.0 },
    specular:       0.9,
    shine:          6.0,

    nodes: [
        {
            type: "texture",

            /* A texture can have multiple layers, each applying an
             * image to a different material or geometry component.
             */

            /* Our first layer texture is the bump map, and is applied within
             * the fragment shader to our geometry node's normal vectors.
             */
            layers: [
                {
                    uri:"normal_map.jpg",
                    applyTo:"normals"
                }
                ,

                /* Our second texture layer is applied within the fragment
                 * shader to the baseColor of our material.
                 */
                {
                    uri:"pattern.jpg",
                    applyTo:"baseColor"
                }
            ],

            nodes: [
                {
                    type: "geometry/box"
                }
            ]
        }
    ]
}
{% endhighlight %}

