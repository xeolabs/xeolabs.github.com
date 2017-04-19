---
layout: post
title: Annotating 3D Scenes in xeogl
description: "Sticking 3D labels on entities in xeogl"
tagline: "Sticking 3D labels on entities in xeogl"
modified: 2017-19-02
category: articles
comments: true
tags: [xeogl, webgl]
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

This month I added the ability to attach annotations to entities in xeogl. A xeogl annotation is a labeled pin that
attaches to a specified position on the surface of an Entity's Geometry. These are roughly similar to annotations in
SketchFab and are provided as a non-core Annotation component.

#### Dynamic positioning

An Annotation's pin position is given as the index of a triangle, along with its barycentric coordinates within that
triangle. Whenever the Entity is transformed to a new location, or the Geometry is modified, the Annotation will
automatically recalculate its position from the triangle index and the barycentric coordinates.

#### Occlusion culling

Annotations can be configured to hide themselves whenever their pin positions are occluded by other objects in the 3D
view. To determine if each Annotation is occluded, xeogl renders the scene to a hidden framebuffer, along with a colored
point for each Annotation's pin. Then xeogl determines if each Annotation is occluded if the pixel at the location of
its pin does not match the special pin color. The pin color is configured as some unusual color that is not used
elsewhere in the scene.

#### Editable

Everything in xeogl is runtime editable, including Annotations. You can reassign an Annotation to a different Entity, pin
it to a different triangle and barycentric coordinate, edit its title and description etc.

#### Camera positions

Like SketchFab, each Annotation may be configured with a viewpoint, given as ````eye````, ````look```` and ````up````,
which you can set the Camera to whenever we focusing the user's attention on that Annotation. As shown in the example,
you can use a CameraFlightAnimation to fly the Camera to each Annotation, for things like guided tours of Annotations.

## Usage

{% highlight javascript %}
 var model = new xeogl.GLTFModel({
    src: "models/gltf/2.0/Reciprocating_Saw/PBR-SpecGloss/Reciprocating_Saw.gltf",
    transform: new xeogl.Rotate({
        xyz: [1, 0, 0],
        angle: 90
    })
 });

 model.on("loaded", function() {

    var view = model.scene.camera.view;
    view.eye = [-110.89, -44.85, 276.65];
    view.look = [-110.89, -44.85, -0.46];
    view.up = [0, 1, 0];
    view.zoom(20);

    // Create three annotations

    var a1 = new xeogl.Annotation({
        entity: model.entities[156], // Red handle
        primIndex: 125,
        bary: [0.3, 0.3, 0.3],
        occludable: true,
        symbol: "1",
        title: "Handle",
        desc: "This is the handle. It allows us to grab onto the saw so we can hold it and make things with it.",
        eye: [-355.481, -0.871, 116.711],
        look: [-227.456, -57.628, 5.428],
        up: [0.239, 0.948, -0.208],
        pinShown: true,
        labelShown: false
    });

    var a2 = new xeogl.Annotation({
        entity: model.entities[156], // Red handle and cover
        primIndex: 10260,
        bary: [0.333, 0.333, 0.333],
        occludable: true,
        symbol: "2",
        title: "Handle and cover",
        desc: "This is the handle and cover. It provides something grab the saw with, and covers the things inside.",
        eye: [-123.206, -4.094, 169.849],
        look: [-161.838, -37.875, 37.313],
        up: [-0.066, 0.971, -0.228],
        pinShown: true,
        labelShown: false
    });

    var a3 = new xeogl.Annotation({
        entity: model.entities[796], // Barrel
        primIndex: 3783,
        bary: [0.3, 0.3, 0.3],
        occludable: true,
        symbol: "3",
        title: "Barrel",
        desc: "This is the barrel",
        eye: [80.0345, 38.255, 60.457],
        look: [35.023, -0.166, 8.679],
        up: [-0.320, 0.872, -0.368],
        pinShown: true,
        labelShown: false
    });

     //

    var cameraFlight = new xeogl.CameraFlightAnimation();
    var lastAnnotation;

    function gotoAnnotation(annotation) {         
        if (lastAnnotation) {
            annotation.labelShown = false;
        }         
        annotation.labelShown = true;        
        cameraFlight.flyTo(annotation);        
        lastAnnotation = annotation; 
    }

    a1.on("pinClicked", gotoAnnotation);
    a2.on("pinClicked", gotoAnnotation);
    a3.on("pinClicked", gotoAnnotation);
 });
{% endhighlight %}



 

 



 
 
 
 
     
 





