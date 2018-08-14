---
layout: post
title: Immaculate Configuration
---
This is more of a rant than anything else. Today I was once 
again fighting to try to externalize my Java web-app's initial configuration.

I mean, the while point of a WAR would seem to be to enable a modular deployment, 
but where do you get your configuration values? Sure, you can pull JNDI environment configuration, 
except that other than a variety of container specific hooks, 
the core JNDI stuff is one global namespace. Also setting system properties 
in the startup script is really annoying (not to mention not that scalable).

Here's my problem: I need to be able to configure the same web-app to be available 
on different URLs, and each such context to have its own configuration. Some of 
these contexts are being automatically built from source control by Jenkins, 
while others are being manually deployed (based on Q.A.), 
all are used for various types of internal testing.

My solution for the Jenkins deployed apps, 
is to build the war, and to include the needed context XML configuration 
(Tomcat specific) in my source repository. When the WAR is done, 
I have a Groovy script that calls the Tomcat Manager deployment 
URLs with the WAR + context.xml for each of my contexts, 
there by deploying multiple times, with different contexts (e.g. /myapp-foo and /myapp-bar).

I don't love my solution, but its pretty workable. The 
one big problem I've run into, is that occasionally one of 
the manually deployed variants of the web-app will fail to 
redeploy when updated. The error logs claim that the MySQL JDBC driver 
can not be found, which is strange, since the most of the other versions of the 
web app are also using MySQL and they are all still running; note that 
I've dropped the mysql driver jar into CATALINA_HOME/lib so it should be 
globally available. The only thing that I can think of is that there is 
some form of classloading issue whereby the driver pooling is preventing 
the redeployed web-app from reconnecting. This is quite annoying 
as the only solution I've found so far is to stop Tomcat, 
delete the contents from ./work, and restart.

Oh, yeah my configuration is: latest Java 6 JDK, Tomcat 7, MySQL 
(plus some other DBs to test against) on Mac OS). I started to look at JBoss 7.1, 
but the current documentation seems to lack any real "getting started" material to walk one through 
actually deploying stuff, and I have enough on my hands that I don't want 
to spend several hours of trial and error to find out at this point.

The main point is : the latest web fragment configuration for components is great, 
but could we please have some standardized way to set basic bootstrap configuration values. I don't 
care what the mechanism is. I don't care what the UI is. I just want to be able to put 
some references to configuration parameters inside my web-app and, 
once the container is configured with them, to auto-magically have them be populated.

The Tomcat context file XML syntax wouldn't be bad if one could create a context that 
persisted across redeployments without putting it into the WAR, 
and without hardcoding it into the root server.xml file, 
basically way to say "if a web-app named X is present then make these 
variables available to it". The problem with Tomcat is I have to either: 
a) hard code this into server.xml, which also means the web-app can NOT be redeployed, or 
b) continually deploy the context side-by-side with the WAR everytime I want to update the WAR.


## Update 

So, 6 years later I'm migrating this post to Git Hub pages.  Looking back I think it's still somewhat 
interesting.  Obviously, it looks like JVM services are already fragmented, and with Oracle dropping JEE 
are likely to do so further.  The issue that I see is that while there is some for a handful of solutions, 
one of the issues that started fragmentation is the lack of non-run time specifications/standards.  Obviously, 
this wasn't all Sun (later Oracle's fault) as the opening up of the standards process was a 
differentiator from say dot-Net, but lead to some vendor sandbagging due to inability to pick 
one or two options.  Sometimes something sub-optimal is better than arguing continually 
about the most optimal solution without ever getting one.