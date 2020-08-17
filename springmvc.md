# Springmvc

## @RequestMapping

映射请求参数、请求方法或请求头

```java
/**
     * RequestMapping 的其他属性
     * method: 限定请求方式、POST、GET(http协议中的所有请求方式)
     *          method = RequestMethod.POST 默认接受所有请求
     *          不是规定的方式就报错，4XX报错都是客户端报错
     * params: 规定请求参数 接收的是String[]数组
     *          eg. params={"username"}必须携带username的参数否则报404
     *          eg. params={"!username"}必须不携带该参数否则404
     *          eg. params={"username=123"}必须携带username参数且值为123
     * @return
     */
    @ResponseBody
    @RequestMapping(value = "simpleAnalysis",method = RequestMethod.GET)
    public String getMsg(){
        return "";
    }
    @RequestMapping(value = "",params = {"username"})
    public String testParams(){
        return "";
    }

```

映射URL

```java
    /**
     *  URL地址可以写模糊通配符：
     *      ？：代表能代替任意一个字符,0个多个都不行，精确匹配优先
     *      * ：能代替任意多个字符，和一层路径
     *      **：代替多层路径
     * @return
     */
    @RequestMapping("/antTest0?")
    public String antStyleTest(){
        return "";
    }
```

@PathVariable-获取路径上的占位符

```java
/**
     * 路径上可以有占位符：占位符语法就是可以在任意路径的地方写一个{变量名}
     * 只能占一层
     * @return
     */
    @RequestMapping("/pathTest/{id}")
    public String pathTest(@PathVariable("id") String id){
        System.out.println(id);
        return "";
    }
```

## REST

资源：URL即为每一个资源的独一无二的识别符

表现层：把资源具体呈现出来的形式。比如HTML、XML格式

状态转化：HTTP协议，四个操作方式分别对应四个基本操作，GET获取资源，POST新建资源，PUT更新资源，DELETE删除资源

```java
//Rest推荐url这样起名： 资源名/资源标识符
/book/1	:GET----查询1号图书
/book/1 :PUT----更新1号图书
/book/1 :DELETE--删除1号图书
/book	:POST---添加图书

```

## 请求处理

### @RequestParam 

```java
/**
     * @RequestParam 获取请求参数的值
     *  默认方式获取请求参数：
     *      直接给方法入参上写一个和请求方式名相同的变量。这个变量接受请求参数的值，带就有值，没带为null
     *  @RequestParam("user") String username 表示username=request.getParameter("user")
     *      value: 指定要获取的参数的key
     *      required:这个参数是否必须
     *      defaultValue:指定默认值
     * 
     * @return
     */
    @RequestMapping("/paramTest?user=admin")
    public String paramTest(@RequestParam(value = "user",required = false,defaultValue = "没带") String username){

        return "";
    }

```

### @RequestHeader

```java
/**
     * @RequestHeader:获取请求头中的某个key的值
     *  request.getHeader("user-agent")
     *
     * @return
     */
    @RequestMapping("/headerTest")
    public String headerTest(@RequestHeader(value = "User-Agent") String userAgent){
        System.out.println(userAgent);
        return "";
    }
```

### @CookieValue

```java
/**
     * @CookieValue获取cookie的值
     *  以前获取某个cookie
     *  Cookie [] cookies = request.getCookies();
     *  for(Cookie c:cookies){
     *      if(c.getName().equals("JSESSIONID"){
     *          c.getValue();
     *      }
     *  }
     *
     * @return
     */
    @RequestMapping("/cookieTest")
    public String cookieTest(@CookieValue(value = "JSESSIONID") String JID){
        System.out.println(JID);
        return "";
    }
```

### 传入POJO-Springmvc自动封装

若提交的表单含有的参数很多，如果用参数@RequestParam进行指定就很麻烦，所以我们可以对表单的属性进行封装成pojo，直接在参数写一个该pojo对象

```java
//如果我们的请求参数是一个pojo，springmvc会自动的为这个pojo进行赋值？
//将pojo中的每一个属性，从request参数中尝试获取出来，并封装即可
//还可以级联封装，属性的属性
//在前端写属性的属性：<input type="text" name="address.province"/>
//<input type="text" name="address.city"/>
@RequestMapping("/pojoTest")
    public String pojoTest(Bookbook){
        System.out.println(book);
        return "";
    }

```

### 传入原生API

```java
/**
     * @param session
     *  可以直接在入参上写原生API，在返回的页面中在域中获取输出内容
     * @param request
     * @return
     */
    @RequestMapping("/apiTest")
    public String apiTest(HttpSession session, HttpServletRequest request){
        request.setAttribute("reqParam","请求域中的");
        session.setAttribute("sessionParam","session域中的");
        
        return "";
    }
```

### 解决乱码



## 数据输出

```java
/**
     * 除了在方法上传入原生API，还能怎样给页面携带数据过来
     * 1).可以在方法中传入Map，或者Model或者ModelMap，给这些参数里面保存的所有数据都会放在域中
     *      可以在页面中获取.这三者最终都是BindingAwareModelMap在工作，
     *		相当于给BindingAwareModelMap中保存的东西都会被放在请求域中。
     *		Map(interface(jdk)),Model(interface(spring))
     *			\/					||	
     *		ModelMap(class)			||	
     *					\\			\/
     *					ExtendedModelMap
     *							\/	
     *					BindingAwareModelMap
     *
     *2).方法的返回值可以变为ModelAndView类型(模型（数据）和试图（页面）)
     *	既包含试图信息(页面地址)也包含模型数据(给页面带的数据)
     *	而且数据是放在请求域中的
     *
     *域：request,session,application
     *request域用的最多，请求一完成，请求内存的数据便没有了，系统就很快。
     *
     *3).Springmvc提供了一种方式可以临时给Session域中保存数据的方式
     *	使用@SessionAttributes(value = "key")
     *	value="key"指定在BindingAwareModelMap中存放为key的值，在session中也放一份
     *	后来推荐session别用了，可能会引发异常，给session放数据请使用原生API
     * @return
     */
    @RequestMapping("/outputTest")
    public String outputTest(Map<String,Object> map){
        map.put("time","2020");
        return "";
    }
    @RequestMapping("/outputTest1")
    public String outputTest1(Model model){
        model.addAttribute("time","2020-6-6");
        return "";
    }
	@RequestMapping("/modelAndViewTest")
    public ModelAndView modelAndViewTest(){
        //传入的为视图名，之前我们返回的就叫视图名，视图解析器会帮我们最终拼串得到页面的真实地址
        ModelAndView modelAndView = new ModelAndView("success");
        modelAndView.addObject("key","value");
        return modelAndView;
    }

```

