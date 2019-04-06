---
layout: post
title: "Developing the BioDigital Human on SceneJS"
description: A case study of my work as platform lead/developer for the BioDigital Human, the leading Web-based 3D platform for the exploration of anatomy, medical conditions and treatments.            Working remotely from Berlin, Germany with the team in Manhattan, New York, I collaborate on the architecture and implementation of the Human's WebGL-based visualisation engine and its public developer API, as well as projects with various clients and partners.
thumbnail: human/human1.png
modified: 2015-06-31
category: articles
tags: [scenejs, biodigitalhuman]
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

<div style="width:100%; margin:0 auto; text-align: center;"><iframe width="640" height="360" src="//www.youtube.com/embed/2vJ13EXG6rc" frameborder="0"> </iframe></div>

<iframe
  id="embedded-human"
  frameBorder="0"
  width="600"
  height="600"
  allowFullScreen="true"
  src="https://human.biodigital.com/widget?be=1p0v&background=255,255,255,51,64,77&dk=1496c2fae9b943b838a40f86bfc8db5314c116e3">
</iframe>
<a href="https://www.biodigital.com"></a>

<a class="twitter-moment" href="https://twitter.com/i/moments/802173933466382336">xeogl - a data-driven 3D engine on WebGL </a> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

#### Smiletrain

[![Smiletrain]({{ site.url }}/images/human/smiletrain.jpg)](https://www.smiletrain.org/)

# JavaScript API

Though the Message API is perfect for two programs to talk to one another, it's somewhat clunky for scripting.
Therefore, we wrapped the message API in an intuitive JavaScript library that asynchronously proxies all of
Human's functionality behind a bunch of objects using pub/sub and promises. In true game scripting fashion, this
API provides everything you need to write specialized UIs and apps on the Human platform.

First, embed a Human in an IFRAME:

{% highlight html %}
<iframe id="my-iframe" src="..human URL.."></iframe>
{% endhighlight %}

Link to the API library:

{% highlight html %}
<script src="human-api.js"></script>
{% endhighlight %}

Instantiate an API client, connected to the IFRAME:

{% highlight javascript %}
var human = new HumanAPI.Human({
    iframe:{ id:"my_iframe" }
});
{% endhighlight %}

Then go ahead and start driving that Human. Let's show the Skeletal and Muscular
systems in X-ray mode, with everything transparent except for the left upper arm and bla. We'll also
 position the camera:

{% highlight javascript %}

// Show these objects:
human.scene.showObjects({
   "maleAdult-Skeletal_System": true,
   "maleAdult-Muscular_System": true
});

// Selected objects will be opaque in X-Ray mode
human.anatomy.selectObjects({
   "maleAdult-Frontal_Bone": true
});

// Activate X-Ray mode
human.view.xray.setEnabled(true);

// Fly the camera to the given position
human.view.camera.flyTo({
    eye: { x: -10, y: -5, z: -100 },
    look: { x: -10, y: -5, z: -100 }
});
{% endhighlight %}

All Human state may be subscribed to via the API, including repository content, camera state, modes, scene objects etc.
For example, to subscribe to the objects currently in the 3D scene:

{% highlight javascript %}
human.scene.on("objects",
    function(objects) { // Called whenever objects are loaded or unloaded
        console.log("Scene objects:");
        for (var objectId in objects) {
            var object = objects[objectId];
            console.log(object.objectId + ", " + object.displayName);
        }
    });
{% endhighlight %}

# Message API

Human has a JSON-RPC API, which lets us control it with commands via either the Web Socket or
[Web Message](http://en.wikipedia.org/wiki/Web_Messaging) protocols:

{% highlight javascript %}
{
    command: "XRay.SetEnabled",
    enabled: true
},
{
    command: "Camera.FlyTo",
    eye: { x: 0, y: 0, z: -100 },
    look: { x: -23, y: 10, z: 3 }
}
{% endhighlight %}

## Use case: Integration with ICE STORM
This was how we networked Human with Lockheed Martin's ICE STORM simulator at Health 2.0 last year:


![Lockheed ICE STORM ICU Simulator]({{ site.url }}/images/icestorm.jpg)


As the patient in the Lockheed's ER simulation on the left experienced changes in their condition, Human on
the right synchronised its animated view of the patient's respiratory and cardiovascular systems in real time to show
changes in respiration and heart rate, as well as conditions such as atrial and ventricular fibrillation, coronary
congestion and so on.