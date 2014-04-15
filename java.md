###Java

####名词
1. maven: 通过pom.xml管理依赖，自动下载到本地repo。其自身设置在%M2_HOME%/conf/setting.xml。
2. artifact: 是与deploy有关的，在其中设置好按照什么样式部署哪些文件和依赖库。
3. tomcat：web容器。server.xml配置vhost，web.xml是针对每个webapp的配置。
4. Spring：自身是用来IoC（反转控制），DI（依赖注入）用的，就是通过包含beans配置的xml文件，反向在java代码中给变量赋值（值或引用）。
5. Spring MVC：一套MVC框架，可以借助spring的上述机制与Hibernate结合。
6. Hibernate：持久化框架，ORM用。

####IDE
使用intelliJ idea 12的，要在settings》plugins中启用tomcat、maven、spring-webservices插件。

####问题
java.lang.IllegalArgumentException at org.springframework.asm.ClassReader.<init>(Unknown Source)
...
这个错误是spring3.2和jdk8的不兼容性问题导致的，换成jdk7就解决了。

####TODO:整理一下那些配置文件