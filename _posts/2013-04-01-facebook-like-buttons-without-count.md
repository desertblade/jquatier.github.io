---
layout: post
title:  "Facebook LIKE buttons without the count"
date:   2013-04-01 00:22:00 -0700
summary: "Facebook seems to really want you to show the number of LIKEs whenever you use one of their buttons on your site. Here's a good hack around that..."
---
![Facebook Like Button](/content/images/facebook-like-button.jpg)

I'm not a huge fan of putting Facebook LIKE buttons on everything, but there's some cases where it makes sense. It's also advantageous to use the code that Facebook provides for buttons rather than building your own, because it handles all the different states a user could be in (logged in, logged out, already liked, etc.). It allows users to like your content on Facebook without leaving your website, so that's kind of nice as well.

What is NOT nice is that if you use their button, they force you to show the number of likes. There isn't a flag you can pass in (like Twitter has) to not show the count next to the button. Here's a quick hack to hide the count by setting the width of the IFrame accordingly.

{% highlight css %}
/* CSS Hack for hiding the LIKE count */
div.fb-like span {
  display:block;
  width:48px !important;  
}

div.fb-like iframe {
  width:48px !important;
}

div.fb-like iframe.fb_iframe_widget_lift {
  width:450px !important;
}
{% endhighlight %}

It's important to note that the 3rd rule is to set the width correctly when the comment model window appears (Facebook allows you to comment on what you just liked). Without this, only the first 48 pixels of that modal would be visible.

This will probably break at some point if Facebook decides to change their API like they often do, but this seems to work right now.
