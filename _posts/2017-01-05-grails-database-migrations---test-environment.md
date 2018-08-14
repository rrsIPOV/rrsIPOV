---
layout: post
title: Grails Database Migrations - Test Environment
---

I've been using Grails 3.x for a while now on various small projects, 
and I really like it overall. The main problem (in general) is that the development 
is moving so fast that documentation of best practices is lacking. One place this 
bit me is the database migrations plugin. I had some minimal experience with an older version of this plugin several 
years; at the time I didn't have time to dive too far in. This time, I wanted to try to get my setup cleaner, 
particularly in regards to data for functional tests / development showcase.

The simple, and usually only answer given for test data is to set it up in the application 
Bootstap class using the Grails Environment to decide where to apply these too as Gorm objects. This works, 
although it's always struck me as a bit limited in terms of modularity - certainly 
you can break stuff up into different methods or even separate classes, but in the end you had a bunch of data in your code.

For my current project it got a bit more annoying as I wanted to import some 
lookup data (tables) that were in use in the current app (non-Grails) so that the data would be there 
during initial setup, and then add some additional data for the integration testing phase. I could 
have done this in the Bootstrap class, but it would be a lot of code, and anyway 
as I said something about it seemed off. So I stated looking at the current version of the 
<a href="https://github.com/grails-plugins/grails-database-migration">Database Migrations plugin</a>.

The DBM plugin is a pretty neat plugin, although it does send you over to 
the <a href="http://www.liquibase.org/">LiquidBase </a> docs a bit too soon in my opinion. After all, 
LiquidBase is XML based, but the Groovy scripting offered by the DBM plugin 
seems like a superior fit, if you can match the DSL up to Liquidbase's XML.

After setting up my common bootstrap data I started looking at how to 
include my testing data via the same mechanism. I came across the concept 
of Liquidabase Contexts, which at first seemed promising. However if the 
rules for contexts basically seem to boil down to:

1. If a context is supplied during the run then ONLY that context is used.
1. If no context is specified then ALL context's are used.

What I really wanted was a "default" run-time context. It turns out there is a 
default author time context, but the run-time rules still apply. What I couldn't seem to do easily was: 

1. For production, ONLY apply production data
1. During the integration testing apply BOTH production data AND testing data.

Now, as I said, the documentation and "best practice" guides are a bit thin, 
so I could have missed something. It may well be that I should set the contexts as part of 
the DBM plugin's configuration using environments and get it to work. However I came up with an alternative solution described below.

Start with the standard migration file: `changelog.groovy` - this fill will contain 
your production data. Now create a 2nd file called `changelog-dev.groovy` and in it do something like:

```groovy
databaseChangeLog = {
	// Include the production file:
	include file 'changelog.groovy'
	
	//	Now can include some custom files.
	include file 'dev-init/test-config.groovy'
	include file 'dev-init/sample-data-v2.groovy'
}
```

Note that the files referenced are relative to the location of the change log.


Now in the application.yml configuration you could do something like:

```yml
grails.plugin.databasemigration:
    changelogLocation: 'grails-app/migrations'
    changelogFileName:'changelog.groovy'
    autoMigrateScripts: ['RunApp', 'TestApp']
#....additional configuration for other stuff as needed...
environments:
	development:
		dataSource:
			dbCreate: none
		grails.plugin.databasemigration:
			dropOnStart: true
			updateOnStart: true
			updateOnStartFileName: 'changelog-dev.groovy'
	test:
		dataSource:
			dbCreate: none
		grails.plugin.databasemigration:
			dropOnStart: true
			updateOnStart: true
			updateOnStartFileName: 'changelog-dev.groovy'
```		

This configuration should apply the changelog-dev.groovy during local development runs as 
well as testing. You can then use the main changelog.groovy to 
generate SQL to update your production DB using the plugin's command line without having to specify any additional parameters.

Finally, I'll note that if any of the include'd files don't exist (e.g. you get the filename misspelled) then 
you will get an error message which does not say which file is the problem. Creating 1 large changelog,groovy might avoid this, 
but if you organize files by milestone / release then you need to watch out for this.


## Update

Currently I have given up on the DBM plugin for my projects and have a set of scripts called by the
application Bootstrap.  I plan to post about my reasons when I have time.

