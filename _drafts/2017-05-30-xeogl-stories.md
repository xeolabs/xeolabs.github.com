---
layout: post
title: A storytelling system for xeogl
description: "Building annotated stories with xeogl"
tagline: "Building annotated stories with xeogl"
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
An **AnnotationStory** is a collection of Annotations that's accompanied by a panel of markdown text. Words
in the text can be linked to Annotations, where clicking a link will focus the view on its Annotation.
<br><br>
The AnnotationStory renders the markdown as HTML in a panel to the left of the canvas. Clicking
an annotation link in the panel will basically fire a method on the AnnotationStory that will focus the camera on the
 Annotation and show its label.
<br><br>
[![]({{ site.url }}/images/xeogl/annotationsTank.png)](http://xeogl.org/examples/#presentation_annotations_tronTank)
<br><br>

## Creating a story

AnnotationStory has an "authoring" mode, in which you can interactively build Annotations and markdown by clicking on entities in the scene, making it a quick-and-easy utility for building interactive guided tours for any sort of xeogl content.
<br><br>
Let's load an example model, then create an empty AnnotationStory in authoring mode:

{% highlight javascript %}
var model = new xeogl.SceneJSModel({
        id: "tank",
        src: "models/scenejs/tronTank/tronTank.json"
    }, function() {
        new xeogl.AnnotationStory({
            authoring: true // <------- Authoring mode
        });
    });
{% endhighlight %}
[![]({{ site.url }}/images/xeogl/annotationsTank.png)](http://xeogl.org/examples/#presentation_annotations_tronTank)
<br><br>

If you then shift-click on an entity, the AnnotationStory will create an Annotation on the surface of that entity, with a dummy title and decription, as well as a paragraph of dummy markdown containing a link that focuses on the annotation when clicked.
<br><br>
[![]({{ site.url }}/images/xeogl/annotationsTank.png)](http://xeogl.org/examples/#presentation_annotations_tronTank)
<br><br>

Pressing ESC when authoring will clear the AnnotationStory.
<br><br>
Pressing ENTER when authoring serializes the AnnotationStory's current state to JSON and displays that in a new browser tab.

{% highlight javascript %}
{
    authoring: true,
    text: [
        "[annotation 1](focusAnnotation(0)) on this tank indicate that ....",
        "The [annotation 2](focusAnnotation(1)) is the tank's main armament, which  ...."
    ],
    annotations: [
        {
            primIndex: 204,
            bary: [0.05, 0.16, 0.79],
            occludable: true,
            glyph: "A",
            title: "Orange tracks",
            desc: "Indicates that the pilot is the rebel hacker, Clu",
            eye: [14.69, 17.89, -26.88],
            look: [5.35, 4.14, -15.44],
            up: [-0.09, 0.99, 0.11],
            pinShown: true,
            labelShown: true,
            entity: "tank.entity2"
        },
        {
            primIndex: 468,
            bary: [0.05, 0.16, 0.79],
            occludable: true,
            glyph: "B",
            title: "Cannon",
            desc: "Fires chevron-shaped bolts of de-rezzing energy",
            eye: [-0.66, 20.84, -21.59],
            look: [-0.39, 6.84, -9.18],
            up: [0.01, 0.97, 0.24],
            pinShown: true,
            labelShown: true,
            entity: "tank.entity9"
        }
    ]
}
{% endhighlight %}

Note how the markdown for the ````text```` parameter is organized as a list of paragraphs. Each link contains a call to a ````focusAnnotation()```` function, which is actually a private method of AnnotationStory. Internally, the AnnotationStory's markdown parser will massage that call to make sure it executes on this AnnotationStory instance (you can have multiple AnnotationStory instances, in case that's useful).

You would then edit that JSON to replace the markdown and annotation title and description with meaningful content, then paste the JSON into the AnnotationStory's constructor, and voila, you've interactively authored a presentation. If you leave the authoring parameter in place, you can repeat the authoring process described above to append more annotations. Remove the parameter when you've finished, otherwise your audience may unintentially create more annotations as they attempt to interact with your scene.

Shown below is our JSON pasted back into the constructor of the AnnotationStory, with tweaks made to the markdown and Annotations.

{% highlight javascript %}
var model = new xeogl.SceneJSModel({
        id: "tank",
        src: "models/scenejs/tronTank/tronTank.json"
    }, function() { // Model loaded
        new xeogl.AnnotationStory({
            // Disallow authoring, we're done
            // authoring: true,
            text: [
                "The [orange tracks](focusAnnotation(0)) on this tank indicate that ....",
                "The [cannon](focusAnnotation(1)) is the tank's main armament, which  ...."
            ],
            annotations: [
                {
                    primIndex: 204,
                    bary: [0.05, 0.16, 0.79],
                    occludable: true,
                    glyph: "A",
                    title: "Orange tracks",
                    desc: "Indicates that the pilot is the rebel hacker, Clu",
                    eye: [14.69, 17.89, -26.88],
                    look: [5.35, 4.14, -15.44],
                    up: [-0.09, 0.99, 0.11],
                    pinShown: true,
                    labelShown: true,
                    entity: "tank.entity2"
                },
                {
                    primIndex: 468,
                    bary: [0.05, 0.16, 0.79],
                    occludable: true,
                    glyph: "B",
                    title: "Cannon",
                    desc: "Fires chevron-shaped bolts of de-rezzing energy",
                    eye: [-0.66, 20.84, -21.59],
                    look: [-0.39, 6.84, -9.18],
                    up: [0.01, 0.97, 0.24],
                    pinShown: true,
                    labelShown: true,
                    entity: "tank.entity9"
                }
            ]
        });
    });
{% endhighlight %}



 

 



 
 
 
 
     
 





