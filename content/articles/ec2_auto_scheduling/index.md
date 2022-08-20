---
title: EC2 Auto Scheduling
summary: Saving on cloud cost by scheduling lower environments.
date: 2022-08-20T12:22:17+01:00
draft: true
tags:
- aws
- ec2
- lambda
- devops
- cost
categories:
- aws

---

{{<
	figure src="cost.jpg"
	alt="A drawing of a stopwatch in front of money and credit-cards."
	class="align-left no-top-margin"
	attr="Image by photoroyalty"
	attrlink="https://www.freepik.com/vectors/coloured-background"
>}}

As your AWS estate grows, so will your bill. The flexibility of the cloud makes
it simple to spin up an EC2 instance here or there, but to keep costs in check
it pays off to use the "pay for what you use" model to your advantage.

A production environment will most likely have to run 24/7. But test
environments will sit idle for a lot of the time. Ideally you'd be able to
destroy the entire environment when not in use, and simply provision a new
environment when you need it. This is often not possible, either because your
environment does not support being treated as cattle, or because you just don't
have the capability or capacity to create an environment suitable for this
treatment.

## Simple scheduling

A simple solution; and the first iteration of scheduling we jumped to; is to run
a lambda script in the morning and again in the evening to start/stop all
instances used by test environments. This may not have an impact on storage
cost, but reduces instance cost by nearly 60%.

There were limitations to this approach though.