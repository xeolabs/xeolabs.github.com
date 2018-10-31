---
layout: biodigital
title: The BioDigital Human™
client: BioDigital, NYC, USA
description: Built the core WebGL rendering technology for the <b>BioDigital Human</b> anatomy visualization platform, which now serves over 3 million users. Lead 3D software development from 2009-2015.
thumbnail: human/biodigital-human-platform-thumb.png
modified: 2018-23-03
category: portfolio
comments: false
tags: [scenejs, human, biodigital, webgl]
tech: "WebGL, SceneJS"
article: true
location: "Manhattan, New York"
website: http://scenejs.org
websiteReadible: http://scenejs.org
---

From 2009 to 2018 I worked remotely from Berlin, Germany with the team at [BioDigital Systems](http://biodigital.com) in Manhattan, New York, to help develop 
and maintain the [BioDigital Human](http://biodigitalhuman.com) anatomy visualization platform and its public developer API.
<br><br>
We developed the Human on **[SceneJS](http://scenejs.org)**, an open source WebGL library I created for developing 3D graphics applications 
in Web browsers without using plugins. One of the first WebGL engines, SceneJS evolved alongside the WebGL specification, before 
we eventually made a private version which we adapted specifically for the Human. 
<br><br>
In this article, I'm going to describe SceneJS' journey from a weekend side  project to how we applied it within BioDigital's 
Web-based anatomy visualization platform. Ten years down the track, the platform now has over three million 
registered users and continues to develop, with a growing library of models of anatomy and physical conditions.
<br><br>
<figure>
	<a href="http://biodigitalhuman.com"><img src="{{ site.url }}/images/human/biodigital-human-platform.png"></a>
</figure>

# Contents

- [2005: SceneJS Origins](#2005--scenejs-origins)
- [2006: Experiments with Canvas3D](#2006--experiments-with-canvas3d)
- [2008: SceneJS Open Sourced](#2008--scenejs-open-sourced)
- [2009: SceneJS Powering the BioDigital Human](#2009--scenejs-powering-the-biodigital-human)
  * [Human Content Pipeline Origins](#human-content-pipeline-origins)
  * [Human Developer API Origins](#human-developer-api-origins)
  * [Using the Developer API](#using-the-developer-api)
  * [API Demo: Lockheed-Martin ICE STORM Integration](#api-demo--lockheed-martin-ice-storm-integration)
  * [Smiletrain Virtual Surgery Simulator](#smiletrain-virtual-surgery-simulator)
- [2015: SceneJS Private Fork](#2015--scenejs-private-fork)
- [SceneJS Presentations](#scenejs-presentations)
- [Next Steps: xeogl](#next-steps--xeogl)
- [Acknowledgements](#acknowledgements)


<!-- > * [scenejs.org](http://scenejs.org) -->
<!-- > * [biodigitalhuman.com](http://biodigitalhuman.com) -->
<!-- > * [Slides from my talk at the Berlin WebGL Meetup](http://slides.com/xeolabs/deck) -->
<!-- > * [SceneJS in OpenGL Insights 2012](http://127.0.0.1:4000/pdfs/OpenGLInsights.pdf) -->

<br>

# 2005: SceneJS Origins

I started SceneJS as a weekend experiment, somewhere around late 2005. Back then, JavaScript wasn't so fast and friends 
like [@ohunt](https://twitter.com/ohunt?lang=en) were busy writing raytracers on JavaScript that took forever, 
as a kind of twisted browser-cooking exercise.<br><br>The first version of SceneJS even rendered wireframe as DIV elements, arranged 
using [Bresenham's line algorithm](https://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm), so I wasn't expecting 
that to be particularly interactive.
<br><br>
That early version was even written in completely functional-style JavaScript, and did a ton of garbage collection and scope traversal. I was 
inspired at the time by LISP and CLOJURE and so perhaps took my fascination with terse scene definitions a little too far!     
  
# 2006: Experiments with Canvas3D
  
Web-based 3D without plugins actually started to look viable in 2006, however, with the Canvas 3D experiments started by [Vladimir Vukićević](https://en.wikipedia.org/wiki/Vladimir_Vuki%C4%87evi%C4%87) at Mozilla, and by the end of 
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

I loved the idea of a 3D world defined declaratively, as pure data. At this point, I was inspired by the likes 
of [VRML](https://en.wikipedia.org/wiki/VRML), which I'd used as a student to visualize data, and by the terse, declarative 
syntax of [JavaFX](https://en.wikipedia.org/wiki/JavaFX).

# 2008: SceneJS Open Sourced

My day job back in 2008 (in a cubicle, maintaining a Java-based spam-scrubbing platform) just wasn't firing my creative circuits. I needed to get back in
 touch with the creative culture that drew me into programming in the first place: 3D graphics, SIGGRAPH journals, cyberpunk 
 science fiction - all that good stuff.
<br><br>
So I quit my job, [put SceneJS on GitHub](https://github.com/xeolabs/scenejs), and devoted my time to getting 
 back into 3D programming, using WebGL.   

# 2009: SceneJS Powering the BioDigital Human
 
A little while later, I signed up with BioDigital Systems and we began developing the [BioDigital Human](http://biodigitalhuman.com) on SceneJS. 

### Human Content Pipeline Origins

[Brandon Smith](https://twitter.com/phaeta) (AKA "The Wizard") began by exporting one of 
BioDigital's models of the human skeletal system to COLLADA, which I then imported into SceneJS using an experimental open 
source [SceneJS asset server](https://github.com/xeolabs/scenejs-asset-server) I'd been working on. 
<br><br>
The 206 bones within that model rendered at a promising rate of around ~20FPS, so we took a gamble on WebGL and so the BioDigital flagship app was born.
<br><br>
Our biggest challenge was getting the platform to work reliably across the various operating systems, browsers and GPUs, 
and so the next few years involved navigating patchy GPU support and a lot of "Aw Snap". We owe a lot to the work of 
the [ANGLE](https://en.wikipedia.org/wiki/ANGLE_(software)) developers, whose work allows full hardware acceleration on 
Windows without relying on OpenGL graphics drivers.
<br><br>
Over the next couple of years I rewrote SceneJS twice, and we managed to double that performance. [Olli Etuaho](https://twitter.com/oletus?lang=en) from NVIDIA even helped 
out and made optimizations for mobile GPUs, and later, after we'd made a private fork (described below), we got it rendering 
at ~60FPS for most of the full anatomy model.

### Human Developer API Origins

Before Human, I'd also done some open source experiments with controlling SceneJS via a JSON-RPC message protocol, and we used those 
to get started with our [developer API](https://developer.biodigital.com/).
<br><br>
Some of those experiments were:
 
* **[xeoEngine-experiment](https://github.com/xeolabs/xeoEngine-experiment)** which allows you to control SceneJS via JSON-RPC, 
* **[actorjs](https://github.com/xeolabs/actorjs)** which allows you to define and update *actor* components via JSON-RPC, and 
* **[scenejs-grid](https://github.com/xeolabs/scenejs-grid)** which applies the actor and JSON-RPC concepts on SceneJS.
   
Those are now archived projects, but were useful for determining the best way to control a Human within an 
IFRAME embedded in a 3rd-party container page.
<br><br>
One of my inspirations for JSON-RPC was the messaging system that [Paul Brunt](https://twitter.com/super_eggbert) had built into his WebGL-based [GLGE](http://www.glge.org) engine.
 
### Using the Developer API

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
 
Next, we'll create an instance of the Human API:
{% highlight javascript %}
var human = new HumanAPI("myWidget"); 
{% endhighlight %}

Through the API, we can now make the widget do things like set the position of the camera etc:
{% highlight javascript %}
human.send("camera.orbit", { yaw: 90 });
{% endhighlight %}

For more info on what's possible with the API, sign up with Human and check out the tutorials at [developer.biodigital.com](https://developer.biodigital.com/).

### API Demo: Lockheed-Martin ICE STORM Integration
We used the API for various presentations. For one presentation, we used it to interface the Human with the Lockheed Martin 
ICE STORM ICU Simulator, so that changes to the patient's heartbeat and respiration within the simulator were rendered as 
morph animations within the Human:

<figure>
	<img src="{{ site.url }}/images/human/icestorm.jpg">
	<figcaption>Human interfaced with the Lockheed Martin ICE STORM ICU Simulator. Human (on the right) is animating the 
	heart and lungs using morph targets, synchronized via JSON-RPC with changes to the patient's condition in the ICE STORM ICU simulator (on the left).</figcaption>
</figure> 

### Smiletrain Virtual Surgery Simulator

The Human is a platform on which we can build applications. One of the most rewarding of those was the WebGL-based SmileTrain 
Surgical Cleft Repair Simulator.
<br><br>
We based the Smiletrain Simulator on [Aaron Oliker's](https://www.linkedin.com/in/aaron-oliker-544a6a2/) 
earlier C++ version, which he implemented on OpenSceneGraph.
  
<figure>
	<iframe width="560" height="315" src="https://www.youtube.com/embed/fPNOXnzaiJ0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
	<figcaption><a href="https://www.smiletrain.org/" title="SceneJS powering the BioDigital Human">The Smiletrain Surgical Simulator is built on the BioDigital Human and assists healthcare professionals in developing countries with learning cleft repair procedures.</a></figcaption>
</figure>

<br><br>
That slick Darth Vader Approved UI you're seeing in Human and Smiletrain is the work of BioDigital front-end engineers Kathia Yau and Avinash Chan. 

### 2015: SceneJS Private Fork 

In 2015, with the company expanding, [Tarek Sherif](https://twitter.com/thsherif) took over my role as 3D programming lead, since it made 
 sense for that job to be a performed on-site by a non-virtual person who could chase people around the office, instead of 
 typing emails all night in the wrong time zone.
<br><br> 
Throwing the git pull requests back and forth, we then added many more features to Human and SceneJS, including a streaming asset 
server, physically-based rendering (PBR), geometry and texture compression, particle systems and an improved post-effects pipeline.
<br><br>
For the post-effects support, Tarek built an extensible plugin-based architecture based off his own open source WebGL engine, [PicoGL](https://tsherif.github.io/picogl.js/).  

# SceneJS Presentations

Along the way, I got to write about SceneJS and present it to fellow graphics nerds:

* Wrote a chapter about SceneJS in *OpenGL Insights* 2012, which you can now [download for free](http://127.0.0.1:4000/pdfs/OpenGLInsights.pdf).
* Talked about SceneJS at the 2015 Berlin WebGL Meetup - [here are the slides](http://slides.com/xeolabs/deck) from that talk, with a few embedded live demos.

# Next Steps: xeogl

I'm going to keep making more of these WebGL engines, because there's never a one-size-fit-all solution (and well, it is a 
bit of a creative compulsion).
<br><br>
The public fork of SceneJS is now archived and no longer under development. However, if you're looking for a production-proven 
WebGL-based 3D engine which is currently used in several commercial IFC and CAD viewers, you might find my latest engine useful: [http://xeogl.org](http://xeogl.org).

# Acknowledgements

 * [Vladimir Vukevic](https://en.wikipedia.org/wiki/Vladimir_Vuki%C4%87evi%C4%87) for kicking WebGL off with his Canvas3D experiments,
 * the [Khronos WebGL Working Group](https://www.khronos.org/) for overseeing the development of the WebGL specification,
 * the [ANGLE developers](https://chromium.googlesource.com/angle/angle/+/master/README.md) for making WebGL work on DirectX,
 * [COLLADA™](https://www.khronos.org/collada/) for the file format that got us started with the Human (and taught me a lot about what goes into a 3D engine), 
 * [Paul Brunt](http://www.paulbrunt.co.uk/#/) for his pioneering open source WebGL-based [GLGE](http://www.glge.org/) engine, which was like a living textbook on graphics algorithms on JavaScript,
 * the [Zygote Body](https://www.zygotebody.com/), which was originally created as the Google Body by [Won Chun](https://twitter.com/won3d?lang=en), as his 20% experiment at Google,
 * the SceneJS community for a crash course on what a 3D engine is, and
 * the model creation team at BioDigital for creating all the cool content that makes the platform shine.

I won't try to mention the whole BioDigital team here because I might miss someone, but I'll give shouts out to our interns, such 
as [Jacqueline Chu](https://www.linkedin.com/in/jacqueline-chu-7a532558) and [Shuai Shao](https://twitter.com/shrekshao) (AKA ShrekShao), 
who came in fresh from academia and added many valuable rendering features. 
<br><br>






