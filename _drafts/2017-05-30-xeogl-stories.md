---
layout: post
title: A storytelling system for xeogl
description: "A storytelling presentation system for xeogl"
tagline: "A storytelling presentation system for xeogl"
modified: 2017-17-08
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

After making xeogl's [annotation system]({{site.url}}/articles/xeogl-annotations) I set to work making a
sort of storytelling component that would render a panel of text with links that would fly the camera to annotations.
<br><br>
The component is implemented by [xeogl.AnnotationStory](). You create the AnnotationStory with a list of
[xeogl.Annotations]() and a string of markdown that contains links that call methods on the AnnotationStory API.
<br><br>
First we'll import our Tron Tank from SceneJS format:
{% highlight javascript %}
var model = new xeogl.SceneJSModel({
    id: "tank",
    src: "models/scenejs/tronTank/tronTank.json"
});
{% endhighlight %}

Once the model has finished loading, we'll create our AnnotationStory.

{% highlight javascript %}
model.on("loaded", function () {

   new xeogl.AnnotationStory({
       speaking: false, // Set true to have a voice announce each annotation
       authoring: true, // Set true to author the annotations
       text: [
           "# [Stories](../docs/classes/AnnotationStory.html) : Tron Tank Program",
           "This is a Light Tank from the 1982 Disney movie *Tron*.",
           "The [orange tracks](focusAnnotation(0)) on this tank indicate that ....",
           "![](./images/Clu_Program.png)",
           "The [cannon](focusAnnotation(1)) is the tank's main armament, which  ....",
           "The [pilot hatch](focusAnnotation(2)) is where Clu enters and exits the tank.",
           "At the back of the tank is the [antenna](focusAnnotation(3)) through ....",
           "*\"I fight for the users!\" -- Clu*"
       ],
       annotations: [
           {
               entity: "tank.entity2"
               primIndex: 204,
               bary: [0.05, 0.16, 0.79],
               glyph: "A",
               title: "Orange tracks",
               desc: "Indicates that the pilot is the rebel hacker, Clu",
               eye: [14.69, 17.89, -26.88],
               look: [5.35, 4.14, -15.44],
               up: [-0.09, 0.99, 0.11]
           },
           {
               entity: "tank.entity9",
               primIndex: 468,
               bary: [0.05, 0.16, 0.79],
               glyph: "B",
               title: "Cannon",
               desc: "Fires chevron-shaped bolts of de-rezzing energy",
               eye: [-0.66, 20.84, -21.59],
               look: [-0.39, 6.84, -9.18],
               up: [0.01, 0.97, 0.24]
           },
           {
               entity: "tank.entity6",
               primIndex: 216,
               bary: [0.05, 0.16, 0.79],
               glyph: "C",
               title: "Pilot hatch",
               desc: "Clu hops in and out of the tank program here",
               eye: [1.48, 11.79, -15.13],
               look: [1.62, 5.04, -9.14],
               up: [0.01, 0.97, 0.24]
           },
           {
               entity: "tank.entity9",
               primIndex: 4464,
               bary: [0.05, 0.16, 0.79],
               glyph: "D",
               title: "Antenna",
               desc: "Links the tank program to the Master Control Program",
               eye: [13.63, 16.79, 13.87],
               look: [1.08, 7.72, 3.07],
               up: [0.08, 0.99, 0.07]
           }
       ]
   });
});
{% endhighlight %}

When this AnnotationStory is instantiated, it creates four annotations on our model and renders a panel of HTML 
from the markdown on the left side of the page.
<br><br>
Note how some of the markdown links contain function calls. These resolve to a set of private methods that the AnnotationStory
 provides as an API for your markdown to call. So far, AnnotationStory provides just one method,
 ````focusAnnotation(index)````, which focuses the view on the Annotation at the given index in the list.

## Building a story




 

 



 
 
 
 
     
 





