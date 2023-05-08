---
title: Writing Robust Software
summary: Exploring approaches that help build reliable software.
date: 2022-10-09T07:06:00+01:00
draft: True
mermaid: True
tags:
- programming
- high availability
categories:
- programming

---

{{<
	figure src="black_knight.png"
	alt="An illustration of Monty Python's Black Knight."
	class="align-left no-top-margin"
	attr="Image by karlestonchew"
	attrlink="https://www.newgrounds.com/art/view/karlestonchew/monty-python-and-the-holy-grail-s-black-knight"
>}}

If you've ever seen Monty Python's and the Holy Grail you may remember the
scene where the film's hero gets stopped by the black knight. As the two knight
do battle the black knight's limbs get chopped off one by one. Despite this the
black knight refuses to surrender claiming "'tis but a scratch!". It is a
(semi) suitable allegory for how robust software should behave. If you lose an
arm you keep fighting with the other. Unlike the black knight though, robust
software should cry for help before it's too late.

Writing software that is robust is important for a number of reasons. Software
that breaks is software that ruins reputations, needs costly time to
troubleshoot and be fixed, and can even lead to security breaches.

A lot goes into deploying robust and secure software, and it is vital that all
layers work together.
There is a lot to be said about highly available deployments, failover,
monitoring and everything that comes into play at the operational
level; but in this article I'll be focusing on the application code itself.

## Error Handling

Most software runs perfectly well when everything is going as expected, it is
the unexpected that tends to throw a spanner in the works. When you've done
a good job at designing your software, and you are confident it handles all
use-cases you could throw at it, you need to consider external factors.
As the software itself usually doesn't change, errors tend to be induced by
external factors. You might experience a temporary issue connecting to your
database, or you're trying to write to a disk that is full - many things can go
wrong. Whilst writing your software you need to ask yourself the following
questions for everything that you do:

1. Could this go wrong?
2. If it does, how do I fix it?




```mermaid
graph TD;
  A-->B;
  A-->C;
  B-->D;
  C-->D;
```