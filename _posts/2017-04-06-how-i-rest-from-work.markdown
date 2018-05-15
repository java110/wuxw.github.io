---
layout: post
title: Spring Boot 中文乱码解决
date: 2018-05-12 13:32:20 +0300
description: 使用SpringBoot开发，对外开发接口供调用，传入参数中有中文，出现中文乱码，查了好多资料。 # Add post description (optional)
img: i-rest.jpg # Add image post (optional)
tags: [spring boot, 乱码]
---
使用SpringBoot开发，对外开发接口供调用，传入参数中有中文，出现中文乱码，查了好多资料，总结解决方法如下：

# 第一步,约定传参编码格式
不管是使用httpclient，还是okhttp，都要设置传参的编码，为了统一，这里全部设置为utf-8

# 第二步，修改application.properties文件

增加如下配置：

{% highlight java %}

spring.http.encoding.force=true
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
server.tomcat.uri-encoding=UTF-8

{% endhighlight %}

此时拦截器中返回的中文已经不乱码了，但是controller中返回的数据依旧乱码。

# 第三步，修改controller的@RequestMapping

修改如下：

{% highlight java %}
produces="text/plain;charset=UTF-8"
{% endhighlight %}

这种方法的弊端是限定了数据类型，继续查找资料，在stackoverflow上发现解决办法，是在配置类中增加如下代码：

{% highlight java %}
@Configuration
public class CustomMVCConfiguration extends WebMvcConfigurerAdapter {

    @Bean
    public HttpMessageConverter<String> responseBodyConverter() {
        StringHttpMessageConverter converter = new StringHttpMessageConverter(
                Charset.forName("UTF-8"));
        return converter;
    }

    @Override
    public void configureMessageConverters(
            List<HttpMessageConverter<?>> converters) {
        super.configureMessageConverters(converters);
        converters.add(responseBodyConverter());
    }

    @Override
    public void configureContentNegotiation(
            ContentNegotiationConfigurer configurer) {
        configurer.favorPathExtension(false);
    }
}
{% endhighlight %}

便可以解决SpringBoot的中文乱码问题了。
