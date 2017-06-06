---
layout: post
title: ifcgl - A WebGL-based glTF+IFC model viewer
description: "Loading IFC models from glTF"
tagline: "Loading IFC models from glTF"
modified: 2017-17-04
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

The Industry Foundation Classes (IFC) data model is used to describe building and construction industry data. glTF is a
file format for 3D graphics content. [xeogl](http://xeogl.org) is a WebGL-based 3D engine I've been working on.
<br><br>
Last month I made [ifcgl](http://xeolabs.com/ifcgl), an IFC model viewer built on [xeogl](http://xeogl.org) that loads glTF. The
idea with this viewer is that you tag your glTF with metadata to indicate IFC types for the scene nodes, then once you've
loaded it, you're able to do do things on sets of objects matching IFC types, such as "make everything transparent except for the walls and ramps":

{% highlight javascript %}
 var viewer = new ifcgl.Viewer({ camvasId: "theCanvas" });

 viewer.loadModel("beachHouse", "models/beachHouse.gltf", function () {
     viewer.setOpacity("beachHouse", 0.3); // Everything in house transparent
     viewer.setOpacity(["IfcCurtainWall", "ifcRamp"], 1.0); // Walls and ramps opaque
     viewer.viewFit(["IfcCurtainWall", "ifcRamp"]); // Fit 3D view to walls and ramps
 });
{% endhighlight %}

When you've programmed the view you want, you can then save the viewer state to a JSON bookmark, which you can
load at any time to restore the view. You can also share a bookmark between two viewers, if they are both able to load
the models referenced in the bookmark:

 {% highlight javascript %}
 var beachHouseWallsAndRamps = viewer.getBookmark();
 var viewer2 = new ifcgl.Viewer({ canvasId: "anotherCanvas" });
 viewer2.setBookmark(beachHouseWallsAndRamps);
 {% endhighlight %}

ifcgl doesn't actually need the IFC types in order to function as a regular glTF viewer. You can also just operate on
objects individually using their IDs:

 {% highlight javascript %}
 viewer.loadModel("toolbox", "models/toolbox.gltf", function () {
    var objects = viewer.getObjects("toolbox");
    viewer.hide(objects);
    viewer.show(["toolbox.pipeWrench", "toolbox.monkeyWrench", "toolbox.socketWrench"]);
    viewer.translate("toolbox.monkeyWrench", [0, 20, 0]);
    viewer.rotate("toolbox.monkeyWrench", [85, 0, 0]);
 });
 {% endhighlight %}

Element types are about as far as the IFC support goes for this viewer. I'm thinking that the application layer would manage other
aspects of the IFC schema, such as layers, relationships etc, while the viewer can just use IDs and types to reference
the objects in the 3D view.

## Features

* load IFC models from glTF,
* transform, show and hide objects,
* control object opacities,,
* query object boundaries,
* raycast to find objects,
* navigate camera to objects, and
* save and load JSON bookmarks.

Being able to programmatically control objects lets us code animated presentations, like exploding parts assemblies, guided tours and so on.



## Architecture

The API consists of a single `````Viewer````` class, which is a facade wrapping a xeogl ````Scene````.

The viewer API represents everything in the Scene
with IDs, managing the scene objects, camera etc. internally.

For scalability, I'd normally break it up
into various classes representing scene objects, camera  etc, but in this case I went for a single facade in order to have
 a simple data-driven boundary between the xeogl graphics engine and the application layer.
 <br><br>
The API represents everything in the viewer with IDs, managing entities internally. This results in a set of self-contained
and data-driven methods that are easy to play around with in the browser's JavaScript console.
 <br><br>
 With xeogl doing all the heavy lifting, and with much of the required functionality built into xeogl by
 design, the facade turned out to be a very thin wrapper of around a thousand lines of code or so.

## Loading IFC types from glTF

When loading glTF, ````ifcgl.Viewer```` creates an object for each leaf ````node```` it parses within the default glTF ````scene````. If we tag those leaf ````nodes```` with IFC type codes, then ````ifcgl.Viewer```` will register the objects against those codes. That enables us do various operations on groups of objects selected by IFC type, such as show, hide, query boundary, and so on.

That't pretty much the extent of ifcgl`s support for the IFC at this point. I'm thinking that the application layer would manage other aspects of the IFC schema, such as layers, relationships etc., while the viewer can just use types and IDs to reference the objects. I don't think the viewer layer needs to know much more than the objects' IFC types.

## Bookmarking

Being data-driven (with a little help from xeogl's fully data-driven design), the state of an ifcgl viewer can be saved and loaded as a JSON bookmark object. You can instantiate an empty ifcgl viewer, call its methods from the JavaScript console to load models, set camera, show/hide/move objects etc, then dump it all out as a serialized JSON bookmark in the console. Copy and paste that JSON into your viewer's constructor call, and voila, you have a custom glTF model view, built up from live coding in the console.

## Customizing

Hopefully ifcg fits your project's needs and you can just use it as-is. In reality however, engines like this are not one-size-fits-all and usually need to be shaped to the specific use cases of your app (generic engines are actually considered an architectural anti-pattern in the games industry). If so, then I hope ifcgl is so simple that it can be hacked and customized to fit. And if you can think of any directions I should take the ifcgl API, then please do let me know via the issue tracker!




 

 



 
 
 
 
     
 





