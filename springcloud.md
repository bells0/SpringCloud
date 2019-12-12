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

Eureka是什么？

EurekaNetflix是的一个子模块,也是核心模块之一。 Eureka是一个基于REST的服务,用于定位服务,以实现云端中间层服务发现和故障转移。**服务注册与发现对于微服务架构来说是非常重要的,有了服务发现与注册,只需要使用服务的标识符,就可以访问到服务,而不需要修改服务调用的配置文件了。**功能类似于dubbo的注册中心,比如 Zookeeper

主管注册与发现

* 这有什么用？为何需要这个？（上文已回答）
* 如何实现？原理是什么？
* 自我保护机制是什么？
* 什么是CAP原则？

![1576115350152](D:\java\code\microservicecloud\springcloud.assets\1576115350152.png)

* ACID又是什么？

![1576115413502](D:\java\code\microservicecloud\springcloud.assets\1576115413502.png)

![1576032200353](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576032200353.png)

![1576032279131](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576032279131.png)

![1576032296229](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576032296229.png)

* 作为微服务注册中心，Eureka比Zookeeper好在哪里？

  ![1576115661282](D:\java\code\microservicecloud\springcloud.assets\1576115661282.png)

![1576115706584](D:\java\code\microservicecloud\springcloud.assets\1576115706584.png)

![1576115778613](D:\java\code\microservicecloud\springcloud.assets\1576115778613.png)





![1575962488189](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1575962488189.png)





![1575962636513](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1575962636513.png)



 ![1575962835262](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1575962835262.png)

![1575962845441](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1575962845441.png)

![1576029476674](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576029476674.png)

![1576029726118](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576029726118.png)

![1576030111116](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576030111116.png)

![1576030747842](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576030747842.png)

![1576030776530](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1576030776530.png)



##### Eureka 集群配置

![1576114041101](D:\java\code\microservicecloud\springcloud.assets\1576114041101.png)

![1576114208559](D:\java\code\microservicecloud\springcloud.assets\1576114208559.png)

hosts文件的修改



# Ribbon负载均衡