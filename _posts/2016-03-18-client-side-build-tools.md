---
layout: post
title: Client Side Build Tools
---

Anyone know any good ones? Many of the server-side frameworks these days have tooling to 
support JS and CSS that fits into their methodolgy, or at least have places 
to plugin your favorite tools, but not so much for "pure" client-side.

In someways its too bad that we developers spend so much time re-inventing 
wheels. But on the other hand using Make (or even Ant) to build a relatively 
modern web app seems a) to break out the build into a completely different language set, 
and b) is not that well integrated into tooling. 

My main needs:

1. Support for some form of modularization. After poking around some, I settled on the main one being CommonJS syntax as it seems to be more closely aligned with how ES6 will work.
1. Ability to inline RequireJS/AMD modules - I need to deploy some code to a system that already exposes/uses AMD. I don't intend to build large chunks of functionality, so I mainly just need to concat (in order) + minify. Full AMD support is fine.
1. Support CSS with contat + minification.
1. Support migrating CSS to SCSS.
1. Fire up one or more servers to do local testing with. In one scenerio I am deploying files to a different sub-domain, so I need to have HTML on localhost:8080 and a JS file on localhost:8888 to allow to test to ensure that a cross-domain issue wasn't accidentally introduced. For those interested, an enabling function is placed directly onto the HTML pages which then calls the more complex functions hosted on the asset server.
1. I also want some mechanism to run a 'vendor build', in my case I'm likely to use the bootstrap code and wrap their CSS with a nesting rule such as '.bsf .component' in order to prevent conflicts with a poorly done CMS CSS set.


I looked at using Gradle, which I am somewhat familiar with, 
but it seemed that the plugins I could locate would have required a lot of glue code
 (in the form of build.gradle coding) to work together in the format I would like.

RequieJS offers its own build pipeline, although one which doesn't meet all of my needs. 

I looked at Mimosa which doesn't seem to have been updated in a while and 
which seems to have been mostly a one man show. I also looked at Brunch which seems to 
have a couple developers and has more recent updates. However the 
documentation on customization was not as intuitive as I would have liked, so before diving in I looked around more.

At the moment Grunt seems to be biggest JS build tool. Having looked at some of it's files, 
I went screaming in the other direction. There are certain times when I find it a useful 
communication tool to have things documented via code, but this seems like a serious case of overkill on that front.

So I have currently settled for Gulp. It does seem to be a good bit cleaner than Grunt. However, 
I really don't want to learn dozens of different gulp and/or npm modules just to get a build up 
and running. Well, I went ahead and did it and things seem to be mostly working. However 
they really do seem to be pushing the limits of modularization in the amount of 
different options not bundled. Well, the all-in-one tools have nice simple configs 
until you need something custom, so I guess there's a price either way.

## Update
Two years later and I have several projects that as noted else where build files that _must_ 
fit into a SaaS CMS with limited flexibility.  One of my builds in particular is fairly scary 
due to the number of different modules it's pulling in.