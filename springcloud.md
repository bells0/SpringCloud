# springcloud微服务

# 学前思考

* 如何创建一个SpringCloud项目？

  利用maven工程，然后在maven中导入SpringCloud的相关依赖

  ```xml
      		<dependency><!--springCloud 关键依赖-->
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-dependencies</artifactId>
                  <version>Dalston.SR1</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
  <!--            spring boot依赖，注意这里跟单体spring boot应用有区别-->
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-dependencies</artifactId>
                  <version>1.5.9.RELEASE</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
  
  ```

  

* 服务的提供者与消费者之间如何联系？

  消费者与提供者通过RestTemplate来进行内部通信。关于RestTemplate的使用，可以可以查相关文档。这是微服务相互通信的基础。

* 何为微服务？

![1575960924858](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1575960924858.png)

* 不同的pom之间如何聚合起来？

  ​	**父工程：** 

  ``` xml
  <!-pom文件内容的这几个连接 ->
  <modules>
          <module>microservicecloud-api</module>
      </modules>
  ```

  ​	**子工程**:

  ``` xml
  <!-需要指定ID ->
  <artifactId>microservicecloud-api</artifactId>
  ```

  

 @Configuration使用



​	configuration标签可以用来代替application.xml配置类，进行配置。

![RestComplate 图标](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1575595121288.png)

```java
public String id;
@configuration

```

# 项目起步
### 创建提供者
* 导入相关依赖pom.xml

* 配置yml

* (实体类单独抽离出来作为公共的部分，在pom文件中作为jar包导入)

* DAO

  ``` java
  @Mapper
  public interface DeptDao {
      public boolean addDept(Dept dept);
  
      public Dept findById(Long id);
  
      public List<Dept> findAll();
  }
  ```

  相关mybatis省略

* service

  ``` java
  @Service
  public class DeptServiceImpl implements DeptService {
  
      @Resource
      private DeptDao dao;
      @Override
      public boolean add(Dept dept) {
          return dao.addDept(dept);
      }
  
      @Override
      public Dept get(Long id) {
          return dao.findById(id);
      }
  
      @Override
      public List<Dept> list() {
          return dao.findAll();
      }
  }
  ```

  

* Controller

  ``` java
  
  @RestController
  public class DeptController {
  
      @Autowired
     private DeptService service;
  
      @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
      public boolean add(@RequestBody Dept dept)
      {
          return service.add(dept);
      }
  
      @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
      public Dept get(@PathVariable("id") Long id)
      {
          return service.get(id);
      }
  
      @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
      public List<Dept> list()
      {
          return service.list();
      }
  
  
  }
  ```

  再启动服务，即可访问到数据。提供者完毕，接下来创建消费者

### 创建消费者

​	前面导入依赖、配置yml都与正常springboot程序一样，由于service层在提供者已经有，所以不需要自己写，只需要一个Controller即可。

* Controller

  ``` java
  @RestController
  public class DeptController_Consumer {
      private static final String REST_URL_PREFIX = "http://localhost:8001";
  //    private static final String REST_URL_PREFIX = "http://microservicecloud-dept";
  //172.16.53.1
  @Autowired
  private RestTemplate restTemplate;
  
      @RequestMapping(value = "/consumer/dept/add")
      public boolean add(Dept dept)
      {
          return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
      }
  
      @RequestMapping(value = "/consumer/dept/get/{id}")
      public Dept get(@PathVariable("id") Long id)
      {
          return restTemplate.getForObject(REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
      }
  
      @RequestMapping(value = "/consumer/dept/list")
      public List<Dept> list()
      {
          return restTemplate.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
      }
  
  
  }
  ```

  这里需要注册RestTemplate到Bean容器才可使用，再写一个配置实体类

  ``` java
  @Configuration
  public class ConfigBean {
      @Bean
      public RestTemplate getRestTemplate(){return new RestTemplate(); }
  }
  ```

  这里稍微补充一点RestTemplate知识

  ##### RestTemplate的使用

  RestTemplate是访问restful接口非常方便的方法。

  三个参数

  例子：

``` java
@RestController
public class DeptController_Consumer
{

	//private static final String REST_URL_PREFIX = "http://localhost:8001";
	private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";

	/**
	 * 使用 使用restTemplate访问restful接口非常的简单粗暴无脑。 (url, requestMap,
	 * ResponseBean.class)这三个参数分别代表 REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。
	 */
	@Autowired
	private RestTemplate restTemplate;

	@RequestMapping(value = "/consumer/dept/add")
	public boolean add(Dept dept)
	{
		return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
	}

	@RequestMapping(value = "/consumer/dept/get/{id}")
	public Dept get(@PathVariable("id") Long id)
	{
		return restTemplate.getForObject(REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
	}

	@SuppressWarnings("unchecked")
	@RequestMapping(value = "/consumer/dept/list")
	public List<Dept> list()
	{
		return restTemplate.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
	}

	// 测试@EnableDiscoveryClient,消费端可以调用服务发现
	@RequestMapping(value = "/consumer/dept/discovery")
	public Object discovery()
	{
		return restTemplate.getForObject(REST_URL_PREFIX + "/dept/discovery", Object.class);
	}

}
```

通过RestTemplate，消费者与提供者便联系到了一起，消费者可以使用提供者的方法。



初步构建完毕。

# Eureka服务注册与发现

### 	思考：

* 这是什么？

### Eureka是什么？

EurekaNetflix是的一个子模块,也是核心模块之一。 Eureka是一个基于REST的服务,用于定位服务,以实现云端中间层服务发现和故障转移。**服务注册与发现对于微服务架构来说是非常重要的,有了服务发现与注册,只需要使用服务的标识符,就可以访问到服务,而不需要修改服务调用的配置文件了。**功能类似于dubbo的注册中心,比如Zookeeper

Spring Cloud封装了 Netflix公司开发的 Eureka模块来实现服务注册和发现(请对比 Zookeeper)Eureka采用了c-s的设计架构 Eureka Server作为服务注册功能的服务器,它是服务注册中心。**Netflix的设计遵循的就是AP原则。**而系统中的其他微服务,使用 Eureka的客户端连接到 Eureka Server并维持心跳连接这样系统的维护人员就可以通过 EurekaServerSpringCl来监控系统中各个微服务是否正常运行的一些其他模块(比如zuul就可以通过Eureka Server来发现系统中的其他微服务,并执行相关的逻辑。

主管注册与发现

* 这有什么用？为何需要这个？（上文已回答）
* 如何实现？原理是什么？

### Eureka的实现

![1575962835262](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1575962835262.png)

![1575962845441](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1575962845441.png)



##### 基础架构

![1576029476674.png](D:\java\code\microservicecloud\springcloud.assets\1576029476674.png)



![1576029726118](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576029726118.png)

![1576030111116](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576030111116.png)

![1576030747842](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576030747842.png)

![1576030776530](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576030776530.png)

##### 创建步骤

* 关键pom配置  (Eureka相关依赖)

``` xml
	<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
```



* yml配置(注意，在搭建集群时还需配置其它参数)

```yaml
server: 
  port: 7001

```

* 启动类配置

```java
@SpringBootApplication
@EnableEurekaServer // 关键核心配置 EurekaServer服务器端启动类,接受其它微服务注册进来
public class EurekaServer7001_App
{
	public static void main(String[] args)
	{
		SpringApplication.run(EurekaServer7001_App.class, args);
	}
}
```

注意，在这里配完之后，消费端与提供端也需要使用@EnableEurekaClient来开启Eureka。



* 配置完成后，在7001端口可以访问后台

![1576119958798](D:\java\code\microservicecloud\springcloud.assets\1576119958798.png)

**（红色的报错在下面的自我保护机制中有解释）**

-  相关优化与解释： 

  1. 这里的application的名字是在yml中配置的指定名字![1576120472789](D:\java\code\microservicecloud\springcloud.assets\1576120472789.png)
  2. 在这个status栏下，鼠标放下面可以显示IP，并且指定对应的名称![1576120796950](D:\java\code\microservicecloud\springcloud.assets\1576120796950.png)
  3. 点击进入，可以弹出对应的信息![1576120841168](D:\java\code\microservicecloud\springcloud.assets\1576120841168.png)

  注意，在用这个的时候，使用的 $ 需要在总工程目录下面添加插件配置：

  ```xml
    <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-resources-plugin</artifactId>
                  <configuration>
                      <delimiters>
                          <delimit>$</delimit>
                      </delimiters>
                  </configuration>
              </plugin>
          </plugins>
  ```

  完成之后效果：

  ![1576120992010](D:\java\code\microservicecloud\springcloud.assets\1576120992010.png)





##### Eureka 集群配置

![1576114041101](D:\java\code\microservicecloud\springcloud.assets\1576114041101.png)

![1576114208559](D:\java\code\microservicecloud\springcloud.assets\1576114208559.png)



配置方法与7001类似，只是yml文件做部分修改即可。

相关配置如下：

``` yaml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
#       defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/

```

其它的只需修改7001，defaultZone，改成对应端口的即可。

配置完成，访问 <http://eureka7001.com:7001/>  <http://eureka7002.com:7002/> <http://eureka7003.com:7003/> 即可访问到对应服务。

 ### 自我保护机制是什么？

**遵循AP原则的设计**

![1576122515759](D:\java\code\microservicecloud\springcloud.assets\1576122515759.png)

之前出现红色的提示

作用一句话总结：好死不如赖活着

##### 导致原因

某个微服务不可用，Eureka不会立刻清理，依旧会对该服务的信息进行保存。默认情况下,如果 EurekaServer在一定时间内没有接收到某个微服务实例的心跳, EurekaServer将会注销该实例(默认90秒)但是当网络分区故障发生时,微服务与 EurekaServer之间无法正常通信,以上行为可能变得非常危险了因为微服务本身其实是健康的,**此时本不应该注销这个微服务。**Eureka通过“自我保护模式”来解决这个问题—当 EurekaServer节点在短时间内丢失过多客户端时(可能发生了网络分区故障),那么这个节点就会进入自我保护模式。一旦进入该模式, EurekaServer就会保护服务注册表中的信息,不再删除服务注册表中的数据(也就是不会注销任何微服务)当网络故障恢复后,该 Eureka Server节点会自动退出自我保护模式。

**在自我保护模式中, Eureka Server会保护服务注册表中的信息,不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上时,该 Eureka Server节点就会自动出自我保护模式。它的设计哲学就是宁可保留错误的服务注册信息,也不盲目注销任何可能健康的服务实例。一句话讲解:好死不如赖活着**

综上,自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务(健康的微服务和不健康的微服务都会保留),也不盲目注销任何健康的微服务。使用自我保护模式,可以让 Eureka集群更加的健壮、稳定。

在 Spring Cloud中,可以使用 eureka. server.enable-pr-self-= false禁用自我保护模式。



 ### 什么是CAP原则？

![1576115350152](D:\java\code\microservicecloud\springcloud.assets\1576115350152.png)

##### ACID又是什么？

![1576115413502](D:\java\code\microservicecloud\springcloud.assets\1576115413502.png)

![1576032200353](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576032200353.png)



##### 作为微服务注册中心，Eureka比Zookeeper好在哪里？

![1576115661282](D:\java\code\microservicecloud\springcloud.assets\1576115661282.png)

![1576115706584](D:\java\code\microservicecloud\springcloud.assets\1576115706584.png)

![1576115778613](D:\java\code\microservicecloud\springcloud.assets\1576115778613.png)





# Ribbon负载均衡



