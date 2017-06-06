---
layout: post
title: "Annotations in xeogl"
description: "A Sketchfab-style annotation system for xeogl"
tagline: "Made a Sketchfab-style annotation system for xeogl"
modified: 2017-17-05
category: articles
comments: true
tags: [xeogl, webgl]
---

<section id="table-of-contents" class="toc">`****`
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

I recently made a Sketchfab-style annotation system for [xeogl](http://xeogl.org), a WebGL-based 3D engine
I've been working on.
<br><br>
An annotation is a label that you pin to the surface an entity. Click this screenshot for a [demo](http://xeogl.org/examples/#presentation_annotations_tronTank):
<br><br>
[![]({{ site.url }}/images/xeogl/annotationsTank.png)](http://xeogl.org/examples/#presentation_annotations_tronTank)
<br><br>
The code below shows how to create an annotation. First, we'll load our model:
{% highlight javascript %}
var model = new xeogl.SceneJSModel({ // Importing a SceneJS model for fun
    id: "tank",
    src: "models/scenejs/tronTank/tronTank.json"
});
{% endhighlight %}

Once the model has loaded, we'll create an [Annotation](http://xeogl.org/docs/classes/Annotation.html) on one of its entities:

{% highlight javascript %}
model.on("loaded", function () {

        var annotation = new xeogl.Annotation({
            entity: "tank.entity9",         // Entity instance or ID
            primIndex: 468,                 // First triangle vertex in geometry indices
            bary: [0.05, 0.16, 0.79],       // Barycentric coordinates in triangle
            glyph: "A",                     // Symbol(s) for pin
            title: "Cannon",
            desc: "Fires chevron-shaped bolts of de-rezzing energy",
            eye: [-0.66, 20.84, -21.59],    // Camera eye position (optional)
            look: [-0.39, 6.84, -9.18],     // Camera point-of-interest (optional)
            up: [0.01, 0.97, 0.24]          // Camera "up" vector (optional)
        });
    });
{% endhighlight %}

The annotation is pinned at the given barycentric coordinates within a triangle of its entity's mesh (the Tank's cannon). Whenever the entity
is transformed or deformed, the annotation will always stick to that position on the surface of the entity. In the demo,
this is demonstrated by the (A) annotation on our Tron Tank's cannon, which will move wherever the cannon goes.

## Camera vantage points

As we saw in the previous example, you can optionally specify a camera position from which to view the annotation, given as
````eye````, ````look```` and ````up```` vectors. You can then fly the camera to look at the annotation by
passing the annotation to a [CameraFlightAnimation](http://xeogl.org/docs/classes/CameraFlightAnimation.html):

{% highlight javascript %}
var cameraFlight = new xeogl.CameraFlightAnimation();

cameraFlight.flyTo(annotation);
{% endhighlight %}

## Creating annotations by picking

I made annotations easy to create from pick results:

{% highlight javascript %}
xeogl.input.on("mouseclicked", function(canvasPos) {

    var hit = myScene.pick({ // Ray-pick entity surface
        canvasPos: canvasPos,
        pickSurface: true
    });

    if (hit) {
        var lookat = myScene.camera.view;

        new xeogl.Annotation({
            entity:     hit.entity,
            primIndex:  hit.primIndex,
            bary:       hit.bary,
            title:      "A new world awaits you",
            desc:       "in the off-world colonies.",
            eye:        lookat.eye,
            look:       lookat.look,
            up:         lookat.up
        });
    }
});
{% endhighlight %}

With this snippet, whenever we click on the surface of an entity, we'll create an annotation at that
point. We'll also save the current camera position with the annotation, so that we can restore the camera to the vantage
point that we created it from (see previous example).

## Occlusion culling

The annotation system automatically hides annotations when they are hidden behind other objects in the 3D view. You can see this in
the demo example, where the (C) annotation disappears while the cannon barrel moves over it.
<br><br>
The system uses a fast GPU-assisted occlusion technique which renders a small point at each annotation's position, then
determines the annotation to be occluded when the color of the point's location on the canvas does not match the point. Since
reading WebGL's drawing buffer is expensive, the system performs this occlusion test for all annotations in a batch on
every 10th frame or so. You'll note a slight lag between when the annotation is occluded/revealed and hidden/shown.
<br><br>
For large meshes, this technique is much more efficient for large meshes and numbers of objects than
a ray-casting approach.
<br><br>
Culling is enabled for an annotation by default, but you can disable it:
{% highlight javascript %}
annotation.culling = false;
{% endhighlight %}

## Events

Annotations fire some events that you can subscribe to. Let's make it so that when you click an
annotation's pin the camera will focus on the annotation and the label will open:

{% highlight javascript %}
var cameraFlight = new xeogl.CameraFlightAnimation();
var lastAnnotation;

annotation.on("pinClicked", function (annotation) {
    if (lastAnnotation) {
        annotation.labelShown = false;
    }
    annotation.labelShown = true;
    cameraFlight.flyTo(annotation);
    lastAnnotation = annotation;
});
{% endhighlight %}

You can also track the annotation's position in Local-space (coordinate space of entity's geometry
before modeling transformation) and World-space (after the modeling transform):

{% highlight javascript %}
annotation.on("localPos", function(localPos) {
    console.log("Local pos changed: " + JSON.stringify(localPos, null, "\t"));
});

annotation.on("worldPos", function(worldPos) {
    console.log("World pos changed: " + JSON.stringify(worldPos, null, "\t"));
});
{% endhighlight %}

The Local position will change when the geometry is edited, while the World position will change when the
 geometry is edited and/or the modeling transform is updated.

## Styling

Annotations manage their HTML elements internally, giving them hardcoded CSS class names. The rules for those classes are defined
in [annotation-style.css](https://github.com/xeolabs/xeogl/blob/master/examples/js/annotations/annotation-style.css), so
you can tweak that file if you want to customize the appearance of your annotations.

## Conclusion

So there you have it - an annotation system for [xeogl](http://xeogl.org) that's inspired by the one
on [Sketchfab](https://help.sketchfab.com/hc/en-us/articles/202512456-Annotations).
<br><br>
The main points in this introduction were:

* The [Annotation](http://xeogl.org/docs/classes/Annotation.html) component class provides the means to pin labels on [Entities](http://xeogl.org/docs/classes/Entity.html) that have triangle mesh
[Geometries](http://xeogl.org/docs/classes/Geometry.html).
* Since an annotation is pinned to a barycentric location with a its triangle, it can  dynamically
recalculate its Cartesian coordinates from the triangle vertices whenever the Entity is transformed or the Geometry is
edited.
* An annotation can be configured with a camera vantage point from which to view it. This is useful for focusing the camera
on the annotation, as part of a presentation.
* The annotation API makes it easy to create annotations from the results of picking operations.
* An Annotation will automatically hide itself when occluded by another object in the 3D view.
* You can customize the appearance of annotations by editing their [CSS stylesheet](https://github.com/xeolabs/xeogl/blob/master/examples/js/annotations/annotation-style.css).
* See the [demo example](http://xeogl.org/examples/#presentation_annotations_tronTank) to see annotations in action, click VIEW SOURCE so see the source code.

Thanks for reading!

 
 
 
 
     
 





