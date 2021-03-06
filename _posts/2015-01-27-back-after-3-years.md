---
layout: post
title: Back after almost 3 years.
comments: False
---

# {{ page.title }}

_27 Jan 2015_

First post in almost 3 years. I've been lazy.

I've been fiddling with a small prototype for a port of an old and problematic [WPF XAML Browser Application](https://msdn.microsoft.com/en-us/library/aa970060(v=vs.90).aspx) project to an ASP.NET MVC 4 site.

That project has a couple of "needs", namely user-defined access control (the customer's administrator is free to grant and revoke permissions to and from user groups and to add and remove users from those groups) and showing reports made with [SAP's Crystal Reports](http://www.sap.com/solution/sme/software/analytics/crystal-reports/index.html) inside its viewer.

It has been a while since I last touched an ASP.NET MVC site (since version 3) so I was wondering if it has gotten any better.

Well, at least somethings stay as hard as ever, namely "globalization".

As for the Crystal Report's viewer, I've ended up just returning a rendered PDF file stream. It has all the functionality my users expect (zoom, search and pagination).

So, my next post will be about the [Crystal Reports prototype implementation]({{ site.baseurl }}/2015/01/28/replacing_the_cr_viewer.html) and the one after that about the dreaded globalization.
