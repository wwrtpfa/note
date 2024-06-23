1. @RestController==@Requestmapping+@RequestBody


2. YAML 数组写法

   ```java
   pets: 
     -dog   
     -cat  
     -pig
   ```

3. [@ConfigurationProperties](https://www.cnblogs.com/tian874540961/p/12146467.html)：用于批量绑定配置文件中的配置；

   可以从注解说明中看到，当将该注解作用于方法上时，如果想要有效的绑定配置，那么该方法需要有@Bean注解且所属Class需要有@Configuration注解。

4. @Value：只能一个一个的指定需要绑定的配置。

   ```java
   person.properties 的配置如下:
       
   person.last-name=李四
   person.age=12
   person.birth=2000/12/15
   person.boss=false
   person.maps.k1=v1
   person.maps.k2=14
   person.lists=a,b,c
   person.dog.name=dog
   person.dog.age=2
   
   
   ```

   ```java
   @PropertySource(value = "classpath:person.properties")//指向对应的配置文件
   @Component
   @ConfigurationProperties(prefix = "person")
   public class Person {
   
       private String lastName;
   
       private Integer age;
   
       private Boolean boss;
   
       private Date birth;
   
       private Map<String, Object> maps;
   
       private List<Object> lists;
   
       private Dog dog;
   
       public Person() {
   
       }
   
       public String getLastName() {
           return lastName;
       }
   
       public void setLastName(String lastName) {
           this.lastName = lastName;
       }
   
       public Integer getAge() {
           return age;
       }
   
       public void setAge(Integer age) {
           this.age = age;
       }
   
       public Boolean getBoss() {
           return boss;
       }
   
       public void setBoss(Boolean boss) {
           this.boss = boss;
       }
   
       public Date getBirth() {
           return birth;
       }
   
       public void setBirth(Date birth) {
           this.birth = birth;
       }
   
       public Map<String, Object> getMaps() {
           return maps;
       }
   
       public void setMaps(Map<String, Object> maps) {
           this.maps = maps;
       }
   
       public List<Object> getLists() {
           return lists;
       }
   
       public void setLists(List<Object> lists) {
           this.lists = lists;
       }
   
       public Dog getDog() {
           return dog;
       }
   
       public void setDog(Dog dog) {
           this.dog = dog;
       }
   
       public Person(String lastName, Integer age, Boolean boss, Date birth, Map<String, Object> maps, List<Object> lists, Dog dog) {
           this.lastName = lastName;
           this.age = age;
           this.boss = boss;
           this.birth = birth;
           this.maps = maps;
           this.lists = lists;
           this.dog = dog;
       }
   
       @Override
       public String toString() {
           return "Person{" +
                   "lastName='" + lastName + '\'' +
                   ", age=" + age +
                   ", boss=" + boss +
                   ", birth=" + birth +
                   ", maps=" + maps +
                   ", lists=" + lists +
                   ", dog=" + dog +
                   '}';
       }
   }
   
   
   ```

   5.@ImportResource 导入 Spring 配置文件

   ```java
   将 beans.xml 加载到项目中
   @ImportResource(locations = {"classpath:/beans.xml"})
   ```

   

   6.**全注解方式加载 Spring 配置**

   使用 @Configuration 注解定义配置类，替换 Spring 的配置文件；

   配置类内部可以包含有一个或多个被 @Bean 注解的方法，这些方法会被 AnnotationConfigApplicationContext 或 AnnotationConfigWebApplicationContext 类扫描，构建 bean 定义（相当于 Spring  配置文件中的<bean></bean>标签），方法的返回值会以组件的形式添加到容器中，组件的 id 就是方法名。

   ```java
   /**
   * @Configuration 注解用于定义一个配置类，相当于 Spring 的配置文件
   * 配置类中包含一个或多个被 @Bean 注解的方法，该方法相当于 Spring 配置文件中的 <bean> 标签定义的组件。
   */
   @Configuration
   public class MyAppConfig {
       /**
        * 与 <bean id="personService" class="PersonServiceImpl"></bean> 等价
        * 该方法返回值以组件的形式添加到容器中
        * 方法名是组件 id（相当于 <bean> 标签的属性 id)
        */
       @Bean
       public PersonService personService() {
           System.out.println("在容器中添加了一个组件:peronService");
           return new PersonServiceImpl();
       }
   }
   
   测试：
       @SpringBootTest
   class HelloworldApplicationTests {
       @Autowired
       Person person;
       //IOC 容器
       @Autowired
       ApplicationContext ioc;
       @Test
       public void testHelloService() {
           //校验 IOC 容器中是否包含组件 personService
           boolean b = ioc.containsBean("personService");
           if (b) {
               System.out.println("personService 已经添加到 IOC 容器中");
           } else {
               System.out.println("personService 没添加到 IOC 容器中");
           }
       }
       @Test
       void contextLoads() {
           System.out.println(person);
       }
   }
   ```

   7. 多版本环境配置

   ```java
   properties:spring.profiles.active=dev
   
   yaml:
   	server:
     port: 8080
   spring:
     profiles:
       active: dev 
   ---
   server:
     port: 8081
   spring:
     profiles: dev
   
   ---
   server:
     port: 8082
   spring:
     profiles: test
   ```

   8. swagger

      > 1. Docket
      > 2. apiInfo():配置文档信息

   9. kafka

      + `Kafka` 是一个分**布式流式平台**，它有三个关键能力
        1. 订阅发布记录流，它类似于企业中的`消息队列` 或 `企业消息传递系统`
        2. 以容错的方式存储记录流
        3. 实时记录流
      + Kafka 有四个核心API，它们分别是
        - Producer API，它允许应用程序向一个或多个 topics 上发送消息记录
        - Consumer API，允许应用程序订阅一个或多个 topics 并处理为其生成的记录流
        - Streams API，它允许应用程序作为流处理器，从一个或多个主题中消费输入流并为其生成输出流，有效的将输入流转换为输出流。
        - Connector API，它允许构建和运行将 Kafka 主题连接到现有应用程序或数据系统的可用生产者和消费者。例如，关系数据库的连接器可能会捕获对表的所有更改
      + Kafka 的底层使用 Zookeeper 储存元数据

   10. **如果properties和yml配置文件同时存在springboot项目中；那么这两类型配置文件都有效**
   
       - **@GetMapping：** 处理get请求，传统的RequestMapping来编写应该是@RequestMapping(value = “/get/{id}”, method = RequestMethod.GET)
         新方法可以简写为：
         @GetMapping("/get/{id}")
   
       - **@PostMapping：** 处理post请求，传统的RequestMapping来编写应该是@RequestMapping(value = “/get/{id}”,method = RequestMethod.POST)
         新方法可以简写为：
         @PostMapping("/get/{id}")
   
       - **@PutMapping：** 和PostMapping作用等同，都是用来向服务器提交信息。如果是添加信息，倾向于用@PostMapping，如果是更新信息，倾向于用@PutMapping。两者差别不是很明显。
   
       - **@DeleteMapping** 删除URL映射，具体没有再实践中用过，不知道好在什么地方
   
       - **@PatchMapping** **至今不知如何用，再什么场景下用。。。有知道的欢迎留言或私信**
   
   11. get请求和post请求的区别：
   
       **get请求特点：**
       a. 请求参数会添加到请求资源路径的后面，只能添加少量参数（因为请求行只有一行，大约只能存放2K左右的数据）
       b. 请求参数会显示在浏览器地址栏，路由器会记录请求地址 （极为的不安全）
       c.如果传输中文，必定会乱码（原因：get请求默认编码格式为：IIO-8859-1,后台编码格式一般为：GBK或者UTF-8）
       **post请求的特点：**
       a. 请求参数添加到实体内容里面，可以添加大量的参数（也解释了为什么浏览器地址栏不能发送post请求，在地址栏里我们只能填写URL，并不能进入到Http包的实体当中）
       b. 相对安全，但是，post请求不会对请求参数进行加密处理（可以使用https协议来保证数据安全）
   
   12. Iterato迭代Iterable
   
   ```java
   List<String> list = new ArrayList<>();
    
   list.add("one");
   list.add("two");
   list.add("three");
    
   Iterator<String> iterator = list.iterator();
    
   while(iterator.hasNext()) {
       String element = iterator.next();
       System.out.println( element );
   }
   ```

​     

---

0815

1. [JWT三部分](https://blog.csdn.net/WuLex/article/details/117109852)
   - `Header` 声明信息。 在`Header`中通常包含了两部分：`type`：代表`token`的类型，这里使用的是`JWT`类型。
     alg:使用的`Hash`算法，例如`HMAC SHA256`或`RSA`.
     `{ “alg”: “HS256”, “typ”: “JWT” }` 这会被经过base64Url编码形成第一部分
   - `Payload token`的第二个部分是荷载信息，它包含一些声明Claim(实体的描述，通常是一个`User`信息，还包括一些其他的元数据)
     声明分三类:
     1)`Reserved Claims`,这是一套预定义的声明，并不是必须的,这是一套易于使用、操作性强的声明。包括：`iss(issuer)、exp(expiration time)、sub(subject)、aud(audience)`等
     2)`Plubic Claims`,
     3)`Private Claims`,交换信息的双方自定义的声明 `{ “sub”: “1234567890”, “name”: “John Doe”,“admin”: true }` 同样经过`Base64Url`编码后形成第二部分
   - `signature` 使用`header`中指定的算法将编码后的`header`、编码后的`payload`、一个`secret`进行加密。例如使用的是`HMAC SHA256`算法，大致流程类似于: `HMACSHA256( base64UrlEncode(header) + “.” + base64UrlEncode(payload),secret)`这个`signature`字段被用来确认`JWT`信息的发送者是谁，并保证信息没有被修改 