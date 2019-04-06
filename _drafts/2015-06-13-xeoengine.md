---
layout: post
title: "Introducing xeogl - a component-based WebGL engine"
description: "A component-based WebGL engine from xeoLabs"
modified: 2015-06-31
category: articles
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

[![Transparency sorting]({{ site.url }}/images/xeogl/transparency_sorting.png)](http://xeogl.org/examples/#materials_techniques_transparencySort)
xeogl transparency sorting demo - [run this](http://xeogl.org/examples/#materials_techniques_transparencySort)

# Introduction

**[xeogl](http://xeogl.org)** is an open-source WebGL-based engine for visualizing complex and dynamic 3D scenes. In addition to a free set of steak knives, you get:

* **Scene graph API** comprised of data-driven components, with lots of love given to the [documentation](http://xeogl.org/docs/classes/Scene.html) and [examples](http://xeogl.org/examples/#geometry_TorusGeometry).
* **Dynamically editable scenes** - everything in a scene can be updated at runtime, right down to 
the [minFilter](http://xeogl.org/docs/classes/Texture.html#property_minFilter) on
a [texture](http://xeogl.org/docs/classes/Texture.html) (which makes it pretty sweet for live coding).
* **Fully-observable scene state**, where *everything* fires change events. In combination with editable everything, this 
makes it possible to wire component properties together as outputs and inputs using change handlers (think dataflowy-node-based 
scene builder UIs!).
* **Sensible defaults** for virtually everything - create a running scene with one line of code, then build it up from there.
* **Fast rendering** using all the tricks, such as optimized draw list compilation, state tracking, sorting, batching and caching, plus lots of other classic game engine patterns. 
* **Performance monitoring** - get running [statistics](http://xeogl.org/examples/#profiling_statistics) on memory usage and WebGL performance.
* **Serializable scene state** - save and load scenes as JSON, easily clone scenes and components.

# Background

# Usage

First, link xeogl in your page:

{% highlight html %}
<iframe id="my-iframe" src="http://xeogl.org/build/xeogl.min.js"></iframe>
{% endhighlight %}

## Defining a 3D scene

Next, we'll create a [Scene](http://xeogl.org/docs/classes/Scene.html), which is a component-object graph, like the example
below. Note that a Scene is essentially a container for soup of [components](http://xeogl.org/docs/classes/Component.html) that are tied together into
entities by [GameObjects](http://xeogl.org/docs/classes/GameObject.html).

![First scene]({{ site.url }}/images/xeogl/firstScene.png)

Since xeogl provides defaults for pretty much everything, we only need to specify what we need to override, in this case the
[Material](http://xeogl.org/docs/classes/Material.html) and [Geometry](http://xeogl.org/docs/classes/Geometry.html) for our
[GameObject](http://xeogl.org/docs/classes/GameObject.html). The GameObject will fall back on the scene's default flyweight
components (eg. [Camera](http://xeogl.org/docs/classes/Camera.html), [Lights](http://xeogl.org/docs/classes/Lights.html) etc) for
all the other components we didn't explicitly attach to it. As you can see, this keeps the code compact, while flattening the learning curve and making harder to
accidentally make scenes that don't render anything.

<iframe height='469' scrolling='no' src='//codepen.io/xeolabs/embed/xGNjPM/?height=469&theme-id=11815&default-tab=js' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/xeolabs/pen/xGNjPM/'>xGNjPM</a> by xeolabs (<a href='http://codepen.io/xeolabs'>@xeolabs</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>
<br>
In fact, we could even just let xeogl provide a default Scene for us. Here's this example again, using xeogl's default Scene:

 <iframe height='492' scrolling='no' src='//codepen.io/xeolabs/embed/WvBJMO/?height=492&theme-id=11815&default-tab=js' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/xeolabs/pen/WvBJMO/'>xeogl: Using the default Scene</a> by xeolabs (<a href='http://codepen.io/xeolabs'>@xeolabs</a>) on <a href='http://codepen.io'>CodePen</a>.
 </iframe>


> I avoided naming the GameObject class "Entity" because that would imply an [Entity component system](https://en.wikipedia.org/wiki/Entity_component_system),
  in which scenes are built using fairly strict [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance). In xeogl, we can
  subclass components to create more specialized components, and may even choose to apply inheritance-based patterns
  like [subclass sandbox](http://gameprogrammingpatterns.com/subclass-sandbox.html), in which we subclass GameObject to make
  template base classes.

## Animation

Animate scene state by setting the values of component properties. Almost everything in xeogl fires change events
which you can subscribe to. This is cool for scripting the engine, while also making it easier to write a UI layer for a
graphical scene editor, which is one of the future goals of xeogl.

<iframe height='482' scrolling='no' src='//codepen.io/xeolabs/embed/MwdGGG/?height=482&theme-id=11815&default-tab=js' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/xeolabs/pen/MwdGGG/'>xeogl: Animation</a> by xeolabs (<a href='http://codepen.io/xeolabs'>@xeolabs</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>


Likewise, you can update properties on any of the scene's default flyweight components, such as the default camera:

{% highlight javascript %}
scene.camera.view.eye = [3.0, 2.0, -10.0];
{% endhighlight %}

> By making every bit of scene state fire a change event when updated, I'm applying
the [open/closed principle](https://en.wikipedia.org/wiki/Open/closed_principle), which lets us make all kinds
  of other objects to control and synchronize the state, without needing to extend xeogl in any way.

## Editing

You can edit everything in your scene dynamically, at runtime. Create and destroy components, link or unlink them to
each other, update their properties, and so on. Let's add a diffuse [Texture](http://xeogl.org/docs/classes/Texture.html)
map to our [Material](http://xeogl.org/docs/classes/Material.html), which will immediately appear on our box:

{% highlight javascript %}
var texture = new xeogl.Texture(scene, {
    src: "myTexture.jpg"
});

material.diffuseMap = texture;
{% endhighlight %}

And of course, since edits are state changes, we can subscribe to them in the usual way:

{% highlight javascript %}
 material.on("diffuseMap",
      function(value) {
             console.log("Our material's diffuse map is now: "
                + value ? value.id : "undefined");
      });
{% endhighlight %}

> The idea here is to TODO

## Saving and loading

You can save or load a JSON snapshot of your scene at any time. This snippet will dump the whole state of our scene to
JSON, then load that JSON again to create a second identical scene:

{% highlight javascript %}
var json = scene.json;

var scene2 = new xeogl.Scene({ json: json });
{% endhighlight %}

> xeogl is **[data-driven](https://en.wikipedia.org/wiki/Data-driven_programming)**, meaning that we can define
scene state using JSON data. As a consequence, we can just serialize that state out as JSON at any time, then deserialize it
  to rebuild the scene.
  
## Cloning

<div data-height="665" data-theme-id="11815" data-slug-hash="bdyRQd" data-default-tab="js" data-user="xeolabs" class='codepen'>
<p>See the Pen <a href='http://codepen.io/xeolabs/pen/bdyRQd/'>xeogl: Component Cloning Demo</a> by xeolabs (<a href='http://codepen.io/xeolabs'>@xeolabs</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//assets.codepen.io/assets/embed/ei.js"></script>