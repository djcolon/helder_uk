---
title: Optimising software using Callgrind
summary: Learn how to use Callgrind to find bottlenecks in your software
date: 2023-05-11T19:25:00+01:00
draft: False
tags:
- software development
- optimisation
- debugging
- tools
categories:
- software development
---

{{<
	figure src="valgrind_logo.png"
	alt="The valgrind logo."
	class="align-left no-top-margin"
	attr="Copyright: Valgrind™ Developers"
	attrlink="https://valgrind.org/"
>}}

With extremely powerful hardware having become a commodity in the present day
and age, optimisation of software has become less of a concern to developers
than it once was. Whilst 4KB of RAM was enough for the Apollo Guidance
Computer to facilitate a mission to the moon, opening a modern web-app will see
a single chrome tab bloat to a gigabyte. When everyone has a large amount of
resource available, concerns such as maintainability and development time take
precedence over optimisation. However, when you're writing software that needs
to be performant, sometimes you will need to go to greater lengths to optimise
your code. In some situations, optimising the use of resource use can yield
significant cost savings. In this article we'll explore how to use Callgrind,
a sub-component of the [Valgrind](https://valgrind.org/)
framework, to locate where to best direct our attention.

*Other than profiling, there is much more the Valgrind toolkit can do. Have a
look at [their website](https://valgrind.org/) to see what other gems it holds.*

## What does Callgrind do?

Callgrind is a profiler; what it does is analyse calls within the compiled
program. All programs consist of a number of functions that get called from
other functions. The execution of all these functions in a particular way is
what takes up the processing time used by the program. This forms a pyramid of
functions being called from other functions with your `main()` at the top.

`main()` will contain 100% of the compute power used by your application, with
subsequent function calls making up parts of that. If we had a simple
application that had a loop in the main function that called function `foo()`
1 time, function `bar()` 9 times, and then quit the application, the so called
call graph would look something like this:

```
main() - 100%
├─ foo() - 10%
└─ bar() - 90%
```

To quote [the manual](https://valgrind.org/docs/manual/cl-manual.html):

> ... Callgrind extends this functionality by propagating costs across function
> call boundaries. If `foo()` calls `bar()`, the costs from `bar()` are added
> into `foo()`'s costs. When applied to the program as a whole, this builds up
> a picture of so called inclusive costs, that is, where the cost of each
> function includes the costs of all functions it called, directly or
> indirectly.

Where "cost" is the amount of processing time spent executing that function.

By analysing this callgraph, we can find good spots to direct our attentions
when optimising the program. If we can optimise a function that has a high cost,
we can optimise the result of our work - as opposed to wasting time optimising
a function that contributes very little to the overall cost of the program.

We'll have a look at an example to make it clearer what this means - and how it
can benefit you.

## Running Callgrind

To demonstrate the use of Callgrind, we'll analyse a commit of my project,
Woud, created as a demonstration. It'll be useful to have read
[the article I wrote on it]({{< ref "woud">}}) to follow along with the example.
You can run the example on a Linux distribution of your choice, the use of Linux
and the surrounding tools falls outside of the scope of this article.

*Do note that there are more profilers than callgrind, and if you search the web
you're likely to find one that suits your language and OS of choice - and the
techniques discussed in this article will carry over.*

You'll need to be set up with git, some sort of text editor, and c++ development
tools (make, g++), dependencies for Woud (libSDL2, libSDL2_ttf), and of course
the Valgrind framework.

First, we'll use git to clone Woud into some directory:
```sh
git clone https://gitlab.com/dcolon/woud.git
```
and check out our example commit:
```sh
git checkout d9b5d45b3db2d7062046caa8da19c3be2f94b845
```
Finally, we'll build Woud for us to run it in Callgrind. We'll use the included
makefile here, but when you use Callgrind on any of your own programs, make
sure you compile the software using your compiler's debug flag (in the case of
g++ this is -g):
```
make debug
```

Which should yield the following output:

```sh
Setting debug flags
Creating build dir
mkdir -p build
g++ -c -g -Wall -std=c++0x src/utils/dccolor.cpp -o build/dccolor.o
g++ -c -g -Wall -std=c++0x src/objects/tree.cpp -o build/tree.o
g++ -c -g -Wall -std=c++0x src/quadtree.cpp -o build/quadtree.o
g++ -c -g -Wall -std=c++0x src/sdlwindow.cpp -o build/sdlwindow.o
g++ -c -g -Wall -std=c++0x src/main.cpp -o build/main.o
Linking
g++   build/dccolor.o build/tree.o build/quadtree.o build/sdlwindow.o build/main.o -o build/woud -lSDL2
Finished building woud
```

we can then run the compiled executable using Callgrind:
```sh
valgrind --tool=callgrind build/woud
```

This will launch Woud wrapped in Callgrind. Because of the overhead of
Callgrind, the application will run considerably more slowly. Leave the
application to run for a good while (let a decently sized fire rage through a
forest) before you quit the application (in Woud, use ctrl+q).
The output over the process should look something like this:
```sh
==34384== Callgrind, a call-graph generating cache profiler
==34384== Copyright (C) 2002-2017, and GNU GPLd, by Josef Weidendorfer et al.
==34384== Using Valgrind-3.19.0 and LibVEX; rerun with -h for copyright info
==34384== Command: build/woud
==34384== 
==34384== For interactive control, run 'callgrind_control -h'.
Starting woud
SDL compile-time version 2.24.1
SDL runtime version 2.24.1
==34384== Warning: noted but unhandled ioctl 0x6444 with no size/direction hints.
==34384==    This could cause spurious value errors to appear.
==34384==    See README_MISSING_SYSCALL_OR_IOCTL for guidance on writing a proper wrapper.
==34384== brk segment overflow in thread #1: can't grow to 0x4882000
==34384== (see section Limitations in user manual)
==34384== NOTE: further instances of this message will not be shown
==34384== 
==34384== Events    : Ir
==34384== Collected : 23582247433
==34384== 
==34384== I   refs:      23,582,247,433
```

You should also see a file was created called `callgrind.out.####`. This file
contains the results of our analysis. You can view my trace
[here](callgrind.out). Or download it by right clicking and choosing *save as*.

## Analysing the output

To analyse the output of Callgrind we'll use an application called
[KCachegrind](https://github.com/KDE/kcachegrind). It will allow us to visualise
the results of our run. Once it's installed open up KCachegrind and open the
`callgrind.out.####` file generated in the previous section. It should look
something like this:

{{<
	figure src="kcachegrind.png"
	alt="KCachegrind for the output of Woud."
	class="align-left no-top-margin"
	attr=""
	attrlink=""
>}}

What you're looking at is a list of all the function calls made inside your
program, organised by (cumulative) cost. As you can see here, 99.99% of our CPU
time is spent in `main()`, 68.47% in `spreadFire()`, and so forth.

The callgraph tab (bottom right) is a very interesting tab to explore. It shows
what functions call which others functions in a tree view, and gives us a great
visual representation of where most cost in the program sits:

{{<
	figure src="callgraph.png"
	alt="The callgraph for our run of Woud."
	class="align-left no-top-margin"
	attr=""
	attrlink=""
>}}

In our callgraph it becomes clear that the program spends most of its time in
the SDLWindow::spreadFire function:

{{<
	figure src="spreadFire.png"
	alt="The callgraph for spreadfire."
	class="align-left no-top-margin"
	attr=""
	attrlink=""
>}}

If we had to choose a place to start optimising our program, this would be an
excellent place:

```cpp
void SDLWindow::spreadFire(Tree* ATree) {
	// Generate a rect to spread in
	Rect ADangerZone = {
		ATree->position.x - SPREAD_RADIUS,
		ATree->position.x + SPREAD_RADIUS,
		ATree->position.y - SPREAD_RADIUS,
		ATree->position.y + SPREAD_RADIUS
	};

	// Get trees that might catch fire
	std::list<Tree*> ATreesInDangerZone;
	scene->retrieve(&ATreesInDangerZone, ADangerZone);
	// Do a collision check on all of them
	for(std::list<Tree*>::iterator ATarget = ATreesInDangerZone.begin(); ATarget != ATreesInDangerZone.end(); ATarget++) {
		if(ATree->collides((*ATarget)->position, SPREAD_RADIUS)) {
			// It's caught fire!
			(*ATarget)->state = Tree::STATE_BURNING;
		}
	}
}
```

We can see that 68% of the program's cost is spent in this function. A lot of
our cost is spent on retrieving potential hits from the QuadTree, but there
isn't all that much that can be done about that... The same goes for the many
calls to `pow()` when computing the distance between trees.

It does look like we're spending an inordinate amount of time allocating memory
for a list to Tree*, moving pointers around and then free-ing everything after.
We could look into moving our collision check into a separate function, and
calling it directly from a QuadTree::retrieve type function that takes a
function to run on potential collisions as an argument.

As an alternative approach we could look at a higher level and think of ways we
could decrease the number of calls to `spreadFire()`. We currently run this for
any tree that is on fire, for every iteration until it has burnt up. A simple
way to vastly increase the running speed of the program is to be more
conservative about running this function. Could we run it on every other tree
per iteration and interleave them? Do we just run it for the first cycle the
tree is on fire, and then stop? There are a number of options we could pursue.
Now that we've found this function to be expensive, it is a prime target to use
less.

What is so useful about a tool like CallGrind is that it allows us to find the
bottlenecks in a program that have the most impact on performance. It gives us
a quantified view of the places in the code that occupy our CPU. That way we
spend our time more effectively, rather than chasing performance in places that
don't actually impact the bottom line.

In a world where CPU cycles are cheap, and developers are expensive, squeezing
the last bit of performance out of your code is often an afterthought. But when
you do need that edge, a profiler is an extremely useful tool to have under your
belt. Finally, when your application is running at scale, a small decrease
in compute use can translate to significant cost savings.
