---
layout: post
title: spring struts2 mybatis integration
date: 2016-07-10 22:19
author: 2freesky
comments: true
categories: [mybatis, spring, struts]
---
<ol>
	<li>
<p class="western"><strong>Configure the spring listener and struts2 filter in web.xml</strong></p>
</li>
</ol>
<blockquote>
<p class="western"><span style="color:#008080;"> <span style="font-family:'Courier New', monospace;"><span style="font-size:small;">&lt;</span><span style="color:#3f7f7f;">context-param</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">param-name</span><span style="color:#008080;">&gt;</span><span style="color:#000000;">contextConfigLocation</span><span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">param-name</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><em><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;!-- the applicationContext.xml should be placed under WEN-INF by </span></span></span><span style="font-size:small;font-family:'Courier New', monospace;color:#008080;line-height:1.7;">default. However we can use &lt;context-param&gt; to declare it is</span><span style="color:#008080;"> <span style="font-family:'Courier New', monospace;"><span style="font-size:small;">in src/main/resource with classpath:applicationContext.xml--&gt;</span></span></span></em></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">param-value</span><span style="color:#008080;">&gt;</span><span style="color:#000000;">classpath:applicationContext.xml</span><span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">param-value</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">context-param</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">listener</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><em><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#3f5fbf;">&lt;!-- Following listener is used to load the spring context file. --&gt;</span></span></span></em></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">listener-class</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#000000;">org.springframework.web.context.ContextLoaderListener</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">listener-class</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">listener</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">filter</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">filter-name</span><span style="color:#008080;">&gt;</span><span style="color:#000000;">struts2</span><span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">filter-name</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><em><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;!-- struts2 filter class path may be differ in different version--&gt;</span></span></span></em></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">filter-class</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#000000;">org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">filter-class</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">filter</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">filter-mapping</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">filter-name</span><span style="color:#008080;">&gt;</span><span style="color:#000000;">struts2</span><span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">filter-name</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">url-pattern</span><span style="color:#008080;">&gt;</span><span style="color:#000000;">/*</span><span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">url-pattern</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="color:#008080;"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;">&lt;/</span></span></span><span style="color:#3f7f7f;"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;">filter-mapping</span></span></span><span style="color:#008080;"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;">&gt;</span></span></span></p>
</blockquote>
<ol start="2">
	<li>
<p class="western"><strong>define struts2 action bean, data source and mybatis sql session in applicationContext.xml</strong></p>
</li>
</ol>
<blockquote>
<p class="western"><span style="color:#008080;"> <span style="font-family:'Courier New', monospace;"><span style="font-size:small;">&lt;</span><span style="color:#3f7f7f;">bean</span> <span style="color:#7f007f;">id</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"helloworld"</i></span> <span style="color:#7f007f;">class</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"example.my.integration.struts.HelloWorld"</i></span><span style="color:#008080;">&gt;</span> </span></span></p>
<p class="western" align="LEFT"><em><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;!-- add sqlSession in action class in this case, because we will q</span></span></span><span style="font-size:small;font-family:'Courier New', monospace;color:#008080;line-height:1.7;">uery db from action class directly with the help of sqlSession</span><span style="color:#008080;"> <span style="font-family:'Courier New', monospace;"><span style="font-size:small;">--&gt;</span></span></span></em></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">property</span> <span style="color:#7f007f;">name</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"sqlSession"</i></span> <span style="color:#7f007f;">ref</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"sqlSession"</i></span> <span style="color:#008080;">/&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">bean</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">bean</span> <span style="color:#7f007f;">id</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"dataSource"</i></span> </span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#7f007f;">class</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"org.springframework.jndi.JndiObjectFactoryBean"</i></span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><em><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;!-- Using JndiObjectFactoryBean such that we can define db resource </span></span></span><span style="color:#008080;"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;">in tomcat context.xml without configuring in project. Meanwhile </span></span></span><span style="color:#008080;"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;">we don't need to use any code to do lookup jndi or connect db, </span></span></span><span style="color:#008080;"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;">just use mybatis sqlSession api to operate db--&gt;</span></span></span></em></p>
<p class="western" align="LEFT"><span style="color:#000000;"> <span style="font-family:'Courier New', monospace;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">property</span> <span style="color:#7f007f;">name</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"jndiName"</i></span> <span style="color:#7f007f;">value</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"java:/comp/env/jdbc/example" </i></span><span style="color:#008080;">/&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="color:#000000;"> <span style="font-family:'Courier New', monospace;"> <span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">bean</span><span style="color:#008080;">&gt;</span> </span></span></p>
<p class="western" align="LEFT"><em><span style="color:#000000;"> <span style="font-family:'Courier New', monospace;"><span style="font-size:small;">&lt;!-- integrate spring with mybatis by using following two beans --&gt;</span></span></span></em></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">bean</span> <span style="color:#7f007f;">id</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"sqlSessionFactory"</i></span> </span></span></p>
<p class="western" align="LEFT"><span style="color:#7f007f;"> <span style="font-family:'Courier New', monospace;"><span style="font-size:small;">class</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"org.mybatis.spring.SqlSessionFactoryBean"</i></span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">property</span> <span style="color:#7f007f;">name</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"configLocation"</i></span> <span style="color:#7f007f;">value</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"classpath:mybatis-config.xml"</i></span> <span style="color:#008080;">/&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"> <span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">property</span> <span style="color:#7f007f;">name</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"dataSource"</i></span> <span style="color:#7f007f;">ref</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"dataSource"</i></span> <span style="color:#008080;">/&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;/</span><span style="color:#3f7f7f;">bean</span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">bean</span> <span style="color:#7f007f;">id</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"sqlSession"</i></span> <span style="color:#7f007f;">class</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"org.mybatis.spring.SqlSessionTemplate"</i></span><span style="color:#008080;">&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;"><span style="color:#008080;">&lt;</span><span style="color:#3f7f7f;">constructor-arg</span> <span style="color:#7f007f;">index</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"0"</i></span> <span style="color:#7f007f;">ref</span><span style="color:#000000;">=</span><span style="color:#2a00ff;"><i>"sqlSessionFactory"</i></span> <span style="color:#008080;">/&gt;</span></span></span></p>
<p class="western" align="LEFT"><span style="color:#008080;"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;">&lt;/</span></span></span><span style="color:#3f7f7f;"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;">bean</span></span></span><span style="color:#008080;"><span style="font-family:'Courier New', monospace;"><span style="font-size:small;">&gt;</span></span></span></p>
</blockquote>
<ol start="3">
	<li>
<p class="western"><strong>In struts.xml, the value of action class should be bean id which declared in applicationContext.xml</strong></p>
</li>
</ol>
<ol start="4">
	<li>
<p class="western"><strong>add struts2-spring-plugin, mybatis-spring jars in pom.xml</strong></p>
</li>
</ol>