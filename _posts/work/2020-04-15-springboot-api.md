---
layout: post
title: SpringBoot实战，手把手教你写出优雅的接口
categories: SpringBoot
description: SpringBoot接口编程规范
keywords:  Java，SpringBoot
---

> 以项目驱动学习，以实践检验真知



<a name="HkAcO"></a>
# 1.前言

<br />一个后端的接口大致分为四个部分组成：接口地址（url）、接口请求方式（get、post等）、请求数据（request）、响应数据（response）。如何构建这几个部分每个公司要求都不同，没有什么"一定是最好的"标准，但一个优秀的后端接口和一个糟糕的后端接口对比起来差异还是蛮大的，其中最重要的关键点就是看**是否规范 ！**<br />本文就一步步演示如何构建起一个优秀的后端接口体系，体系构建好了自然就有了规范，同时再构建新的后端接口也十分轻松。

**文章末尾贴上了项目的演示的github地址，clone下来即可运行，并且我将每一次的优化记录都分别做了代码提交，你可以清晰的看到项目的改进过程！**<br />**<br />**
<a name="9IWSx"></a>
# 2.所需的依赖包

<br />这里用的是SpringBoot配置项目，本文讲解的重点是后端接口，所以只需要导入一个spring-boot-starter-web包就可以了：<br />

```xml
  <!--web依赖包，web应用必备-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

<br />本项目还采用了swagger来生成API文档，lombok来简化类，不过这两者不是必须的，可用可不用。<br />

<a name="6XL8j"></a>
# 3.参数校验

<br />一个接口一般对参数（请求数据）都会进行安全校验，参数校验的重要性自然不必多说，那么如何对参数进行校验就有讲就了。<br />

<a name="3cNc2"></a>
## 业务层校验
首先我们来看一下最常见的做法，就是在业务层进行参数校验：<br />

```java
 public String addUser(User user) {
        //参数校验
        if(Objects.isNull(user) || Objects.isNull(user.getId()) || Objects.isNull(user.getAccount())
                || Objects.isNull(user.getPassword())|| Objects.isNull(user.getEmail())
        ){
            return "对象或者对象字段不能为空";
        }

        if (StringUtils.isEmpty(user.getAccount()) || StringUtils.isEmpty(user.getPassword()) || StringUtils.isEmpty(user.getEmail())) {
            return "不能输入空字符串";
        }
        if (user.getAccount().length() < 6 || user.getAccount().length() > 11) {
            return "账号长度必须是6-11个字符";
        }
        if (user.getPassword().length() < 6 || user.getPassword().length() > 16) {
            return "账号长度必须是6-16个字符";
        }
        if (!Pattern.matches("^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\\.[a-zA-Z0-9_-]+)+$", user.getEmail())) {
            return "邮箱格式不正确";
        }
        //直接返回业务逻辑
        return "success";
    }
```

<br />这样做当然是没有错的，而且格式排版格式也一目了然，不过这样实在是太繁琐了，这还没有进行业务操作呢,光是一个参数校验就已经这么多行代码，实在不够优雅，接下来来改进一下，使用Spring Validator和Hibernate Validator这两套来进行方便的参数校验！ 这两套Validator依赖包已经包含在前面所有的web依赖包里面了 ，所以可以直接使用。<br />

<a name="2rXRW"></a>
## Validator +BindResult 进行校验

<br />Validator可以非常方便的指定校验规则，并自动帮你完成校验。首先在入参里需要校验的字段加上注解，每个注解对应不同的校验规则，并可指定校验失败后的信息：<br />

```java
@Data
@ApiModel("用户")
public class User {

    @ApiModelProperty("用户id")
    @NotNull(message = "用户id不能为空")
    private Long id ;

    @ApiModelProperty("用户账号")
    @NotNull(message = "用户账号不能为空")
    @Size(min = 6,max = 11,message = "账号长度必须是6-11个字符")
    private String account;

    @ApiModelProperty("用户密码")
    @NotNull(message = "用户密码不能为空")
    @Size(min = 6,max = 16,message = "密码必须是6-16个字符")
    private String password;

    @ApiModelProperty("用户邮箱")
    @NotNull(message = "用户邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;
}

```

<br />校验规则和错误信息配置完毕后，接下来只需要在接口需要校验的参数上加上@Valid注解，并添加BindResult参数即可方便完成验证：<br />

```java
@RestController
@Api(tags = {"用户接口"})
@RequestMapping("/user")
public class UserController {
    @Autowired
    private IUserService userService;
    
    @ApiOperation("添加用户")
    @PostMapping("/addUser")
    public String addUser(@Valid  @RequestBody User user, BindingResult bindingResult){
        //如果有参数校验失败，会将错误信息封装成对象在BingResult里
        for (ObjectError error : bindingResult.getAllErrors()) {
            return error.getDefaultMessage();
        }
        return userService.addUser(user);
    }
    
}
```

<br />这样当请求参数传递到接口的时候Validator就自动完成校验了，校验的结果就会封装到BindResult中去，如果有错误信息我们直接返回给前端，业务逻辑代码也根本没有执行下去。此时，业务层的校验代码就已经不需要了：<br />

```java
  public String addUser(User user) {
        //直接返回业务逻辑
        return "success";
    }
```

<br />现在可以看下参数校验的效果，我们故意给这个接口传递一个不符合校验规则的参数，先传递一个错误数据给接口，故意将password这个字段不满足校验条件：<br />

```json
{
	"account": "12345678",
	"email": "123@qq.com",
	"id": 0,
	"password": "123"
}
```

<br />再来看一下接口的响应数据：<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1587693250254-ec8bafcc-0178-4521-b01c-4c4d73b0a6eb.png#align=left&display=inline&height=479&margin=%5Bobject%20Object%5D&name=image.png&originHeight=479&originWidth=715&size=25288&status=done&style=none&width=715)<br />这样是不是方便很多，不难看出使用Validator校验有如下几个好处:

1. 简化代码，之前业务层那么一大段校验代码都被省略了 。
1. 使用方便。那么多检验规则可以轻而易举的实现，比如邮箱格式验证，之前自己手写正则表达要手写那么一大串，还容易出错，用Validator直接一个注解搞定。（还有更多校验规则注解，可以自行去了解）
1. 减少耦合度，使用Validator能够让业务层只关注业务逻辑，从基本的参数校验逻辑中脱离出来。


<br />使用Validator+BindResult已经是非常方便实用的参数校验方式了。 在实际开发中也有很多项目就是这么做的，不过这样还是不太方便。因为你每一个接口都要添加一个BindResult参数，然后再提取错误信息返回给前端。这样有点麻烦，并且重复代码很多（尽管可以将这个重复代码封装成方法）。我们是否可以去掉BindResult这一步呢？当然是可以的。<br />

<a name="cez9Y"></a>
## Validator + 自动抛出异常

<br />我们完全可以将BindingResult这一步给去掉：<br />

```java
  @ApiOperation("添加用户")
    @PostMapping("/addUser")
    public String addUser(@Valid  @RequestBody User user){
        return userService.addUser(user);
    }
```

<br />去掉之后会发生什么事情呢？直接来试验一下，还是按照之前一样传递一个不符合规则的参数给接口。此时我们观察控制台可以发现接口引发 `MethodArgumentNotValidException` 异常了：<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1587694631965-ec9769f6-c31a-4196-b2ae-a175c515cbfe.png#align=left&display=inline&height=57&margin=%5Bobject%20Object%5D&name=image.png&originHeight=57&originWidth=1749&size=73220&status=done&style=none&width=1749)<br />
<br />其实这样就已经达到我们想要的结果了，参数校验不通过自然就不执行接下来的业务逻辑，去掉BindingResult后自动引发异常，异常发生了自然而然就不会执行业务逻辑。也就是说，我们完全没必要添加相关BingResult相关操作嘛。不过事情还没有完，异常是引发了，可我们并没有编写返回错误信息的代码呀，那参数检验失败了会响应什么数据给前端呢？我们来看一下刚才异常发生后接口响应的数据：<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1587694646285-706da21f-4e08-4e59-a355-85a2a29dc109.png#align=left&display=inline&height=630&margin=%5Bobject%20Object%5D&name=image.png&originHeight=630&originWidth=674&size=41028&status=done&style=none&width=674)<br />
<br />没有，是直接将整个错误对象相关信息都响应给前端了！  这样就很难受，不过解决这个问题也很简单，就是我们接下来的全局异常处理。<br />

<a name="YteKI"></a>
# 4.全局异常处理

<br />参数校验失败会自动引发异常，我们当然不可能再去手动捕捉异常进行处理，不然还不如用之前BindResult方式呢。**又不想手动捕捉这个异常，又要对这个异常进行处理。**那正好使用SpringBoot全局异常处理来达到一劳永逸的效果！

<a name="WCkIs"></a>
## 基本使用

<br />首先，我们需要新建一个类，在这个类上加上@ControllerAdvice 或 RestControllerAdvice注解，这个类就配置成全局异常处理类了。<br />（这个根据你的Controller层用的是@Controller还是@RestController来决定）然后在类中新建方法，在方法上加上@ExceptionHandler 注解并指定你想处理的异常类型，接着在方法在方法编写对该异常的操作逻辑，就完成了对该异常的全局处理！我们现在就来演示一下对参数校验失败抛出的 ``MethodArgumentNotValidException` `<br />全局处理：<br />

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public String MethodArgumentNotValidExceptionHandler(MethodArgumentNotValidException e){
        //从异常对象中拿到ObjectError对象
        ObjectError error = e.getBindingResult().getAllErrors().get(0);
        //然后提取错误提示信息进行返回
        return error.getDefaultMessage();
    }

}

```
 我们再来看下这次校验失败后的响应数据：<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1587695694495-e7b45bab-ec90-4ec8-95af-9f00f0898e86.png#align=left&display=inline&height=185&margin=%5Bobject%20Object%5D&name=image.png&originHeight=185&originWidth=585&size=7959&status=done&style=none&width=585)<br />
<br />没错，这次返回就是我们制定的错误提示信息，我们通过全局异常处理优雅的实现了我们想要的功能！ 以后我们再想写接口参数校验，就只需要在入参的成员变量上加上Validator校验规则注解，然后在参数上加上@Valid注解即可完成校验，校验失败会自动返回错误提示信息，无需任何其它代码。<br />

<a name="YdHz1"></a>
## 自定义异常

<br />全局处理当然不会只能处理一种异常，用途也不仅仅是对一个参数校验方式进行优化。在实际开发，如何对异常处理其实是一个很麻烦的事情。传统处理异常一般有以下烦恼：

1. 是捕获异常（ `try...catch` ）还是抛出异常。
1. 是在controller层做处理还是在service层处理又或是在dao层做处理。
1. 处理异常的方式是啥也不做，还是返回特定数据，如果返回又返回什么数据。
1. 不是所有异常我们都能预进行捕捉，如果发生了没有捕捉到的异常该怎么办？


<br />以上这些问题都可以用全局异常处理来解决，全局异常处理也叫统一异常处理，全局和统一处理代表什么?<br />**代表规范  **规范有了， 很多问题就会迎刃而解！全局异常处理的基本使用方式大家都已经知道了，我们接下来更近一步的规范项目中的异常处理方式：**自定义异常。 **在很多情况下，我们需要手动抛出异常，比如在业务层有些条件并不符合业务逻辑，我这时候就可以手动抛出异常从而触发事务回滚。那手动抛出异常最简单的方式就是 <br />` throw new RuntimeException（"异常信息"）`了，不过使用自定义更好一些：

1. 自定义异常可以携带更多的信息，不像这样只能携带一个字符串。
1. 项目开发中经常是很多人负责不同的模块，使用自定义异常可以统一了解对外异常展示的方式。
1. 自定义异常语义更加清晰明了，一看就知道是项目中手动抛出的异常。


<br />我们现在就来开始写一个自定义异常：<br />

```java
@Getter
public class APIException extends RuntimeException {
    private int code;
    private String msg;

    public APIException(){
        this(1001,"接口错误");
    }

    public APIException(String msg){
        this(1001,msg);
    }

    public APIException(int code, String msg){
        super(msg);
        this.code = code;
        this.msg = msg;
    }
}

```

<br />在刚才的全局异常处理类中记得添加对我们自动义异常的处理：<br />

```java
@ExceptionHandler(APIException.class)
    public String APIExceptionHandler(APIException e){
        return e.getMsg();
    }

```

<br />这样就对异常的处理就比较规范了，当然还可以添加对`Exception`的处理，这样无论发生什么我们都能屏蔽掉，然后响应数据给前端，不过建议最后项目上线时这样做，能够屏蔽掉错误信息暴露给前端，在开发中为了方便还是不要这样做。现在全局异常处理和自定义异常已经弄好了，不知道大家有没有发现一个问题，就是当我们抛出自定义异常的时候全局异常处理只响应了异常中的错误信息msg给前端，并没有将错误代码code返回，这就要引申出我们接下来要讲的东西了： **数据统一响应**<br />**<br />**
<a name="OEI4F"></a>
# 5.数据统一响应
现在我们规范好了参数校验方式和异常处理方式，然而还没有规范响应数据！比如我要获取一个分页信息数据，获取成功了呢自然就返回的数据列表，获取失败就会响应异常信息，即一个字符串，就是说前端开发者压根就不知道后端响应过来的数据会是啥样的！ 所以，统一响应数据是前后端规范中必须要做的！<br />
<br />

<a name="enPkf"></a>
## 自定义统一响应体

<br />统一数据响应第一步肯定要做的就是我们自定自定义一个响应体类，无论后台是运行正常还是发生异常，响应给前端的数据格式是不变的！ 那么如何定义响应体呢？可以参考我们自定义异常类，也来一个响应信息代码code和响应信息说明msg：<br />

```java
@Data
public class ResultVo<T> {
    /**
     * 状态码 比如1000代表响应成功
     */
    private int code;

    /**
     * 响应信息，用来说明响应情况
     */
    private String msg;

    /**
     * 响应的具体数据
     */
    private T data;
    
    public ResultVo(T data){
        this (1000,"success",data);
    }
    
    public ResultVo(int code ,String msg,T data){
        this.code = code ;
        this.msg = msg;
        this.data = data;
    }
    
}
```

<br />然后我们修改一下全局异常处理那的返回值：<br />

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResultVo<String> MethodArgumentNotValidExceptionHandler(MethodArgumentNotValidException e){
        //从异常对象中拿到ObjectError对象
        ObjectError error = e.getBindingResult().getAllErrors().get(0);
        //然后提取错误提示信息进行返回
        return new ResultVo<>(1001,"参数校验失败",error.getDefaultMessage());
    }

    @ExceptionHandler(APIException.class)
    public ResultVo<String> APIExceptionHandler(APIException e){
        return new ResultVo<>(e.getCode(),"响应失败",e.getMsg());
    }

}
```

<br />我们再来看一下此时如果发生了异常了会相应什么数据给前端：<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1587699521685-44cb6905-0247-4a6b-9518-73b51da3a08e.png#align=left&display=inline&height=249&margin=%5Bobject%20Object%5D&name=image.png&originHeight=249&originWidth=595&size=11857&status=done&style=none&width=595)<br />
<br />Ok,这个异常信息就非常好了，状态码和响应说明说明还有错误提示数据都返给了前端，并且是所有异常都会返回相同的格式！异常这里搞定了，别忘了我们到接口那也要修改返回类型，我们新增一个接口好来看看效果：<br />

```java
@GetMapping("/getUser")
    public ResultVo<User> getUser(){
        User user = new User();
        user.setAccount("133333333");
        user.setId(33L);
        user.setPassword("333333333");
        user.setEmail("333@qq.com");

        return new ResultVo<>(user);
    }
```

<br />看一下如果响应正确返回的是什么结果：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1587700525436-4da67d1f-ae2f-4cab-9c37-d0ef352f7bb8.png#align=left&display=inline&height=272&margin=%5Bobject%20Object%5D&name=image.png&originHeight=272&originWidth=471&size=13459&status=done&style=none&width=471)<br />这样无论是正确响应还是发生异常，响应数据的格式都是统一的，十分规范！<br />
<br />数据格式是规范了，不过响应码code和响应信息msg还没有规范呀！ 大家发现没有，无论是正确响应，响应码和响应信息是想怎么设置就怎么设置，要是10个人开发人员对同一个类型写10个不同的响应码，那这个统一响应体的格式规范就毫无意义！ 所以，必须要将响应码和响应信息给规范起来。<br />

<a name="TKAzY"></a>
## 响应码枚举

<br />要规范响应体中的响应码和响应信息用枚举简直再恰当不过了，我们现在就来创建一个响应码枚举类：<br />

```java
@Getter
@AllArgsConstructor
public enum  ResultCode {
    
    SUCCESS(1000,"操作成功"),
    
    FAILED(1001,"响应失败"),
    
    VALIDATE_FAILED(1002,"参数校验失败"),
    
    ERROR(5000,"未知错误"),
    
    ;
    /**
     * 响应码
     */
    private int code;

    /**
     * 提示消息
     */
    private String msg;
}

```

<br />然后修改响应体的构造方法，让其只准接受响应码来设置响应码和响应信息：<br />

```java
   public ResultVo(T data){
        this (ResultCode.SUCCESS,data);
    }

    public ResultVo(ResultCode resultCode,T data){
        this.code = resultCode.getCode() ;
        this.msg = resultCode.getMsg();
        this.data = data;
    }

```

<br />然后同时修改全局异常处理的响应码设置方式：<br />

```java
 @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResultVo<String> MethodArgumentNotValidExceptionHandler(MethodArgumentNotValidException e){
        //从异常对象中拿到ObjectError对象
        ObjectError error = e.getBindingResult().getAllErrors().get(0);
        //注意哦，这里传递的响应码枚举
        return new ResultVo<>(ResultCode.VALIDATE_FAILED,error.getDefaultMessage());
    }

    @ExceptionHandler(APIException.class)
    public ResultVo<String> APIExceptionHandler(APIException e){
        //注意哦，这里传递的响应码枚举
        return new ResultVo<>(ResultCode.FAILED,e.getMsg());
    }
```

<br />这样响应码和响应信息只能是枚举规定的那几个，就真正做到了响应数据格式、响应码和响应信息规范化、统一化。<br />

<a name="SiZTP"></a>
## 全局处理响应数据

<br />接口返回响应体+异常也返回统一响应体，其实这样已经很好了，但还是有可以优化的地方。要知道一个项目下来定义的接口搞个几百个太正常不过了，要是每一个接口返回数据时都要用响应体包装一下好像有点麻烦，有没有办法省去这个包装过程呢？当然是有滴，还是要用到全局处理。<br />
<br />首先，先创建一个类加上注解使其成为全局处理类。然后继承ResponseBodyAdvice接口重写其中的方法，即可对我们的 `controler` 进行增强操作，还是要用到全局处理。<br />
<br />具体看代码和注释：<br />

```java
@RestControllerAdvice(basePackages = "com.liuscoding.api.controller")
public class ResponseControllerAdvice implements ResponseBodyAdvice<Object> {
    @Override
    public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> aClass) {
        //如果接口返回的类型的本身就是ResultVO那就没有必要进行额外的操作，返回false
        return ! returnType.getGenericParameterType().equals(ResultVo.class);
    }

    @Override
    public Object beforeBodyWrite(Object data, MethodParameter returnType, MediaType mediaType, Class<? extends HttpMessageConverter<?>> aClass, ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse) {
        
        //String 类型不能直接包装，所以要进行特别的处理
        if(returnType.getGenericParameterType().equals(String.class)){
            ObjectMapper  objectMapper = new ObjectMapper();
            try {
                return objectMapper.writeValueAsString(new ResultVo<>(data));
            } catch (JsonProcessingException e) {
                throw new APIException("返回String类型错误");
            }
        }
        //将原本的数据包装在ResultVo里面
        return new ResultVo<>(data);
    }
}

```

<br />重写的这两个方法是用来在 `controller`  将数据进行返回前进行增强包装，supports方法要返回为true才会执行<br />`beforeBodyWrite` 方法，所以如果有些情况不需要进行增强的时候可以在 `supports` 方法里进行判断。对返回数据进行真正的操作在 `beforeBodyWrite` 方法中，我们可以直接在该方法里包装数据，这样就不需要每个接口包装了，省去了很多麻烦。<br />
<br />我们可以现在去掉接口的数据包装来看下效果：<br />

```java

    @GetMapping("/getUser")
    public User getUser(){
        User user = new User();
        user.setAccount("133333333");
        user.setId(33L);
        user.setPassword("333333333");
        user.setEmail("333@qq.com");

        return user;
    }
```

<br />然后我们来看下响应数据：<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/440247/1587714248030-727811b3-0f82-43fd-a655-bf8c820deee3.png#align=left&display=inline&height=312&margin=%5Bobject%20Object%5D&name=image.png&originHeight=312&originWidth=492&size=14709&status=done&style=none&width=492)<br />
<br />成功对数据进行了包装！<br />

>  注意：beforeBodyWirte方法里包装数据无法对String类型的数据直接进行强转，所以要特殊处理，这里不讲过多的细节，有兴趣可以自行深入了解。



<a name="IEv6i"></a>
# 6.总结
自此整个后端接口基本体系就构建完毕了。<br />

- 通过Validator  + 自动抛出异常来完成了方便的参数校验
- 通过全局异常处理  +  自定义异常来完成了异常操作的规范
- 通过数据统一响应来完成了响应数据的规范
- 多个方面组装非常优雅的完成了后端接口的协调，让开发人员有更多的经历注重业务逻辑代码，轻松构建后端接口。


<br />再次强调，项目体系该怎么构建，后端接口该怎么写都没有一个绝对统一的标准，不是说一定要按照本文的来才是最好的，你怎么样都可以，本文每一个环节你都可以按照自己的想法来进行编码，我只是提供了一个思路。<br />
<br />最后在这里放上此项目的[github地址](https://github.com/liusCoding/java-learning/tree/master/validation-exception-handler)地址，clone到本地即可直接运行，并且我将每一次的优化记录都分别做了代码提交，你可以清晰的看到项目的改进过程，如果对你有帮助请在github上点个star，我还会继续更新更多【项目实践】哦！<br />

