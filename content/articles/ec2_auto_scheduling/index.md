---
title: EC2 Auto Scheduling
summary: Saving on cloud cost by scheduling lower environments.
date: 2022-08-20T12:22:17+01:00
draft: true
tags:
- aws
- ec2
- cost
categories:
- aws

---

{{<
	figure src="cost.jpg"
	alt="A drawing of a stopwatch in front of money and creditcards."
	class="align-left no-top-margin"
	attr="Image by photoroyalty"
	attrlink="https://www.freepik.com/vectors/coloured-background"
>}}

As your AWS estate grows, so will your bill. The flexibility of the cloud makes
it simple to spin up an EC2 instance here or there, but to keep costs in check
it pays off to use the "pay for what you use" model to your advantage.

A production environment will most likely have to run 24/7. For workloads like
this it is sensible to