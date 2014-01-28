---
layout: post
title: "Quadtree vs Spatial Hashing - Visually Comparing Algorithms"
description: ""
category: 
tags: []
---
{% include JB/setup %}

## Quadtree vs Spatial Hashing

Here is a quick visualization of two algorithms used to reduce the the number of collision checks in a 2d plane. The visualization
and the quadtree algorithm was originally implemented by Timo Hausmann ([https://github.com/timohausmann/quadtree-js/](https://github.com/timohausmann/quadtree-js/)) and
slightly adapted by me. I also wrote the spatial hashing implementation.

<iframe src="/assets/code/2014-01-26/quadtree/examples/insert_retrieve.html" width="730" height="440" frameBorder="0">
</iframe>
<iframe src="/assets/code/2014-01-26/spatialhash/examples/insert_retrieve.html" width="730" height="440" frameBorder="0">
</iframe>

### Background

Lately I've been working on a browser platform game using JavaScript and WebGL accelerated canvas powered by [PIXI.JS](https://github.com/GoodBoyDigital/pixi.js/)

For the game feeling I'm targeting, it's important to have a lot of objects at the same time.
I noticed that the game slowed down and sometimes frooze completely when I had a too many objects at the same time.
The bottle neck was, as I had anticipated, the collision handling. By checking each object for collisions with every other objects,
you get a nasty non-linear complexity.

An old problem, of course, but I didn't quite know what the solution would be, so I started googling. One approach I stumpled upon first
was the [quadtree](http://en.wikipedia.org/wiki/Quadtree) data structure (see the wikipedia link for an explanation). I tried a few implementations, but I found the simplest was [one by Timo Hausmann](https://github.com/timohausmann/quadtree-js/).
Unfortunately it didn't seem to reduce the number of collisions checks very much. But why was it slow?

One hack I used to reduce the number of collisions was to limit the number of collided objects per object to 2 or 3. This increased the speed,
but made the collision detection inaccurate.

Timo had the good taste to include an excellent visualization of the algorithm in the github repo.
His visualization shows a green square (following the mouse pointer) and highlights all the squares that it is collision checked against.

After some googling for alternatives I decided to try [spatial hashing](http://www.gamedev.net/page/resources/_/technical/game-programming/spatial-hashing-r2697). I implemented one spatial hash using the same interface as Timo's quadtree implementation. I could then save myself a lot of work
by retrofitting his visualization for my spatial hashing implementation.

Now I could easily compare both algorithms. To me, was pretty obvious that spatial hashing was more efficient.
This was also shown as I plugged in my spatial hash implementation - the performance problem were greatly reduced and I could have 80 objects without the collision detecting became a bottle neck.

It is of course possible, that there are cases where a quadtree is a more efficient data structure than a spatial hash, but for uniform size objects, spatial hashing seems quicker.

### The power of visualization

Algorithmic complexity is usually represented with using big O notation. While trying to find a good algorithm for reducing the
number of objects to collision check, I realized the power of visualizing algorithms. Using visualizations 
I gained more insight in the quadtree algorithm than I could have gotten by
just reading a formal description of it.

### The visualizations

I added run-time control over some parameters using dat.gui. Now you can really compare the algorithms.
What I don't measure is actual cpu time used, but this gives a pretty good hint of how they work.

### Links

* [Timo Hausman's quadtree implementation](https://github.com/timohausmann/quadtree-js/)
* [My spatial hashing implementation](/assets/code/2014-01-26/spatialhash/spatialhash.js) (doesn't have its own github repo yet): 

