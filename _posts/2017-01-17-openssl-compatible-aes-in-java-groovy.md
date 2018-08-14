---
layout: post
title: OpenSSL Compatible AES in Java/Groovy
---

I have 2 web "service" projects (one predating the other) that need to 
take a text parameter that has been encoded using AES with a pre-shared key (aka "password").

At first this seemed like it was going to be pretty straight 
forward: "I'll just make sure I have the unlimited strength security policy files and I'm sure there are some implementations" I thought. Little did I know.

As an aside, let me say that there really ought to be some mechanism to pull 
previously installed versions of the Java unlimited strength security policy files into upgraded 
installs. Especially with apt-get it seems like there should be some 
option. If anyone is aware of anything please let me know.  **Update** This finally seems to have been addressed on Ubuntu.

Search high and search low, and there is seemingly nothing that will work with 
an OpenSSL encrypted text in that Java world. I finally found not-yet-commons-ssl, which seems like it 
is really not-**ever**-common. I had used this in my initial project, back in 2015, 
but it appears that I came across the last gasp of breath for a one man library, 
no new releases nor seemingly any code changes since Spring of 2015 seems to indicate a now dead project.
As it has several dependencies on older libraries this is worrying and I'd like to remove it.

During my latest attempt, I decided to again check google to see if I could locate 
any libraries or sample code. I finally found 
<http://stackoverflow.com/questions/32508961/java-equivalent-of-an-openssl-aes-cbc-encryption> 
which has a nicely implemented Encoding method, but no decoding. I also was lucky enough to have access to a working C# source.

The basic issue is that there are multiple ways to create a key and an initialization vector for 
the Cipher that will be used, and OpenSSL uses something rather quirky compared 
to what Java supports in that they derive everything from the password using a certain algorithm.

The end point of my ramblings is that by comparing the complete C# with the 
partial (but cleanly written) Java I was able to work through with some trial and 
error to a working implementation (in Groovy, but converting to Java 
should be fairly trivial). I've copied a version to <https://gist.github.com/rrsdev/4d0f6be7c58173c16e9edf9f97c7d7f2>

If anyone is aware of a maintained library that supports OpenSSL compatible, 
password based AES Cipher/encryption please let me know. My time on searching Stack Overflow 
has made me fairly skeptical that such a beast exists. 
