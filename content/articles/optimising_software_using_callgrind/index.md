---
title: Optimising software using Callgrind
summary: Learn how to use Callgrind to find bottlenecks in your software
date: 2023-05-11T19:25:00+01:00
draft: True
tags:
- optimisation
- debugging
- tools
categories:
- software
---

{{<
	figure src="valgrind_logo.png"
	alt="The valgrind logo."
	class="align-left no-top-margin"
	attr="Valgrind™ Developers"
	attrlink="https://valgrind.org/"
>}}

With extremely powerful hardware having become a commodity in the present day
and age, optimisation of software has become less of a concern to developers
than it once was. Whilst 4KB of RAM was enough for the Apollo Guidance
Computer to facilitate a mission to the moon, opening a modern web-app will see
a single chrome tab bloat to a gigabyte. When everyone has a large amount of
resource available, concerns such a maintainability and development time take
precedence over optimisation. But when you're writing software, or a subsection
of a piece of software, that needs to be performant. Sometimes you will need to
go to greater lengths to optimise your code. In this article we'll explore how
to use Callgrind, a sub-component of the [Valgrind](https://valgrind.org/)
framework to locate where to best direct our attentions.

## What does Callgrind do?

Callgrind is a profiler, what it does is analyse calls within the compiled
program. All programs consist of a number of functions that get called from
other functions. The execution of all these functions in a particular way is
what takes up the processing time used by the program. This forms a pyramid of
functions being called from other function with your `main()` at the top.

`main()` will contain 100% of the compute power used by your application, with
subsequent function calls making up parts of that. If we had a simple
application that had a loop in the main function that called function `foo()`
1 time, function `bar()` 9 times, and then quite the application, the so called
call graph would look something like this:

```
main() - 100%
├─ foo() - 10%
└─ bar() - 90%
```

To quote [the manual](https://valgrind.org/docs/manual/cl-manual.html)

> ... Callgrind extends this functionality by propagating costs across function
> call boundaries. If function foo calls bar, the costs from bar are added into
> foo's costs. When applied to the program as a whole, this builds up a picture
> of so called inclusive costs, that is, where the cost of each function
> includes the costs of all functions it called, directly or indirectly.

Where "cost" means that the application spends a lot of processing time
executing that function.

By analysing this callgraph, we can find good spots to direct our attentions
when optimising the program. If we can optimise a function that has a high cost
we can optimise the result of our work - as opposed to wasting time optimising
a function that contributes very little to the overall cost of the program.

## Running Callgrind

To demonstrate the use of Callgrind, we'll analyse a commit of my project
[Woud]({{< ref "woud">}}) created as a demonstration. I'll be using a Linux
distribution (in this case arch) in this example, and will assume proficiency in
the use thereof.

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
Callgrind the application will run considerably more slowly. Leave the
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
contains the results of our analysis.

## Analysing the output

To analyse the output of Callgrind we'll use an application called
[KCachegrind](https://github.com/KDE/kcachegrind). It will allow us to visualise
the results of our run. Once it's installed open up KCachegrind and open the
`callgrind.out.####` file generated in the previous section. It should look
something like this: