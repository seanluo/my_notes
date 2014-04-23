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
java.lang.IllegalArgumentException at org.springframework.asm.ClassReader.`<init>`(Unknown Source)
...
这个错误是spring3.2和jdk8的不兼容性问题导致的，换成jdk7就解决了。

####整理一下那些配置文件
* tomcat的配置 web.xml 几个重要标签 
	1. `<welcome-file-list>`
	2. `<servlet>` 定义要装载的servlet，有`<servlet-name />` `<servlet-class />` `<init-param />` `<load-on-startup />`等标签供设置。`<init-param />`可以有多个，每个内包含`<param-name/>``<param-value/>`。通过下面的servlet-mapping，tomcat将部分（或全部）url的处理交给这个servlet，对于Spring MVC来说，就是org.springframework.web.servlet.DispatcherServlet。由Spring MVC分发到具体的Controller。并可以在init-param中指定特定的配置文件路径，默认是servlet-name-servlet.xml
	3. `<servlet-mapping>` 每个`<servlet-name>`对应一个`<url-pattern>`
	4. `<filter>` 与servlet相似，一般不是请求的最终处理者，做前置处理用。
	5. `<filter-mapping>` 同上。
	6. `<context-param>` 全局的上下文，一般用`<param-name>`contextConfigLocation`</param-name>` `<param-value>`classpath:applicationContext.xml`</param-value>`加载spring的配置。
	7. `<session-config>` `<session-timeout>`

* Spring的配置applicationContext.xml
	1. `<context:component-scan base-package="com.xxx" />` 这种用来指定从哪个包里通过注解找Service或者Repository组件等。
	2. `<bean>` 这是配置各种bean的地方了，像是dataSource，Hibernate，ViewResolver，自定义的DAO等等各种各样的组件，都在这里按照规则声明及配置依赖关系。

* Spring MVC的配置文件，即tomcat的servlet标签指定的配置文件。默认是servlet-name-servlet.xml。
	1. `<context:component-scan base-package="com.xxx" />` 这种用来指定从哪个包里通过注解找Controller等组件。
	2. `<bean>` 配置viewResolver等等各种各样的组件，这个组件决定了Controller返回的ModelView如何被解析。
	3. `<mvc:resources mapping="/image/**" location="/images/" />` 静态文件配置。

* 从SpringMVC层就把控制权交给了我们的业务逻辑代码。

####Hibernate
* hibernate.cfg.xml：hibernate自身的配置，设置了连接字符串，SQL方言等
* xxx.hbm.xml：映射文件，xml形式声明的数据库存储对象，一般每个配置文件都是对应了一个表。
* 在applicationContext.xml中
	* 用dataSource的bean来利用自身的数据库连接配置让spring加载hibernate
	* 用seesionFactory的bean中注入dataSource并设置hibernate session的一些属性，也指定了加载哪些hbm.xml
	* transactionManager，事务管理，要装配sessionFactory
	* 各种DAO.java，service.java生成的对象，在这里注册并把sessionFactory装配进去
* 映射文件xxx.hbm.xml, 存储对象xxx.java都可以用已有的数据库和数据表生成。
* DAO里面封装基本的操作，增删改查。由于Hibernate必须有事务，所以在DAO的操作或者更上层的service中使用@Transactional注解。

####Spring的注解
* @Component类型，细分还有@Service对应业务层，@Controller对应控制路由层，@Repository对应于数据组件如DAO。
* @Autowired @Resource注解用于自动装配，Autowired使用byType方式查找，Resource使用byName方式查找。