---
layout: biodigital
title: Developing the BioDigital Human™ on SceneJS
description: <b>SceneJS</b> is an open source JavaScript library I created for WeGL-based 3D graphics.<br><br>In this article, I describe SceneJS' journey from a weekend experiment to how we ended up creating the world's leading Web-based anatomy visualization platform around it.
thumbnail: human/biodigital-human-platform.png
modified: 2018-23-03
category: portfolio
comments: false
tags: [scenejs, human, biodigital, webgl]
---

Since 2009, I've worked remotely from Berlin, Germany with [BioDigital Systems](http://biodigital.com) in NYC, to lead 3D development 
on the [BioDigital Human](http://biodigitalhuman.com) and its developer API on [SceneJS](http://scenejs.org).
<br><br>
In 2015, with the company rapidly expanding, [Tarek Sherif](https://twitter.com/thsherif) took over as 3D programming lead, since it made 
sense for that role to be a performed on-site by a non-virtual person who could chase people around the office, instead of 
typing emails all night in the wrong time zone. 
<br><br>
SceneJS is an open source (MIT) JavaScript library I created for developing 3D graphics applications on WebGL. One of 
the first WebGL engines, SceneJS evolved alongside the WebGL specification to include advanced features such as posteffects, 
physics, LoD, culling and many GL state optimizations. 
<br><br>
In this article, I'm going to describe SceneJS' journey from a weekend experiment to how we applied it within the world's leading 
Web-based anatomy visualization platform, which now has over three million registered users, plus a bunch a bunch of awards, 
including the 2015 Webby Award for Best Health Website.
<br><br>
<figure>
	<a href="http://biodigitalhuman.com"><img src="{{ site.url }}/images/human/biodigital-human-platform.png"></a>
	<figcaption><a href="http://biodigitalhuman.com" title="SceneJS powering the BioDigital Human">SceneJS powering the BioDigital Human</a></figcaption>
</figure>

## 2005: SceneJS Origins

I started SceneJS as a weekend experiment, somewhere around late 2005. Back then, JavaScript wasn't as fast as it is now and friends 
at university like [@ohunt](https://twitter.com/ohunt?lang=en) were busy writing raytracers on JavaScript that took forever, 
as a kind of novelty.<br><br>The first version of SceneJS even rendered wireframe as DIV elements, arranged 
using [Bresenham's line algorithm](https://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm), so I wasn't expecting 
that to be particularly interactive.
<br><br>
That version was even written in completely functional-style JavaScript, and did a ton of garbage collection and scope traversal. I was 
inspired by LISP and CLOJURE and maybe took my fascination with terse scene definitions a little too far with that!     
  
## 2006: Experiments with Canvas3D
  
Web-based 3D without plugins started to look viable in 2006, however, with the Canvas 3D experiments started by [Vladimir Vukićević](https://en.wikipedia.org/wiki/Vladimir_Vuki%C4%87evi%C4%87) at Mozilla, and by the end of 
2007, both Mozilla and Opera had made their own separate implementations. Suddenly, interactive 3D in the browser didn't 
seem so crazy, so I switched SceneJS over to using Canvas3D. 
<br><br>
Following tradition, my first SceneJS demo 
was, of course, a [Gouraud-shaded](https://en.wikipedia.org/wiki/Gouraud_shading) [Utah Teapot](https://en.wikipedia.org/wiki/Utah_teapot), a bit like this one:
<br><br>
[![SceneJS First Example]({{ site.url }}/images/scenejs/firstExample.png)](http://scenejs.org/examples.html?page=firstExample)
  
{% highlight javascript %}
var scene = SceneJS.createScene({
    nodes:[{
        type:"material",
        color: { r: 0.3, g: 0.3, b: 1.0 },
        nodes:[{
            type: "rotate",
            id: "myRotate",
            y: 1.0, angle: 0,
            nodes: [{
                type:"geometry/teapot",
                id: "myTeapot"
            }]
        }]
    }]
});

setInterval(function() {)
    scene.getNode("myRotate").incRotateX(0.5);
});
{% endhighlight %}

I loved the idea of a 3D world defined declaratively, as pure data. At this point, I was inspired by the likes of [VRML](https://en.wikipedia.org/wiki/VRML), which I'd used as a student to 
visualize software architectures and OOP metrics, and by the terse, declarative syntax of [JavaFX](https://en.wikipedia.org/wiki/JavaFX).

## 2008: SceneJS Open Sourced

My day job back then (in a cubicle, maintaining a Java-based spam-scrubbing platform) just wasn't firing my creative circuits. I needed to get back in
 touch with the creative culture that drew me into programming in the first place: 3D graphics, SIGGRAPH journals, cyberpunk 
 science fiction, Tron - all that good stuff.
<br><br>
So I quit my day job, [put SceneJS on GitHub](https://github.com/xeolabs/scenejs), and devoted my time to getting 
 back into 3D programming, using WebGL.   

## 2009: SceneJS Powering the BioDigital Human
 
A little while later, I began working for BioDigital Systems and we ended up making the [BioDigital Human](http://biodigitalhuman.com) on SceneJS. 

### Human Content Pipeline Origins

Using my earlier open source [SceneJS asset server experiments](https://github.com/xeolabs/scenejs-asset-server), we began by parsing a COLLADA 
model of the human skeletal system into a JSON-based SceneJS scene definition. The scene rendered at a promising rate 
of around ~20FPS, so we took a gamble on WebGL and thus the BioDigital flagship app was born.
<br><br>
Our biggest challenge was getting the platform to work reliably across the various operating systems, browsers and GPUs, 
and so the next few years involved navigating patchy GPU support and a lot of "Aw Snap". We owe a lot to the work of 
the [ANGLE](https://en.wikipedia.org/wiki/ANGLE_(software)) developers, whose work allows full hardware acceleration on 
Windows without relying on OpenGL graphics drivers.
<br><br>
Over the next couple of years I rewrote SceneJS twice, and we managed to double that frame rate. NVIDIA even helped 
out and made optimizations for mobile GPUs, and later, after we made a private fork (described below), we got it rendering 
at ~60FPS for most of the basic anatomy model.

### Human Developer API Origins

Before Human, I'd also done some open source experiments with controlling SceneJS via a JSON-RPC message protocol, and we used those 
to get started with our [developer API](https://developer.biodigital.com/).
<br><br>
Some of those experiments were:
 
* **[xeoEngine-experiment](https://github.com/xeolabs/xeoEngine-experiment)** which allows you to control SceneJS via JSON-RPC, 
* **[actorjs](https://github.com/xeolabs/actorjs)** which allows you to define and update *actor* components via JSON-RPC, and 
* **[scenejs-grid](https://github.com/xeolabs/scenejs-grid)** which combines the actor and JSON-RPC concepts to SceneJS.
   
Those are now archived projects, but were useful for determining the best way to control a Human within an 
IFRAME from a 3rd-party container page.
<br><br>
One of my inspirations for JSON-RPC was the messaging system that Paul Brunt had built into his WebGL-based [GLGE](http://www.glge.org) engine.
 
#### Using the Developer API

To use the API, start by embeding the Human Widget in your page. In the example below, we'll use the cochlear implant model:
{% highlight html %}
<!DOCTYPE html>
<html style="height: 100%; overflow: hidden;">
  <body style="height: 100%; margin: 0">
    <iframe id="myWidget"
       src="https://human.biodigital.com/widget/?m=cochlear_implant&dk=<YOUR-DEVELOPER-KEY>"
       width="100%"
       height="100%">
    </iframe>
  <script src="https://developer.biodigital.com/builds/api/2/human-api.min.js"></script>
  </body>
</html>
{% endhighlight %}
 
Next, create an instance of the Human API:
{% highlight javascript %}
var human = new HumanAPI("myWidget"); 
{% endhighlight %}

Through the API, we can now make the widget do things like set the position of the camera etc:
{% highlight javascript %}
human.send("camera.orbit", { yaw: 90 });
{% endhighlight %}

For more info on what's possible, sign up with Human and check out the API tutorials at [developer.biodigital.com](https://developer.biodigital.com/).

#### API Demo: Lockheed-Martin ICE STORM Integration
We used the API for various presentations. For one presentation, we used it to interface the Human with the Lockheed Martin 
ICE STORM ICU Simulator, so that changes to the patient's heartbeat and respiration within the simulator were rendered as 
morph animations within the Human:

<figure>
	<img src="{{ site.url }}/images/human/icestorm.jpg">
	<figcaption>Human interfaced with the Lockheed Martin ICE STORM ICU Simulator. Human (on the right) is animating the 
	heart and lungs using morph targets, synchronized via JSON-RPC with changes to the patient's condition in the ICE STORM ICU simulator (on the left).</figcaption>
</figure> 

#### Smiletrain Virtual Surgery Simulator

The Human is a platform on which we can build applications. One of the most rewarding of those was the WebGL-based SmileTrain 
Surgical Cleft Repair Simulator, on which I was also lead 3D programmer.
<br><br>
We based this off [Aaron Oliker's](https://www.linkedin.com/in/aaron-oliker-544a6a2/) 
earlier C++ version, which he implemented on OpenSceneGraph.
  
<figure>
	<iframe width="560" height="315" src="https://www.youtube.com/embed/fPNOXnzaiJ0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
	<figcaption><a href="https://www.smiletrain.org/" title="SceneJS powering the BioDigital Human">The Smiletrain Surgical Simulator is built on the BioDigital Human and assists healthcare professionals in developing countries with learning cleft repair procedures.</a></figcaption>
</figure>

## 2015: SceneJS Private Fork 

With [Tarek Sherif](https://twitter.com/thsherif) taking over as on-site lead graphics programmer 2015, we added many more 
features to Human and SceneJS, including a streaming asset server, physically-based rendering (PBR), geometry and texture compression, particle 
systems and an improved post-effects pipeline.
<br><br>
For post-effects support in SceneJS, Tarek built an extensible plugin-based architecture based off his own open source WebGL engine, [PicoGL](https://tsherif.github.io/picogl.js/).  

## SceneJS Presentations

Along the way, I got to write about SceneJS and present it to fellow graphics nerds:

* In 2012, I contributed a chapter on the pre-fork version of SceneJS to *OpenGL Insights*, which you can now [download for free](http://127.0.0.1:4000/pdfs/OpenGLInsights.pdf).
* In 2015, I gave a talk on SceneJS at the Berlin WebGL Meetup - [here are the slides](http://slides.com/xeolabs/deck) from that talk, with a few embedded demos.

## Acknowledgements

 * [Vladimir Vukevic](https://en.wikipedia.org/wiki/Vladimir_Vuki%C4%87evi%C4%87) for kicking WebGL off with his Canvas3D experiments,
 * the [Khronos WebGL Working Group](https://www.khronos.org/) for overseeing the development of the WebGL specification,
 * the [ANGLE developers](https://chromium.googlesource.com/angle/angle/+/master/README.md) for making WebGL work on DirectX,
 * [COLLADA™](https://www.khronos.org/collada/) for the file format that got us started with the Human (and taught me a lot about what goes into a 3D engine), 
 * [Paul Brunt](http://www.paulbrunt.co.uk/#/) for his pioneering open source WebGL-based [GLGE](http://www.glge.org/) engine, which was like a living textbook on graphics algorithms on JavaScript,
 * the [Zygote Body](https://www.zygotebody.com/), which was originally created as the Google Body by [Won Chun](https://twitter.com/won3d?lang=en), as his 20% experiment at Google,
 * the SceneJS community for a crash course on what a 3D engine is, and
 * the model creation team at BioDigital for creating all the cool content that makes the platform shine.

I won't list the whole BioDigital team here because I might miss someone, but I'll give props to interns, such 
as [Jacqueline Chu](https://www.linkedin.com/in/jacqueline-chu-7a532558) and [Shuai Shao](https://twitter.com/shrekshao) (AKA ShrekShao), 
who came in fresh from academia and added many valuable rendering features. 
<br><br>

<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->




