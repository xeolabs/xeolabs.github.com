---
layout: post
title: Alpha Mapping in SceneJS
description: "Using textures to make transparent areas on the surfaces of objects"
modified: 2013-05-31
category: articles
comments: true
tags: [scenejs, tutorial, transparency, texturing]
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

Alpha mapping is a technique where an image is mapped to a 3D object to designate certain areas of the object to be transparent or translucent.
<br><br>
In this tutorial, I'll build on techniques from the [Transparency](/articles/scenejs-transparency) tutorial to show you how to

* apply a texture to an object make holes in it
* apply a video texture to make animated areas of transparency
<br><br>

#Basic alpha map

As usual, we'll point SceneJS at where you keep your plugins:
{% highlight javascript %}
SceneJS.setConfigs({
     pluginPath:"./plugins"
});
{% endhighlight %}

Then we'll create the scene below, in which the outer green box has an alpha texture which creates transparent areas through
which you can see the inner blue box.
<br><br>
[![Alpha map]({{ site.url }}/images/scenejs/basicAlphaMap.png)](http://scenejs.org/examples.html?page=alphaMap)
[Run this demo](http://scenejs.org/examples.html?page=alphaMap)
<br><br>Shown below is the texture we'll use for the alpha map. The white areas will be where the outer box will be opaque,
while the black areas are where it will be transparent.

<br><br>

[![Alpha map image]({{ site.url }}/images/scenejs/leavesAlphaMapSmall.jpg)]({{ site.url }}/images/scenejs/leavesAlphaMap.jpg)

<br><br>Listed below is the code that defines our scene. It's using ```flags``` and ```material``` nodes to set up
transparency (which you learned about in the [Transparency](/articles/scenejs-transparency) tutorial), plus a ```texture``` node, which
  varies the transparency across the surface of the outer green box.

{% highlight javascript %}
var scene = SceneJS.createScene({
    nodes:[

        // Orbiting camera node, implemented by plugin at
        // http://scenejs.org/api/latest/plugins/node/cameras/orbit.js
        {
            type:"cameras/orbit",
            yaw:30,
            pitch:-30,
            zoom:5,
            zoomSensitivity:1.0,

            nodes:[

                // Outer green box with alpha map

                // Flags node enables transparency for the box.
                // This enables the material `alpha` to specify
                // the degree of opacity.
                {
                    type:"flags",
                    flags:{
                        transparent:true
                    },
                    nodes:[

                        // Material node's 'alpha' property makes
                        // the pixels of the box opaque by default.

                        {
                            type:"material",
                            color:{
                                r:0.5, g:1, b:0.5
                            },
                            alpha:1.0,
                            nodes:[

                                // The alpha map is a texture that's
                                // applied to the alpha channel of the
                                // material, to vary it according to
                                // the intensity of each texture pixel.
                                {
                                    type:"texture",
                                    id:"theTexture",
                                    layers:[
                                        {
                                            src:"../../../textures/leavesAlphaMap.jpg",
                                            applyTo:"alpha",
                                            blendMode: "mul"  // Default
                                        }
                                    ],
                                    nodes:[
                                        {
                                            type:"prims/box"
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                },

                // Inner blue box

                {
                    type:"material",
                    color:{
                        r:0.3, g:0.3, b:1
                    },
                    nodes:[
                        {
                            type:"scale", x:0.7, y:0.7, z:0.7,
                            nodes:[
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
});
{% endhighlight %}

Things to note:

* The ```transparent``` flag on that ```flags``` node enables transparent rendering of the outer green box.
* The ```material``` node around the outer green box has an ```alpha``` property, which is set to ```1.0```.
* An ```alpha``` value of 0.0 is completely transparent, while 1.0 is completely opaque.
* The ```texture``` node applies a texture to the outer box.
* The ```texture``` node's ```applyTo``` property causes the image to be applied to the ```material``` node's ```alpha``` property,
effectively modulating the alpha across the box's surface.

### Texture blend mode

The ```blendMode``` on our texture has the value ```multiply```, which causes it to multiply the colour of each image pixel
with the material alpha value, causing the material alpha to be 0.0 where the pixels are black, and 1.0 where the pixels are white.
<br><br>
The other supported value for that property is ```add```, which adds the value of each image pixel to the alpha. To
achieve the same effect as shown in this example, you would then need an ```alpha``` of 0.0 on the ```material```.

#Video alpha map

Now I'll show you how to apply a video as an alpha map. This time, we'll dynamically sample our texture from [an .OGV file]({{ site.url }}/images/scenejs/testVideo.ogv), to
create a cool flowing gas pattern across the surface of the outer box.


<br><br>
[![Alpha video map]({{ site.url }}/images/scenejs/videoAlphaMap.jpg)](http://scenejs.org/examples.html?page=videoAlphaMap)
[Run this demo](http://scenejs.org/examples.html?page=videoAlphaMap)

<br><br>The code is identical to the previous example, except for the ```texture``` node, which looks like this:

{% highlight javascript %}
{
    type:"texture",
    layers:[
        {
            source:{
                type:"video",
                src:"../../../movies/testVideo.ogv"
            },
            applyTo:"alpha"
        }
    ],
    //...
}
{% endhighlight %}

Things to note:

* The ```source``` on this ```texture``` causes it to employ the core SceneJS "video" texture plugin to stream the video file into the texture
* Each of the video's frames is a grayscale image, applied using the same principles as used for regular image textures
<br>
<br>
# Conclusion

In this tutorial I showed you how to use the ```texture``` node, in conjunction with transparency techniques I described
in the [Transparency](/articles/scenejs-transparency) tutorial, to make holes in things. I also showed you how it's easy
 to use a video, instead of the usual image, in order to make animated regions of transparancy.