---
layout: post
title: Grails 3 with Angular 4 combined WAR
---


The Grails 3.2+ 'angular' profile is a really good start on 
Angular 2/4+ development with a Grails backend. The main issue, 
like so much of Grails is limited documentation and/or an assumption 
of relatively deep expertise in the tools that they are wrapping.

In this case one potential issue is deploying and running as a single web-app. The angular profile 
creates a multi-project gradle build with a 'server' and 'client' sub-project. There is a 
guide at <http://guides.grails.org/angular2-combined/guide/index.html> which attempts to walk through 
how to combine these into a single project. However it seems that this guide is already out of date - 
downloading and attempting to run the build results in various 
errors and it becomes apparent on looking that the client build files they are 
using are quite a few versions out of date with the node packages. 

Also, even with some fixes, applying the instructions above
to the a modern project seems to result in a build that constantly hangs (on Windows). I 
have seen various reports on this and it appears to be changes to node, npm, and yarn combined 
with a gradle wrapper (for them) that hasn't been updated in a while.  Given the velocity 
of these JS / node projects it is understandable, although frustrating. I found 
that because the client build was requiring me to use task manager to kill it, 
I greatly prefered to keep client and server seperate for development.

So after fighting to get a unified build, I decided to stick with the 
seperate sub-projects for development but to work through the 
issues with gradle to combine the builds for production deployment.

The first thing I noticed was that in the 
client/build.gradle there is no build/assemble task defined or accessible. In this case, 
adapting the instructions from the guide I was able to add one like:

```groovy
task buildClient(type: YarnTask, dependsOn: 'yarn') {
    group = 'build'
    description = 'Compile client side assets for production'
    args = ['run', 'build']
}
```


Note that in the instuctions they use NpmTask in the build.gradle file, 
but the current profile as updated to using YarnTask.

If you plan to deploy to some context other than ROOT, then you need to update your 
`client/package.json` file, in the scripts block do something like - not the "build" line which is the change:

```json
  "scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build --prod --base-href=\"/my-app/\"",
    "test": "ng test --watch=false",
    "lint": "ng lint",
    "e2e": "ng e2e"
  },
```


The other thing I did was to update the node plugin 
and the yarn version used as they seemed to be a bit out of date. Again in client/build.gradle:

```groovy
plugins {
    id "com.moowork.node" version "1.2.0"   // was "1.0.1"
}
node {
    version = '7.5.0'
    yarnVersion = '1.0.2'   // was  '0.21.3'
    download = true
}
```

**Update**
Opps and once again node moves fast - my new configuration is as follows.  Note that the yarn wrapper now triggers errors, 
so I'm setting download to false.  This means I need node, npm and yarn instaled locally... but I probably already need that 
because running them "through" gradle isn't really workable.

```groovy
plugins {
    id "com.moowork.node" version "1.2.0"
}

node {
    version = '8.11.3'
    yarnVersion = '1.9.2'
    download = false
}
```

Now we need to copy the files, 
which will be created in client/dist into the war. Luckily a little digging turned up 
various pointers that allowed me to get this working.

In server/build.gradle:

```groovy
war.dependsOn(':client:buildClient');
war {
    baseName = 'my-app'    // otherwise this is 'server'
    version = project.version + '.' + System.currentTimeMillis();   // I simply prefer to append a ms timestamp in case I've run multiple builds

    // Finally, include the files from the client:
    from('../client/dist') {
        include '**/*'
    }
}
```

The above will place the Angular front-end in the root of your web-app (e.g. / will go to your angular files). This  
may not be what you want.  If you want, you can update the `--base-href` argument in package.json to some 
sub-directory (say something like `--base-href=\"/my-app/app\""`). Then you probably need to change you 
URL Mappings so that '/' is redirected to '/app/index.html' or something.

Now running 'gradle assemble' or 'gradle server:assemble' in the root will 
build a complete war file that can be deployed as is. Note that there are various other
ways to accomplish this task, but the method above works ok for me.  It is even quite 
feasible to use a 