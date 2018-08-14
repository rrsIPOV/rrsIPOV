---
layout: post
title: Damn You JIRA - Take 1
---

## Or why Enums are Great.

It's always interesting to watch a product evolve. Will they alienate early adopters in order to 
fix the inevitable early architectural issues that are uncovered as the product grows and the various 
platforms evolve? Will they use early success to allow their product to rest on it's laurels 
while others are experimenting with new technologies? Will they somehow manage to walk 
the fine line between those two positions as well as the various other opportunities and pitfalls?

Way back when, I tried out a ton of "issue trackers", looking for something that 
a) didin't suck and b) wasn't unaffordable. Back them, there were a ton of perl and php 
scripts which did basic issue / to-do lists. Of course they were all web 1.0 style, 
and if you were lucky they had a few simple javascripts to keep every page 
from reloading everything. Amoung the various tools, I tried out Jira, and liked a nuimber of the features, 
but at the time there was no hosted version, and there really is a fairly large bit of work to setup a new instance 
(as well as to update one) and to configure projects, etc... Also the actual agile 
tools in Jira was in a separate add-on which was non-intuitive to get working. Frustratingly there were 
one or two fairly nice commercial tools, but with per-seat at $1300+ they were well over our budget at the 
time. There were and seem to perpetually be, a handful of really interesting 
open source projects that always stalled out right before they managed to become truely useful to a wide enough audience.

Anyway, I'm back using Jira now as a user. And damn but some of the same things that 
annoyed me about it the first time around aren't still part of the product.

Specifically, I'm part of a small team (in a much bigger organization) and while we try to 
do new features using agile, we also have a ton of in-house work based on the legacy apps that we support, 
including a lot of things that get hung up with external vendors or 
internal decision makers. So we end up with a rather messy set of tasks/stories as Jira is our primary tool.

The other day I tried setting up a custom issue query 
and then linking that to a dashboard. The idea is to get a grid view of 
open issues sorted by priority and then status 
(we have 2 backlog statuses as well as "vendor", etc). However, wow, 
the proiority sorts like either "High, Medium, Low, Critical" or 
"Critial, Low, Medium, High" which is enough to send me on this rant. Also 
the statuses are set on the backend (admin) and 
are global (which to some degree could be seen as a bonus nice), however our organization now has 
around 100 Jira projects (of which the projects I work on have about 7), 
so my Jira admin naturally declined to sort them for me (and I don't have access).

So I guess I'm ranted out. It would be super great if the Jira Query Language 
could add something like a case or some other mechanism to allow custom sort as reporting is important.

## Update
Of course having some level of organization with configuration below the "master" level would be great too. 