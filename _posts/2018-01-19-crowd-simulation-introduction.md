---
published: true
layout: post
date: '2018-01-19'
categories:
  - crowd simulation
title: Introduction to Crowd Simulation
---
This post is the first of a series about crowd simulation. "_Why crowd simulation specifically ?_" one might ask. And I say it's because I studied crowd simulation for several years and, although I do not work on this subject anymore, I do like to talk about it. 

In this article, I will give an overview about crowd simulation (no code this time), but in the following articles in this series I will focus mostly on crowd navigation. 

# But first, what is crowd simulation about ?

Crowd simulation is a multidisciplinary field that focuses on simulating the behavior of several entities (let's call them '_agents_') as well as the interactions among them that might emerge naturally. These agents not necessarily represent humans, they can be fishes, birds, robots or whatever you want. Consequently, we are not limited to simulating walking behavior but also flying or swimming, example. 

Simulating a crowd is challeging because it involves not only steering the agents (what itself is complex enough), but also making them look visually different and having different animations. Not to mention the need of rendering several of them (sometimes in realtime). 

# Application

There are different applications for crowd simulators. They can be used in videogames, such as we have seen in Kameo, Assassin's creed series and Dead rising series, for example. The following videos show footages of Assassin's Creed Unity and the recent Planet Coaster (check [this gamasutra article](https://www.gamasutra.com/view/news/288020/Game_Design_Deep_Dive_Creating_believable_crowds_in_Planet_Coaster.php)).

<br>
<iframe width="560" height="315" src="https://www.youtube.com/embed/zJppUe1D-ZA" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/xogpwPAd2rA" frameborder="0" allowfullscreen></iframe>

They are also useful in the architectural engineering field. When designing a stadium, for example, one need to place exit doors in order to optimize the evacuation of people in an emergency situation. Crowd simulators can be used to help this kind of design. Some simulators can even give information about the build up pressure when human traffic jams happen.

<iframe width="560" height="315" src="https://www.youtube.com/embed/97QIFsVqeh4" frameborder="0" allowfullscreen></iframe>

Some crowd simulators are intended to study the human behavior and try to faithfully replicate it from ground truth. Several emerging patterns can be observed within real crowds interactions and these simulators try to reach that.

<iframe width="560" height="315" src="https://www.youtube.com/embed/JbQe19DyUMc" frameborder="0" allowfullscreen></iframe>

Finally, crowd simulators can be used in movies and TV series, of course. Lord of the rings, world war z, the walking dead, game of thrones, just to cite a few examples.

<iframe width="560" height="315" src="https://www.youtube.com/embed/cr5Cwz-5Wsw" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/jC2igrqqIC0" frameborder="0" allowfullscreen></iframe>

> Trivia: the Massive software was specially developed for Lord of the rings.

> Trivia 2: Golaem is the Maya plugin used in TV series like Game of Thrones and The Walking Dead.

The demands for a simulator will depend on its objective. For movies, both visuals and behaviors have to have a high quality, whereas real-time execution is not expected. Human behavior studies and civil engineering applications expect "realistic" behavior, real-time execution, but not much from visuals. In games, we need real-time interaction, pleasant visual and pleasant (but not necessarily realistic) behavior. 

> Realism is subjective, that's why I used the quotation marks when saying "realistic" behavior. A simulator can have a perfect execution, organizing everybody in a beatiful flow, where everybody avoids collision with each other, forming lanes in the counterflow. Is that realistic? It might be, if we are simulating an army, for example; and it is not if we are trying to simulate the Shibuya crossing (Tokyo, Japan).

# Navigation

Perhaps the most important, and consequently the most investigated, part of a crowd simulator is the navigation system. A simulator needs to be able to make agents reach their goals while traversing complex environments and avoiding collisions with other agents or obstacles. We can separate the navigation in two categories: global and local.

The global navigator is responsible for managing the global goals of the agents. For example, an agent starts at A and has to go to B, once it gets to B the global navigator update its objective and now the agent knows that it has to go to C, and so on. You probably already heard about the algorithm A*, it is a extended Djikstra broadly used for global navigation or at least used as inspiration for some other algorithms. If instead of moving point-to-point we are interested in moving from a set of points within an area to another set of points, we can resort then to a navigation mesh, that represents the navigable space within an environment and this way provides more options of movement.

![nav_systems.jpg]({{site.baseurl}}/images/nav_systems.jpg)
Difference of paths when moving between waypoints and using a navigation mesh (found this image [here](https://docs.unrealengine.com/udk/Three/AIAndNavigationHome.html)).

A local navigator is responsible for making the agent move safely from A to B. The main task here is to avoid collisions with static and dynamic obstacles. There are several approaches to solve this, agents can: be endowed with cooperative rules, be treated as particles in a particle system, move according flow fields, react according to their synthetic vision, adjust their velocities reciprocally, among many others approaches that you can find the literature.    

<iframe width="560" height="315" src="https://www.youtube.com/embed/1Fn3Mz6f5xA" frameborder="0" allowfullscreen></iframe>

# Engines

Both Unity and Unreal come with global and local navigators. For global navigation, they use navigation meshes and for local navigation they use a model of reciprocal collision avoidance (RVO) (Unreal also comes with Detour crowd) ([Unity's doc](https://docs.unity3d.com/Manual/nav-InnerWorkings.html) and some info from [Unreal](https://wiki.unrealengine.com/Unreal_Engine_AI_Tutorial_-_2_-_Avoidance)). 

# Conclusion

This was a brief introduction to crowd simulation, not very deep but with some useful information to start. The next posts in this series will be related to navigation, I'll start with global navigators and then move to the local ones (this order might change or this might never happen as well :P).

Here are some references for those that want more info about this field.

Books

[Simulating Heterogeneous Crowds with Interactive Behaviors](https://www.amazon.com/Simulating-Heterogeneous-Crowds-Interactive-Behaviors/dp/1498730361/)

[Crowd simulation](https://www.amazon.com/Crowd-Simulation-Daniel-Thalmann/dp/144714449X/)


Articles

[Flocks, Herds, and Schools: A Distributed Behavioral Model (All started here)](http://www.macs.hw.ac.uk/~dwcorne/Teaching/Craig%20Reynolds%20Flocks,%20Herds,%20and%20Schools%20A%20Distributed%20Behavioral%20Model.htm)

[Social force model for pedestrian dynamics](https://arxiv.org/pdf/cond-mat/9805244.pdf)

[Continuum crowds](https://pdfs.semanticscholar.org/f6c6/ae791bf0696a4053e171fd27918e8fc17ab3.pdf)

[Reciprocal Velocity Obstacles for Real-Time Multi-Agent Navigation](http://ai2-s2-pdfs.s3.amazonaws.com/8e49/f498a6494758b741f993ea9088ce8c1099d3.pdf)

[Gradient-based steering for vision-based crowd simulation algorithms (my last work on crowds)](http://people.rennes.inria.fr/Julien.Pettre/pdf/EG2017Dutra.pdf)
