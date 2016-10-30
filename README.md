spring boot + redis 实现session
在spring boot的文档中，告诉我们添加@EnableRedisHttpSession来开启spring session支持，配置如下：

    @Configuration  
    @EnableRedisHttpSession  
    public class RedisSessionConfig {  
    }  

而@EnableRedisHttpSession这个注解是由spring-session-data-redis提供的，所以在pom.xml文件中添加：

    <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-redis</artifactId>  
    </dependency>  
    <dependency>  
            <groupId>org.springframework.session</groupId>  
            <artifactId>spring-session-data-redis</artifactId>  
    </dependency>  

 

 

接下来，则需要在application.properties中配置redis服务器的位置了，在这里，我们就用本机：

    spring.redis.host=localhost  
    spring.redis.port=6379  

这样以来，最简单的spring boot + redis实现session共享就完成了，下面进行下测试。

 

首先我们开启两个tomcat服务，端口分别为8080和9090，在application.properties中进行设置【下载地址】   ：

    server.port=8080  

 

接下来定义一个Controller： 

    @RestController  
    @RequestMapping(value = "/admin/v1")  
    public class QuickRun {  
        @RequestMapping(value = "/first", method = RequestMethod.GET)  
        public Map<String, Object> firstResp (HttpServletRequest request){  
            Map<String, Object> map = new HashMap<>();  
            request.getSession().setAttribute("request Url", request.getRequestURL());  
            map.put("request Url", request.getRequestURL());  
            return map;  
        }  
      
        @RequestMapping(value = "/sessions", method = RequestMethod.GET)  
        public Object sessions (HttpServletRequest request){  
            Map<String, Object> map = new HashMap<>();  
            map.put("sessionId", request.getSession().getId());  
            map.put("message", request.getSession().getAttribute("map"));  
            return map;  
        }  
    }  

 

启动之后进行访问测试，首先访问8080端口的tomcat，返回 获取【下载地址】   ：

    {"request Url":"http://localhost:8080/admin/v1/first"}  

 接着，我们访问8080端口的sessions，返回：

    {"sessionId":"efcc85c0-9ad2-49a6-a38f-9004403776b5","message":"http://localhost:8080/admin/v1/first"}  

最后，再访问9090端口的sessions，返回：

    {"sessionId":"efcc85c0-9ad2-49a6-a38f-9004403776b5","message":"http://localhost:8080/admin/v1/first"}  

可见，8080与9090两个服务器返回结果一样，实现了session的共享

 

如果此时再访问9090端口的first的话，首先返回：

    {"request Url":"http://localhost:9090/admin/v1/first"}  

而两个服务器的sessions都是返回：

    {"sessionId":"efcc85c0-9ad2-49a6-a38f-9004403776b5","message":"http://localhost:9090/admin/v1/first"}  

 

通过spring boot + redis来实现session的共享非常简单，而且用处也极大，配合nginx进行负载均衡，便能实现分布式的应用了。



# share-session_with_redis-cache_with_spring-boot
spring boot 和 redis 实现session共享和集中式缓存

登陆：用户信息保存至redis服务中  
http://localhost:8080/login/account/1001/pass/123
登出：删除redis session缓存
http://localhost:8080/login/logout

查询用户列表：查询所有用户信息，并缓存至redis
http://localhost:8080/user/list

查询单个用户信息：将查询到的用户信息缓存至redis
http://localhost:8080/user/findUser/account/1002

修改密码：修改数据库中账户密码，并更新缓存中的用户信息
http://localhost:8080/user/updatePass/account/1002/pass/789

删除用户：删除数据库中用户，并删除redis缓存
http://localhost:8080/user/deleteUser/account/1002
