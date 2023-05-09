---
title: Woud
summary: Cellular automata forest fire
date: 2023-05-09T20:37:00+01:00
draft: True
tags:
- cellular automata
- SDL2
categories:
- software
---

After reading about Conway's game of life I decided to try my hand at cellular
automata. Building on the base SDL display built for Prime I decided to simulate
a forest where trees go through a life-cycle. They grow from a seed, they
mature, they reproduce, they die and finally rot away. Occasionally, lightning
strikes and a wildfire rampages through the woods (or woud [wɑut] in dutch).

{{< 
	video
	webm_src="woud.webm"
	mp4_src="woud.mp4"
	poster="woud_500.png"
	alt="Output of the Woud program. The cellular automata inspired forest simulating though life and death."
	caption="Output of the Woud program. The cellular automata inspired forest simulating though life and death."
>}}

## Cellular Automata
A cellular automaton is placed somewhere on a grid of cells. The automaton has
a finite number of states, and interacts with its direct neighbourhood. The
lattice of cells goes though generations by mutating each cell according to
some fixed rules. This mutates state of cells on the grid over time. The steps
in time are called generations or cycles.

What is so fascinating about cellular automata is that very complex behaviour
emerges from very simple rules. When many very simple parts interact the
macro-scale outcome is chaotic, and often beautiful. A good example is
[rule 30](https://en.wikipedia.org/wiki/Rule_30) from Wolfram's classification
scheme:

{{<
	figure 
	src="rule30.png"
	alt="rule 30"
	class=""
	attr="Zhiming Wang, CC0, via Wikimedia Commons"
	attrlink="https://commons.wikimedia.org/wiki/File:Rule30-256-rows.png"
>}}

##  The Automaton
### States
The automaton in this simulation is the tree. It goes through 4 stages in it's
natural life-cycle, each drawn a different colour:

- **Growing** - *Light green* - The tree has spawned from a seed and is growing
  to maturity. It's size increases every generation.
- **Mature** - *Dark green* - The tree has reached is full hight. It is sexually
  mature and will go into a regular "bloom-cycle".
- **Blooming** - *Yellow* - The tree is reproducing. For the duration of this
  period the tree spawn new trees in random locations in a radius around it.
- **Dead** - *Brown* - The tree has died after going through a number of bloom
  cycles. It is slowly rotting away, decreasing it's size until it is removed
	from the scene.

One extra state a tree can find itself in is:
- **Burning** - *Red* - The tree is on fire. It's size is reduced every
  generation until it is removed from the scene. Whilst it is burning it may
	set fire to any other trees in a radius around it.

### Rules
Every iteration of the simulation the function Tree::live() is called, moving
the tree through it's states by following the following decision tree:

- If the tree is growing it:
  - Increases it's age and size by one every cycle.
  - If it reaches max age it progresses into mature.
- If the tree is mature it:
  - Increases it's age by one every cycle.
  - If it reaches max age it:
    - Progresses into dead when it has reached it's maxBloomCycles.
    - Otherwise progresses into blooming.
- If the tree is blooming it:
  - Increases it's age by one every cycle.
  - Has a chance to spawn a new tree inside it's bloom-radius.
  - If it reaches max age it progresses into mature.
- If the tree is dead it:
  - Increases it's age and decreases it's size by one every cycle.
  - If it reaches max age it gets removed.
- If the tree is burning it:
  - Increases it's age and decreases it's size by ten every cycle.
  - Has a chance to set fire to any tree inside it's burn-radius.
  - If it reaches zero size it gets removed.

## Scene
The scene is an important part of every simulation or game. It is where we store
all the elements inside the simulation. It can be as simple as an array or more
complex. Because the collision detection in this simulation is more complex than
a simple next-door-neighbour a more complex data-structure was required.
Collision detection is necessary:

- To prevent two trees from spawning in the same place.
- To determine which trees get set on fire when lightning strikes.
- To determine which trees get set on fire when a close-by tree is burning.

This could be achieved in a way that one would traditionally use for cellular
automata: by creating a 2-dimensional array that represents the world and
storing the automata in this structure. Possibly with a secondary unidimensional
array to make iteration more efficient (since the 2-dimensional array would be
sparsely populated).
While this approach would be very efficient (retrieving object at set
coordinates would be an O(1) operation) it also has drawbacks. Firstly the
two-dimensional array would use up a large amount of memory even when mostly
empty. This could be reduced by storing pointers to a secondary array where the
actual data is stored, but even with that approach an empty 1000x1000 pixel
forest would consume 8 Mb of memory (assuming 64 bit pointers). Another
drawback arises should we choose to use trees that are bigger than 1x1 pixel
or have odd shapes. Lastly, it is just not very interesting. If we're building
programs to learn new things and create art we may as well try new things, and
visually interesting ones at that. As such, I decided to implement (and draw) a
quad tree.

## The Quadtree
The [quadtree](https://en.wikipedia.org/wiki/Quadtree) is a tree data structure
that uses a divide-and-conquer mechanism to spatially index objects. It works
by dividing itself into four quadrants. Any object stored in the quad-tree that
fits entirely in a quadrant is stored into that quadrant. If the quadrant gets
too full it will split into another four quadrants, redistributing it's contents
into the newly created sub-quadrants. Any object that doesn't fit entirely in
the new sub-quadrants remains in the top one.

This way retrieving any object that fits into a certain hit-box is relatively
efficient. Instead of testing every single object in the scene you can quickly
discard any object that is far outside the range of your area of interest.



{{<
	figure 
	src="quadtree.png"
	alt="The quadtree overlaid unto the woud render."
	class=""
	attr=""
	attrlink=""
>}}
The quadtree is overlaid on the woud simulation in grey. Note how denser areas
split up into more dub-divisions.

You can find the source-code
[on my gitlab page.](https://gitlab.com/dcolon/woud)

Rendered Doxygen documentaiton is [here.](./docs/index.html)
