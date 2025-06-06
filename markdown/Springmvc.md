---
title: SpringMVC
date: 2024/08/01 20:46:25
categories:
- Java
tags:
- Java 
---


# SpringMVC

1. 一种设计规范。MVC主要作用是**降低了视图与业务逻辑间的双向偶合**

2. **Model（模型）：**数据模型，提供要展示的数据，因此包含数据和行为，可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据Dao） 和 服务层（行为Service）。也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务

3. **View（视图）：**负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。

   **Controller（控制器）：**接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。也就是说控制器做了个调度员的工作。

   **最典型的MVC就是JSP + servlet + javabean的模式。**

4. ```java
   //映射访问路径,必须是Get请求
   @RequestMapping(value = "/hello",method = {RequestMethod.GET})
   public String index2(Model model){
       model.addAttribute("msg", "hello!");
       return "test";
   }
   ```

5. ```java
   ServletAPI 直接输出
   @Controller
   public class ResultGo {
   
      @RequestMapping("/result/t1")
      public void test1(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
          rsp.getWriter().println("Hello,Spring BY servlet API");
     }
   
   重定向
      @RequestMapping("/result/t2")
      public void test2(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
          rsp.sendRedirect("/index.jsp");
     }
       
   请求转发
      @RequestMapping("/result/t3")
      public void test3(HttpServletRequest req, HttpServletResponse rsp) throws Exception {
          //转发
          req.setAttribute("msg","/result/t3");
          req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req,rsp);
     }
   
   }
   
   
   
   通过SpringMVC来实现转发和重定向 - 无需视图解析器；测试前，需要将视图解析器注释掉
   @Controller
   public class ResultSpringMVC {
      @RequestMapping("/rsm/t1")
      public String test1(){
          //转发
          return "/index.jsp";
     }
   
      @RequestMapping("/rsm/t2")
      public String test2(){
          //转发二
          return "forward:/index.jsp";
     }
   
      @RequestMapping("/rsm/t3")
      public String test3(){
          //重定向
          return "redirect:/index.jsp";
     }
   }
   
   
   
   通过SpringMVC来实现转发和重定向 - 有视图解析器；
   
   重定向 , 不需要视图解析器 , 本质就是重新请求一个新地方嘛 , 所以注意路径问题.
   
   可以重定向到另外一个请求实现 .
   
   @Controller
   public class ResultSpringMVC2 {
      @RequestMapping("/rsm2/t1")
      public String test1(){
          //转发
          return "test";
     }
   
      @RequestMapping("/rsm2/t2")
      public String test2(){
          //重定向
          return "redirect:/index.jsp";
          //return "redirect:hello.do"; //hello.do为另一个请求/
     }
   
   }
   ```

6. 数据显示到前端

   ```java
   1. 第一种 : 通过ModelAndView
       public class ControllerTest1 implements Controller {
   
      public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
          //返回一个模型视图对象
          ModelAndView mv = new ModelAndView();
          mv.addObject("msg","ControllerTest1");
          mv.setViewName("test");
          return mv;
     }
   }
   
   
   2. 第二种：通过ModelMap
   @RequestMapping("/hello")
   public String hello(@RequestParam("username") String name, ModelMap model){
      //封装要显示到视图中的数据
      //相当于req.setAttribute("name",name);
      model.addAttribute("name",name);
      System.out.println(name);
      return "hello";
   }
   
   3.第三种 : 通过Model
   @RequestMapping("/ct2/hello")
   public String hello(@RequestParam("username") String name, Model model){
      //封装要显示到视图中的数据
      //相当于req.setAttribute("name",name);
      model.addAttribute("msg",name);
      System.out.println(name);
      return "test";
   }    
   ```

8. 获取项目名 

   ```jva
   ${pageContext.request.contextPath}
   ```

   

9. **因为默认的配置DispatcherServlet屏蔽了html页面的访问，你需要加上如下：**

   ```java
   <servlet-mapping>
   	<servlet-name>default</servlet-name>
   	<url-pattern>*.html</url-pattern>
   </servlet-mapping>
   ```



**请使用80%的时间打好扎实的基础，剩下18%的时间研究框架，2%的时间去学点英文，框架的官方文档永远是最好的教程。**

