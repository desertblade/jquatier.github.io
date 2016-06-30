---
layout: post
title:  "Freemarker and Exception Logging"
date:   2013-05-08 20:54:00 -0700
summary: "Freemarker pushes a lot of information to the logs. Here's how to specify the logging framework it uses, or disable logging entirely..."
redirect_from: "/posts/2013/05/08/freemarker-exception-logging/"
---
Freemarker is a pretty nice templating language for Java-based servlets, especially if you are looking for an alternative to JSP pages. One problem with it, or "feature" I should say, is that it puts a pretty large stack trace in your logs whenever it encounters an error rendering a template. In my travels, I've seen a single missing null-check fill up server logs rather quickly for pages that are hit often (especially if there is looping going on in the template). I decided it wasn't really helping me at all to have these in the logs, so I wanted to turn them off, at least on the live site.

Turns out all I needed to do was extend the FreemarkerConfigurer class that Spring MVC uses to instantiate the Freemarker view resolver. This is important because you need to set up the logger library before any of the Freemarker classes load up. Thankfully, there's an afterPropertiesSet() method that you can override for this purpose. Here's an example:

{% highlight java %}
public class MyAwesomeFreemarkerConfigurer extends FreeMarkerConfigurer
{
  private boolean loggingEnabled = true;

  @Override
  public void afterPropertiesSet() throws IOException, TemplateException
  {
    // disable freemarker logging if the bean is configured as such
    if(!loggingEnabled)
    {
      resetFreemarkerLogging();
    }
    super.afterPropertiesSet();
  }

  public void resetFreemarkerLogging()
  {
    try {
      freemarker.log.Logger.selectLoggerLibrary(Logger.LIBRARY_NONE);
    } catch (ClassNotFoundException e) {
      System.out.println("Failed to disable Freemarker logging =(");
    }
  }

  public void setLoggingEnabled(boolean loggingEnabled)
  {
    this.loggingEnabled = loggingEnabled;
  }
}
{% endhighlight %}

That class will disable all Freemarker logging (Logger.LIBRARY_NONE). You'll notice I added a property which allows me to enable/disable logging via bean configuration. This means I can turn logging off in the live site, but leave it on for test and development. You could also use this approach to set the logger library to whatever logging framework you use. By default, Freemarker will use the first logging library it finds, per the [documentation](http://freemarker.sourceforge.net/docs/pgui_misc_logging.html).

Anyway, depending on your situation you might not want logging turned off completely. But, in my case it was far too noisy to provide any value. The smallest template error made the logs a nightmare since the pages get hit so often. Template errors are easy to overlook (especially if they only happen when dynamic data is in a certain state), and I need my server logs readable at all times. Also, if a problem arises with a template, I can test it in an environment that has Freemarker logging turned on and I'll see the stack trace. Hopefully by keeping logging enabled in test and dev, I get the best of both worlds. We'll see! =)
