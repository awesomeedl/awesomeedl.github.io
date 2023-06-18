---
title: "Boids: The Flocking Algorithm"
date: 2023-02-15T15:30:25+08:00
description: "How simple rules allow complex behaviors to emerge"
tags: ['Algorithm', 'Simulation']
draft: true
---

(https://www.youtube.com/watch?v=TYPFenJQciw)

I have always been intrigued by the concept of "Emergence", which is the phenomenon when a group consisting of multiple entities, each behaves according to some simple set of rules, exhibits complex behaviors as if the group is a single intelligent entity.  
Emergence is very commonplace in nature. A recent YouTube video by Kurzgesagt – In a Nutshell talks about how dumb beings (aka your body cells) together can become very smart (aka you). There is also the example of an ant colony – while each ant on its own does very limited things, the communication between ants allows massive colonies to grow and sustain. 
This way of creating behaviors is very different than how we usually program. In order to allow multiple things to cooperate, we programmers usually think of an overarching central entity that controls the behavior of all smaller units. This way, it is much easier for the programmer to understand what is going on and much easier to control the behavior (What’s the fun in that?). I remember a couple years ago, I was watching a video (I couldn’t find the video now) about how researchers program drones to fly in a close formation. One obvious approach is to have each drone constantly report its position back to a central computer, and have the computer calculate movement commands for each of them. As you can probably imagine, it is not a very feasible method once you start to scale up the number of drones. Therefore, the researchers  
