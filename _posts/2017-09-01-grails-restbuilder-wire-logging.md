---
layout: post
title: Grails RestBuilder and Wire Logging
---

Grails 3.x has a nice plugin - RestBuilder, which had been part of the
 Grails Data project so it has some official support.

One of the things that I've struggled with is to log the HTTP traffic 
when I'm in development / functional testing so that it can be debugged. Under the hood, 
the RestBuilder uses Spring's RestTemplate and there are plenty of StackOverflow answers 
documenting a generic solution, which is to build an interceptor, 
then setup and use a custom RestTemplate. However there is another solution.

By default, the Spring RestTemplate is by default using the underlying 
JDK HTTP libraries, which are ok but a bit basic. One of the issues 
is that any logging must be configured via Java Utility Logging, 
and that may not be very well documented. In my case I decided to switch to the 
Apache HttpClient library, which the RestTemplate supports and to configure it for logging.

First, you need to add the HttpClient dependency to build.gradle:

```groovy
dependencies {
    //... other dependencies here
    // Apache HTTP Client:
    compile 'org.apache.httpcomponents:httpclient:4.5.3'
}
```

At this point you can edit your logback.groovy file to contain:

```groovy
if (Environment.DEVELOPMENT == Environment.current) {
    def appenders = ['STDOUT', 'FULL_STACKTRACE'];
    // ... other logger setup
    logger('org.apache.http.headers', DEBUG, appenders, false);
    logger('org.apache.http.wire', DEBUG, appenders, false);
}
```

This will set both header and wire logging for the Apache HttpClient, 
so you will see a lot of details in the output once it's used.

There are several ways to configure a RestBuilder with a 
customized RestTemplate to use the HttpClient libraries. For example 
you could use the PreConstruct annotation on a service to 
setup a private variable. In my case I was using the RestBuilder in 
a few difference services and for now I wanted to just log all of 
them. In this case I deded the conf/spring/resources.groovy file 
to create a bean that can be DI'd into my services.

```groovy
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory
import grails.plugins.rest.client.RestBuilder
import org.springframework.web.client.RestTemplate

beans = {

    restBuilder(RestBuilder) { bean -&gt;
        bean.scope = 'prototype'
        restTemplate = new RestTemplate(new HttpComponentsClientHttpRequestFactory())
    }

}
```

Finally, in my services I just declare a standard Grails name based dependency like:
```groovy
class MyRestApiService {
 
    RestBuilder restBuilder

}
```


### Update
In 2 other cases, I ended up using direct calls to the Jetty HTTP Client,
because it has a bit more of a mid-level access, allowing things such 
as async file download without requiring configuring 24 different subclasses.