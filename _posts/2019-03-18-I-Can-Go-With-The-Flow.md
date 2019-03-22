---
published: true
layout: post
title: I Can Go With The Flow
date: '2019-03-18'
categories:
  - unity
  - shaders
  - flowmap
---

Last year I [tweeted this](https://twitter.com/teodutra/status/1023804477915058176) and it reached a quite large audience. I decided then to transform that thread in this post. This is basically a copy and paste with high quality gifs. Enjoy!

<br/>

---

I've been using flow maps a lot recently to get some cool vfx. In this thread, I'll talk about some things you can do with few math and some textures.

![Flowmap01]({{site.baseurl}}/images/flowmaps/Flowmap11.gif)

<br/>

---

Let's start with a simple flow map shader. You can use [this (not mine)](https://gist.github.com/TarasOsiris/e0e6e6c3b8fdb0d8074b) as starting point. We can set a vector directly to drive the flow. So, `flowDir = float2(1,0)` set the flow to the right. I created this texture with "difference clouds" filter in gimp.

![Flowmap02]({{site.baseurl}}/images/flowmaps/Flowmap01.gif)

<br/>

---

If we expose that vector as a property we can control the flow direction. We can also expose properties to control the lerp speed and flow speed.

![Flowmap03]({{site.baseurl}}/images/flowmaps/Flowmap02.gif)

<br/>

---

In this case, we know that the uv coordinates at the center are `(0.5,0.5)`. Well, we can then compute the difference between the uvs and the center and have a radial flow.

![Flowmap04]({{site.baseurl}}/images/flowmaps/Flowmap04.gif)

<br/>

---

Let's make this additive. Let's also add a mask to drive the border opacity and another one to be added to the current color. By using Unity's default particle texture we get this kinda energy flow. That looks cool already!

![Flowmap05]({{site.baseurl}}/images/flowmaps/Flowmap05.gif)

<br/>

---

Let's add some colors. I added two colors and I'm interpolating them according to the distance between the uv and the center position.

![Flowmap06]({{site.baseurl}}/images/flowmaps/Flowmap13.gif)

<br/>

---

Could I rotate those vectors? Yes! In this case, I'm multiplying by the rotation matrix and by the distance to the center, so we have this smooth rotation instead of a simple texture rotation.

![Flowmap07]({{site.baseurl}}/images/flowmaps/Flowmap06.gif)

<br/>

---

Another example with different settings.

![Flowmap08]({{site.baseurl}}/images/flowmaps/Flowmap10.gif)

<br/>

----

You can also use a texture flow map. This one uses the flow map found [here](https://catlikecoding.com/unity/tutorials/flow/texture-distortion/).

![Flowmap09]({{site.baseurl}}/images/flowmaps/Flowmap08.gif)

<br/>

---

And the final shader you can find [here](https://github.com/teofilobd/APIs-stuff/blob/master/Unity/ShaderXP/Assets/Shaders/FlowMap.shader) :)

![Flowmap10]({{site.baseurl}}/images/flowmaps/Flowmap09.gif)

<br/>

---

For the non-coders, I added the [ShaderGraph version](https://github.com/teofilobd/APIs-stuff/blob/master/Unity/ShaderXP/Assets/Shaders/FlowMap_Radial_SG.ShaderGraph) of the radial shader (you have to install ShaderGraph and SRP to use this).

![Flowmap11]({{site.baseurl}}/images/flowmaps/Flowmap14.gif)

<br/>

## Bonus

My friend [Bruno Croci](https://twitter.com/CrociDB) created a version of this on [ShaderToy](https://www.shadertoy.com/view/ltcyzS).

