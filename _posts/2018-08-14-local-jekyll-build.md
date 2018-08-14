---
layout: post
title: A Local Jekyll Build for an Offline Site Folder
---

I've moved my "blog" to GitHub + Jekyll since I mostly post code snippets, 
rants and thoughts. Blogger wasn't cutting it, and I didn't want pay. Plus this 
seems cooler.  

Before I made this switch, I tested the waters with Jekyll with 
some documentation that I needed anyway.  I'd previously tried hosting it as a
Sharepoint "blog" (nuked during a migration), and as a Confluence Wiki - great 
but turns out inaccessible to the people who needed it 
(the PDF export in Confluence is really cool... just needs a 'tree' mode).  
In the end I basically gave up for a while, but something needed to be done.

I had done various markdown documents before, and felt ok using the syntax, but 
it wasn't until the idea of Jekyll (or another static tool) caught my attention.  I 
ended up going w/ Jekyll mainly due to GitHub support since I wasn't "sold" on anything in particular. 

However, after getting a good start, I realized two things:

1. The _site that Jekyll builds still needs to be hosted (due to asset paths).
2. The people who needed my documentation have **no** online tools, e.g. they 
have a sharepoint, but no where to deploy a folder full of static web pages + assets.

I went back to the drawing board for a bit, and did a bunch of reading.  There isn't 
an easy option since the built site tends to have at least some sub-directories while 
still needing to reference files (images, css, js) in asset folders.

After a while I found someone posting a reply to a similar issue with - use wget to 
download the site for offline use.

So I worked out a very basic windows .bat file to do that.  It's not pretty but it works - 
assuming you have a version of wget for windows (or can convert the commands below to shell).

```bash
rd /q /s "_static-site" 2>nul
mkdir _static-site
start jekyll serve
timeout 5
wget --convert-links --limit-rate=250k --directory-prefix=_static-site -nH -l 10 -k -p -r  http://127.0.0.1:4000/
echo "Complete"
```

1. Remove the contents of the `_static-site` folder if it exists, 
   if it doesn't exist "fail" silently and continue.
1. Remove the `_static-site` folder.  Since I'm on windows this is non-recursive.
1. Run Jekyll to host the site locally.
1. Wait 5 seconds to allow the site to start up - this could be changed to a keypress or 
   to look for some other signal, but for me timeout is simple and currently works.
1. Use wget to build a truely static, offline version of the site into the `_static-site` folder.

Now, as long as the user can open the generated index.html in a browser that 
loads local CSS/JS then everything will work.  The `_static-site` folder can be
copied to a shared network drive and I can keep the source (markdown) under version control.