---
layout: post
title:  "Spring MVC Mapped Interceptors"
date:   2013-07-15 22:38:00 -0700
summary: "Spring MVC 3.2 adds the ability to exclude certain request interceptors from running based off the incoming request path. Here's how it works..."
redirect_from: "/posts/2013/07/15/spring-mvc-mapped-interceptors/"
---
Interceptors in Spring MVC are a great way to add logic that needs to execute for every request to your servlet. The nice thing is that you can have logic that executes BEFORE and/or AFTER the request is handled by your controller, which is done by overriding the preHandle and postHandle methods in the [HandlerInterceptor](http://static.springsource.org/spring/docs/3.2.x/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html) interface that Spring provides.

The one problem in the past with using interceptors was that there wasn't a good way to exclude them from running for certain requests. For example, you might have a specific service that doesn't need to run that additional logic in your custom interceptor so it's just adding unnecessary execution time.

Enter Spring 3.2 and we now have a great way of selectively including or excluding interceptors from running based off a mapping system. It's called Mapped Interceptors, and it uses Ant path matching to see if a request path matches the included or excluded paths. Check out the Spring [documentation](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-config-interceptors) for more detail. Here's an example:

XML configuration:

{% highlight xml %}
<mvc:interceptors>
  <mvc:interceptor>
      <mvc:mapping path="/**"/>
      <exclude-mapping path="/foo/**"/>
      <bean class="com.jacobquatier.MyCustomInterceptor" />
  </mvc:interceptor>
</mvc:interceptors>
{% endhighlight %}

Java configuration:

{% highlight java %}
@Configuration
@EnableWebMvc
public class MyWebConfig extends WebMvcConfigurerAdapter
{
  @Override
  public void addInterceptors(InterceptorRegistry registry)
  {
    registry.addInterceptor(new MyCustomInterceptor())
            .addPathPatterns("/**")
            .excludePathPatterns("/foo/**");
  }
}
{% endhighlight %}

Anyway, because the path matcher uses Ant paths, it's pretty easy to include/exclude anything you want.

One thing you might want to watch out for is that Spring automatically detects any beans of type MappedInterceptor and registers them in your servlet. So, if you have multiple servlets with some shared configuration/beans, any MappedInterceptor bean that's sitting in the context will get picked up and be put in the list of interceptors that get run. This behavior can be overridden by subclassing the handler mapping and changing the detectMappedInterceptors method. See [AbstractHandlerMapping](http://static.springsource.org/spring/docs/3.2.x/javadoc-api/org/springframework/web/servlet/handler/AbstractHandlerMapping.html) for more information.
