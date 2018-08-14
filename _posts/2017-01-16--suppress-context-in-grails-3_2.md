---
layout: post
title: Suppress context in a Grails 3.2 Web-App
---

I recently rebuilt an older and very simple PHP "web service". As is typically, 
what started life as a single, simple API had grown and we had committed to adding yet 
another set of functionality. If the existing code had been written around an 
existing (and supported) framework, I would have left it intact and extended it. However it 
was a bundle of homegrown code. So I decided to replace it with a Grails app 
in order to benefit from, well just about everything.

However as deployment time got near, I realized a had a bit of a problem. The production 
end-point URLs looked like - https://myapi.hostname.com/* and alas had been hard coded 
into a 3rd party web-app and was not going to be cost effective have to changed.

In production the setup used an Apache2 front-end and mod_proxy_ajp with Tomcat 
hosting the actual app. The Apache front-end actually is a bit of a mess, but that's not immediately 
relevant other than as an additional reason that I couldn't change the end points. What is relevant 
is that it's a wildcard'd set of virtual hosts 
(e.g. as an example: web.hostname.com, myapi.hostname.com, files.hostname.com). The longer term solution 
would either be multiple virtual machines, or using catatlina_base and 
having multiple AJP ports. However, neither was appealing as I 
really needed something faster to setup and with less server maintenance.

I had setup a simple configuration for Apache such as:
```
ProxyPass                   /      ajp://localhost:8009/myapi/
ProxyPassReverse            /      ajp://localhost:8009/myapi/
ProxyPassReverseCookiePath /myapi  /
```
 
Internally, the API portion was working ok, but one of the big 
improvements had been to add a simple admin control panel. All the 
links were like /myapi/admin/* instead of /admin/*. Now, I could have
 used Apache to rewrite the links to remove the /myapi pre-fix, but I 
wanted to preferable find a solution to the issue at the source.

A quick google search turned up 3 configuration variables:

1. `grails.assets.url`
1. `grails.serverURL`
1. `grails.app.context`

The first can be set to something like `https://myapi.hostname.com/assets/` in order to correct 
the assets-pipeline URLs. However, setting the `grails.app.context` was not having 
any effect that I could see on my g.createLink(...) calls - I didn't want to do a search 
and replace on my entire codebase... if it came to that I'd have gone back the mod_rewrite. 
However I had a little time, so I started googling further and found the `org.grails.plugins.web.taglib.ApplicationTagLib` class, 
which I found actually delegates most of the work to an instance of `grails.web.mapping.LinkGenerator`. There is a 
DefaultLinkGenerator that I was seeing used, and it didn't seem to look at any config 
properties, but it did look for an (undocumented) includeContext (e.g. `LinkGenerator.ATTRIBUTE_INCLUDE_CONTEXT`) map value.

So my solution, which is working so far is to extend the ApplicationTagLib and override 
it's `doCreateLink()` helper method to insert the `attrs.includeContext = false` which is not by default present:

```groovy
package my.api

import grails.web.mapping.LinkGenerator
import groovy.transform.CompileStatic
import org.grails.plugins.web.taglib.ApplicationTagLib

class LinksTagLib extends ApplicationTagLib {

    @Override
    @CompileStatic
    protected String doCreateLink(Map attrs) {
        if (!attrs.containsKey(LinkGenerator.ATTRIBUTE_INCLUDE_CONTEXT)) {
            attrs[LinkGenerator.ATTRIBUTE_INCLUDE_CONTEXT] = false
        }
        return super.doCreateLink(attrs)
    }

}
```


It would be great if there was some supported option in the `application.yml` to set some 
flags controlling the link generation. However, I don't have time to try creating a 
pull request. I may add a ticket, but then again the Grails team is pretty busy 
with their existing roadmap and I really don't see a point in 
reporting this without a pull request of some sort.

I am pretty sure that I could have dug around into the default 
Grails Spring Beans configuration and replaced the appriate beans with a 
custom subclass, but as I didn't see any documentation on where to find this, 
or what beans / factories may be involved (a quick check of the code looked like several might be players), 
I found that at the moment the tag library was a faster approach. 

Maybe the Apache mod_rewrite option would be better - time may tell and I may have to switch over to it. However, 
I really prefer that in this mode the main config options are 
still all in the Grails app and I don't have to have important 
parts out in the apache server config.

Feedback is very welcome. 


## Update

After writing this post, I eventually upgraded the server and removed Apache. Now it's 
Tomcat running directly using the native connector (TC Native). The taglib based approach is working fine
as of Grails 3.3.x