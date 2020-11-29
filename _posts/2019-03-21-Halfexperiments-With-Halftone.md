---
published: true
layout: post
title: Halfexperiments with Halftone
date: '2019-03-21'
image: /assets/images/blog/halftone/HalftonePP_3.gif
categories:
  - unity
  - shaders
  - halftone
  - experimental
---

Some experiments I did last year with [halftone](https://en.wikipedia.org/wiki/Halftone).

<br/>

I was playing with halftone in a post processing shader. The concept of halftone is basically you get the fraction part of a scaled uv, subtract from the center (0.5, 0.5) and perform a step with a given radius in order to get circular shapes (the halftone itself):

```ShaderLab
...
float2 center = float2(0.5, 0.5);
float2 newUV = frac(i.uv * float2(_Columns, _Rows)); // number of columns and rows of circles
float ht = distance(newUV, center );
ht = step(ht, _Radius);

float4 htColor = float4(ht,ht,ht, 1);
...
```

<br/>

The difference when working with that as post processing is that your uv will be the screen coodinates normalized. In my experiments, I tried some kind of chromatic aberration where each color channel is displaced according to its intensity. I also added a noise to disturb the direction of the displacement over time. I won't share the code because it's ~~a mess~~ too experimental.


Playing with parameters

![Halftone01]({{site.baseurl}}/assets/images/blog/halftone/HalftonePP.gif)

![Halftone02]({{site.baseurl}}/assets/images/blog/halftone/HalftonePP_2.gif)

Playing with parameters

![Halftone03]({{site.baseurl}}/assets/images/blog/halftone/HalftonePP_3.gif)

Dramatic tone

![Halftone04]({{site.baseurl}}/assets/images/blog/halftone/HalftonePP_4.gif)



