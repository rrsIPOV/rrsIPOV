---
layout: post
title: Limits of Modularization Featuring Gulp
---

I've been working on some client-side tooling, which unfortunately 
doesn't fit well into any particular build tool given that it's basically a 
bunch of files being loaded into a CMS. The CMS in question is a SaaS platform that doesn't feature 
custom script support such as file concatenation or gzip on files, it just serves 
them as-as. To make a long story short, I decided to learn a bit of GulpJS. I'd looked briefly at 
Brunch but it seemed to have just a little bit too much inflexibility in that I'm suck deploying 
my files to 2 different hosted SaaS solutions where I have to fit into their rather strict limitations.

Currently, I'm writing some of the code in a CommonJS format, since I'm bundling in the build and 
this is a bit cleaner looking code than using RequireJS/AMD callbacks. The problem is that browserify is great, 
but support for use with Gulp seems to have a majorly annoying point of friction, which I think is fairly common.
If you search for gulp + browserify, one of the articles you're find is 
<a href="https://wehavefaces.net/gulp-browserify-the-gulp-y-way-bb359b3f9623#.3t8g53ej9">gulp + browserify, the gulp-y way</a> , 
alas it seems that the recipe provide is no longer gulp-y. There is an alternative, but said alternative basically pushes 
what in my mind should be a small common module or function off to be copied and pasted into a multitude of users of Gulp.

Basically this boils down to a couple of things: Gulp using a virtual filesystem called vinyl. I can certainly see that 
being able to pass a full record the build file info along each pipe is a bit advantage; 
however given that Gulp is at version 3.9+ it seems like at this point there should be a more reasonably succinct, 
maintained method for this. It could be a wrapper built into Gulp or a side project with good support and great docs.

The current vinyl-source-stream module appears to be very powerful, 
but now the Gulp developers are trying to get people to switch and re-use "normal" npm modules instead of building thousands of custom wrappers.
However this approach has a proble in that 
a) there isn't a simplified method to include stream merging / globbing, which leads to 
b) a relatively large chunk of what seems like boiler plate for every implementor which could potentially be avoided.

To sum up. I'm all for avoiding certain types of monoliths. However it is possible to go too far in the 
opposite direction. Even with NPM's modern support for modularization, there comes a point where things get 
too modularizarized.  I think the key, which is difficult, is to review from someone consuming your project to see where
there needs to be more helper functions.  Getting the cleanest archecture need to take a back seat to actually 
solving real problems and to enabling DRY.

### Update
Just to clarify what I want to do is something like the following. Say I have 3 entry points:


1. main.js - on all nearly all pages
1. user.js - on only normal user pages
1. admin.js - admin pages use a different set of functionally than user.js


So I want to pull './src/js/*.js' into my build pipe with any sub-modules placed into 
sub-directories to keep things a bit more organized.  It's certainly possibly to make 
this work w/ gulp + browserify, but it seems unnecessarily painful.
