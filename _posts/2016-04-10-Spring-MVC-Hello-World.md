---
layout : post
title : Spring MVC Hello World
comments: true
categories : Spring
---

  This is a hello world example with spring boot 2.

### Flow

![_config.yml]({{ site.baseurl }}/images/spring_mvc.png)

### Example

  This is a very simple example, when user open it in the browser, `index.jsp` will be shown and user can click some buttons to fetch
  data from controller.

#### Maven POM

```xml

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.2</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<dependency>
    		<groupId>org.apache.tomcat.embed</groupId>
    		<artifactId>tomcat-embed-jasper</artifactId>
    		<scope>provided</scope>
		</dependency>
```

**Note: tomcat-embed-jasper is used to compile jsp files in embeded tomcat.
        If it is ommitted, then jsp cannot be shown when you access it if you use embed tomcat.
        After adding this dependency, you have to restart your spring boot app manually.
        **

#### jsp file

  `index.jsp` is placed under `src/main/webapp`.
    
```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Hello SpringMVC</title>
    <script src="jquery-3.3.1.min.js"></script>
</head>
<body>
<input type="button" value="post json" onclick="getUser()" />
<input type="button" value="get user 2" onclick="getUser2()" />
<input type="button" value="get user 3" onclick="getUser3()" />
<script>
function getUser() {
	

	   var search = {
	      "userId" : 2
	   }
	   
	   jQuery.ajax({
	      type: "POST",
	      contentType : 'application/json; charset=utf-8',
	      dataType : 'json',
	      url: "/sbw/user/getUser",
	      data: JSON.stringify(search), 
	      success :function(result) {console.log(result);
	       alert(result.userName);
	     }
	  });
}

function getUser2() {
	

	   var search = {
	      "userId" : 2
	   }
	   //both post and get are ok
	   jQuery.ajax({
	      type: "GET",	  	     
	      url: "/sbw/user/getUser2",
	      data: search, 
	      success :function(result) {console.log(result);
	       alert(result.userName);
	     }
	  });
}

function getUser3() {
	

	   var search = {
	      "userId" : 2
	   }
	   
	 //both post and get are ok
	   jQuery.ajax({
	      type: "POST",	  	     
	      url: "/sbw/user/getUser3",
	      data: search, 
	      success :function(result) {console.log(result);
	       alert(result.userName);
	     }
	  });
}
</script>
</body>
</html>
```

When you open website in browser by url 'http://localhost:8080/sbw`, the `index.jsp` or `index.html` will be lookuped under the root path or the path defined in `spring.mvc.view.prefix`. Alternatively, you can access `index.jsp` by 'http://localhost:8080/sbw`.

##### WEB-INF

  It's a good practice to place resources like jsp under the `WEB-INF ` folder such that user cannot access them directly from the
  browser by typing the url.
  
  The Servlet 2.4 specification says this about `WEB-INF`:
  
  > A special directory exists within the application hierarchy named WEB-INF. 
  > This directory contains all things related to the application that aren’t in the document root of the application. 
  > The WEB-INF node is not part of the public document tree of the application. 
  > No file contained in the WEB-INF directory may be served directly to a client by the container. 
  > However, the contents of the WEB-INF directory are visible to servlet code 
  > using the getResource and getResourceAsStream method calls on the ServletContext, 
  > and may be exposed using the RequestDispatcher calls.
  
  The change in Servlet 3.0 & 3.1 (JSR 340) allows serving static resources and JSPs from within a JAR stored in `WEB-INF/lib`. 
  
  > Except for static resources and JSPs packaged in the META- INF/resources of a JAR file 
  > that resides in the WEB-INF/lib directory, no other files contained in the WEB-INF directory
  > may be served directly to a client by the container. 

  Thus from Servlet 3.0, static files can be served from: 
  **WAR file > WEB-INF > lib > JAR file > META-INF > resources > yourStaticFilesGoHere**

#### Application yml

```yml
server:
  servlet:
    context-path: /sbw
  
spring:
  mvc:
    view:
      prefix: /
      suffix: .jsp
  datasource: 
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456  
    
mybatis:
  type-aliases-package: com.example.sb.model
  mapper-locations: classpath:sql/*.xml
  configuration:
    map-underscore-to-camel-case: true        
        
logging.level:
  root: info
  java.sql.PreparedStatement: debug
  com.example.sb.dao: debug
```

#### Controller

```java
@RestController
@RequestMapping("/user/")
public class UserAction {

	@Autowired
	private UserService userService;  
	
	@RequestMapping(value = "getUser")
	public User getuser(@RequestBody User user) {
		
		return this.userService.getUser(user.getUserId());
	}	
	
	@RequestMapping(value = "getUser2")
	public User getUser2(User user) {
	
		return this.userService.getUser(user.getUserId());
	}
	
	//@RequestParam cannot be ommited in this case
	@RequestMapping(value = "getUser3")
	public User getUser3(@RequestParam Map<String,Object> map) {
	
		return this.userService.getUser(Integer.valueOf(map.get("userId").toString()));
	}

}
```

#### @RequestMapping

  It can be applied to the controller class as well as methods.
  
  It is used to map web requests onto specific handler classe and/or handler method.
  
  You can define the request method like `@RequestMapping(value="/getUser", method=RequestMethod.POST)`.
  
  If you do not specify any mapping this method will resolve all the http request, 
  i.e. you can send GET, POST, HEAD, OPTIONS, PUT, PATCH, DELETE, TRACE 
	request to the specified url and it will be resolved.
  
#### @RequestParam

  Indicating that a method parameter should be bound to a web request parameter.  For example,
  
  `http://localhost:8080/sbw/user/getUser?userId=2`
  
  In the above URL request, the values for userId can be accessed as below:
  
  ```java
  public User getUser( @RequestParam(value="userId") String userId ){
   ...
  }
  ```
  
  If you're sending your data as classic request params, you can bind to object by simply omitting the `@RequestParam`, e.g.
  
  ```java
  @RequestMapping(value = "getUser2")
	public User getUser2(User user) {
	
		return this.userService.getUser(user.getUserId());
	}
  ```
  
  ```jsp
  function getUser2() {
	

	   var search = {
	      "userId" : 2
	   }
	  
	   jQuery.ajax({
	      type: "GET",	  	     
	      url: "/sbw/user/getUser2",
	      data: search, 
	      success :function(result) {
        //....
	     }
	  });
  }
  ```

#### @PathVariable

  `@PathVariable` is similar as `@RequestParam` except that it is used for accessing the values from the URI template.
  
  `http://localhost:8080/sbw/getUser/2?userName=fk`
  
  ```java
  @RequestMapping("/getUser/{userId}")
	public String getUser(@PathVariable(value="userId") int userId,
	@RequestParam(value="userName") String userName,
	){
     .......
  }
  ```
  
#### @Requestbody

  If you're sending json, you have to use `@RequestBody` to get the json data. 

#### @ModelAttribute