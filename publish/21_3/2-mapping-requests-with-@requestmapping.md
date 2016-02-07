# 21.3.2 使用@RequestMapping注解映射请求路径

> [Original] You use the `@RequestMapping` annotation to map URLs such as `/appointments` onto an entire class or a particular handler method. Typically the class-level annotation maps a specific request path (or path pattern) onto a form controller, with additional method-level annotations narrowing the primary mapping for a specific HTTP method request method ("GET", "POST", etc.) or an HTTP request parameter condition.

你可以使用`@RequestMapping`注解来将请求的URL（比如`/appointments`之类的）路径映射到整个类或某个特定的处理方法上去。一般来说，类级别的注解负责将一个特定的请求路径（或者一个符合某种模式的请求路径）映射到一个固定的控制器上，同时通过方法级别的注解来细化映射，即根据HTTP方法的请求方式（“GET”“POST”方法等）或者HTTP请求中携带的参数特征来将请求映射到匹配的方法上。

> [Original] The following example from the Petcare sample shows a controller in a Spring MVC application that uses this annotation:

下面这段代码示例来自Petcare，它展示了在Spring MVC的应用中如何在控制器上使用这个注解：

```java
@Controller
@RequestMapping("/appointments")
public class AppointmentsController {

    private final AppointmentBook appointmentBook;

    @Autowired
    public AppointmentsController(AppointmentBook appointmentBook) {
        this.appointmentBook = appointmentBook;
    }

    @RequestMapping(method = RequestMethod.GET)
    public Map<String, Appointment> get() {
        return appointmentBook.getAppointmentsForToday();
    }

    @RequestMapping(path = "/{day}", method = RequestMethod.GET)
    public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date day, Model model) {
        return appointmentBook.getAppointmentsForDay(day);
    }

    @RequestMapping(path = "/new", method = RequestMethod.GET)
    public AppointmentForm getNewForm() {
        return new AppointmentForm();
    }

    @RequestMapping(method = RequestMethod.POST)
    public String add(@Valid AppointmentForm appointment, BindingResult result) {
        if (result.hasErrors()) {
            return "appointments/new";
        }
        appointmentBook.addAppointment(appointment);
        return "redirect:/appointments";
    }
}
```

> [Original] In the example, the `@RequestMapping` is used in a number of places. The first usage is on the type (class) level, which indicates that all handling methods on this controller are relative to the `/appointments` path. The `get()` method has a further `@RequestMapping` refinement: it only accepts GET requests, meaning that an HTTP GET for `/appointments` invokes this method. The `add()` has a similar refinement, and the `getNewForm()` combines the definition of HTTP method and path into one, so that GET requests for `appointments/new` are handled by that method.

在上面的示例中有许多地方都使用到了`@RequestMapping`注解。第一次出现是作用于类型（类）级别的，指示了该控制器下的所有处理方法都将以`/appointments`开头。`get()`方法上的`@RequestMapping`注解将其接受的请求进行了进一步的细化：它只接收GET方法。这样，一个请求路径为`/appointments`的HTTP GET请求将会最终调用到这个方法。`add()`方法也做了类似的细化，而`getNewForm()`方法则同时注解了能够接受的请求的HTTP方法和路径。在这种情况下，一个路径为`appointments/new`的GET请求将会被这个方法所处理。

> [Original] The `getForDay()` method shows another usage of `@RequestMapping`: URI templates. [(See the next section)](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates).

`getForDay()`方法则展示了`@RequestMapping`注解的另一个作用：URI模板。（这个内容请[见下节](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates)）

> [Original] A `@RequestMapping` on the class level is not required. Without it, all paths are simply absolute, and not relative. The following example from the PetClinic sample application shows a multi-action controller using `@RequestMapping`:

类级别的`@RequestMapping`注解并不是必须的。没有指定它的话则所有的路径都是绝对路径，而非相对路径。以下的代码示例来自PetClinic，它展示了一个具备多个处理方法的控制器：

```java
@Controller
public class ClinicController {

    private final Clinic clinic;

    @Autowired
    public ClinicController(Clinic clinic) {
        this.clinic = clinic;
    }

    @RequestMapping("/")
    public void welcomeHandler() {
    }

    @RequestMapping("/vets")
    public ModelMap vetsHandler() {
        return new ModelMap(this.clinic.getVets());
    }

}
```

> [Original] The above example does not specify GET vs. PUT, POST, and so forth, because `@RequestMapping` maps all HTTP methods by default. Use `@RequestMapping(method=GET)` to narrow the mapping.

以上代码并不区分请求是GET方法还是PUT/POST或其他方法，因此`@RequestMapping`注解默认是映射所有的HTTP请求方法的。如果想要指定相应的请求方法类型，请使用`@RequestMapping(method=GET)`来缩小范围。

> [Original] ## @Controller and AOP Proxying

## @Controller和面向切面（AOP）代理

> [Original] In some cases a controller may need to be decorated with an AOP proxy at runtime. One example is if you choose to have `@Transactional` annotations directly on the controller. When this is the case, for controllers specifically, we recommend using class-based proxying. This is typically the default choice with controllers. However if a controller must implement an interface that is not a Spring Context callback (e.g. `InitializingBean`, `*Aware`, etc), you may need to explicitly configure class-based proxying. For example with `<tx:annotation-driven/>`, change to `<tx:annotation-driven proxy-target-class="true"/>`.

在某些情形下，一个控制器可能需要在运行时配置AOP代理来做一些工作。一个很好的例子就是当你直接使用`@Transactional`注解来标注一个控制器的时候。如果——更多的时候是指控制器——有这样的需要，我们推荐使用类级别的代理方式。这是代理控制器的默认做法，但如果控制器同时必须实现另一些不支持Spring Context回调（比如`InitializingBean`, `*Aware`等）的接口，可能就需要你手动地去配置类级别的代理了。比如，配置文件可能需要由`<tx:annotation-driven/>`改为`<tx:annotation-driven proxy-target-class="true"/>`。

> [Original] ## New Support Classes for @RequestMapping methods in Spring MVC 3.1

## Spring MVC 3.1中新增支持@RequestMapping的一些类

> [Original] Spring 3.1 introduced a new set of support classes for `@RequestMapping` methods called `RequestMappingHandlerMapping` and `RequestMappingHandlerAdapter` respectively. They are recommended for use and even required to take advantage of new features in Spring MVC 3.1 and going forward. The new support classes are enabled by default by the MVC namespace and the MVC Java config but must be configured explicitly if using neither. This section describes a few important differences between the old and the new support classes.

Spring 3.1中新增了一组支持`@RequestMapping`注解的类，分别是`RequestMappingHandlerMapping`和`RequestMappingHandlerAdapter`。我们推荐你使用它们，有些Spring MVC 3.1之后的版本才新增的特性，这几个注解甚至是必须的。不管是通过MVC的命名空间还是通过MVC Java编程的方式来配置，这组新增的类及其功能默认是开启的，但若你哪种方式都不想用，则必须通过手动去配置。本小节将简要描述新支持的类与旧的哪些有什么主要的不同。

> [Original] Prior to Spring 3.1, type and method-level request mappings were examined in two separate stages — a controller was selected first by the `DefaultAnnotationHandlerMapping` and the actual method to invoke was narrowed down second by the `AnnotationMethodHandlerAdapter`.

在Spring的3.1版本之前，类级别和方法级别的请求映射是分别在两个阶段中完成的——首先`DefaultAnnotationHanlderMapping`会先选中一个控制器，然后再通过`AnnotationMethodHandlerAdapter`把请求定位到具体要调用的那个方法上。

> [Original] With the new support classes in Spring 3.1, the `RequestMappingHandlerMapping` is the only place where a decision is made about which method should process the request. Think of controller methods as a collection of unique endpoints with mappings for each method derived from type and method-level `@RequestMapping` information.

现在有了Spring 3.1后引入的这组新类，`RequestMappingHandlerMapping`成为了这两个决策实际发生的唯一一个地方。你可以把控制器中的一系列处理方法当成是一系列独立的服务节点，每个从类级别和方法级别的`@RequestMapping`注解中获取到足够请求路径映射信息。

> [Original] This enables some new possibilities. For once a `HandlerInterceptor` or a `HandlerExceptionResolver` can now expect the Object-based handler to be a `HandlerMethod`, which allows them to examine the exact method, its parameters and associated annotations. The processing for a URL no longer needs to be split across different controllers.

这种新的处理方式带来了新的可能性。之前的`HandlerInterceptor`或`HandlerExceptionResolver`现在可以确定拿到的这个处理器肯定是一个`HandlerMethod`类型，因此它能够精确地了解这个方法的所有信息，包括它的参数、应用于其上的注解等。这样，内部对于一个URL的处理流程再也不需要分隔到不同的控制器里面去执行了。

> [Original] There are also several things no longer possible:
> [Original] * Select a controller first with a `SimpleUrlHandlerMapping` or `BeanNameUrlHandlerMapping` and then narrow the method based on `@RequestMapping` annotations.
> [Original] * Rely on method names as a fall-back mechanism to disambiguate between two `@RequestMapping` methods that don’t have an explicit path mapping URL path but otherwise match equally, e.g. by HTTP method. In the new support classes `@RequestMapping` methods have to be mapped uniquely.
> [Original] * Have a single default method (without an explicit path mapping) with which requests are processed if no other controller method matches more concretely. In the new support classes if a matching method is not found a 404 error is raised.

同时，也有其他的一些变化，比如有些事情就没法这么玩儿了：
* 先通过`SimpleUrlHandlerMapping`或`BeanNameUrlHandlerMapping`来拿到负责处理请求的控制器，然后通过`@RequestMapping`注解配置的信息来定位到具体的处理方法；
* 依靠方法名称来作为选择处理方法的标准。比如说，两个注解了`@RequestMapping`的方法除了方法名称拥有完全相同的URL映射和HTTP请求方法。在新版本下，`@RequestMapping`注解的方法必须具有唯一的请求映射；
* 定义一个默认方法（即没有声明路径映射），在请求路径无法被映射到控制器下更精确的方法上去时，为该请求提供默认处理。在新版本中，如果无法为一个请求找到合适的处理方法，那么一个404错误将被抛出；

> [Original] The above features are still supported with the existing support classes. However to take advantage of new Spring MVC 3.1 features you’ll need to use the new support classes.

如果使用原来的类，以上的功能还是可以做到。但是，如果要享受Spring MVC 3.1版本带来的方便特性，你就需要去使用新的类。

> [Original] ## URI Template Patterns

## URI模板

> [Original] URI templates can be used for convenient access to selected parts of a URL in a `@RequestMapping` method.

URI模板可以为快速访问`@RequestMapping`中指定的URL的一个特定的部分提供很大的便利。

> [Original] A URI Template is a URI-like string, containing one or more variable names. When you substitute values for these variables, the template becomes a URI. The proposed RFC for URI Templates defines how a URI is parameterized. For example, the URI Template `http://www.example.com/users/{userId}` contains the variable userId. Assigning the value fred to the variable yields `http://www.example.com/users/fred`.

URI模板是一个类似于URI的字符串，只不过其中包含了一个或多个的变量名。当你使用实际的值去填充这些变量名的时候，模板就退化成了一个URI。在URI模板的RFC提议中定义了一个URI是如何进行参数化的。比如说，一个这个URI模板`http://www.example.com/users/{userId}`就包含了一个变量名_userId_。将值_fred_赋给这个变量名后，它就变成了一个URI：`http://www.example.com/users/fred`。

> [Original] In Spring MVC you can use the `@PathVariable` annotation on a method argument to bind it to the value of a URI template variable:

在Spring MVC中你可以在方法参数上使用`@PathVariable`注解，将其与URI模板中的参数绑定起来：

```java
@RequestMapping(path="/owners/{ownerId}", method=RequestMethod.GET)
public String findOwner(@PathVariable String ownerId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    model.addAttribute("owner", owner);
    return "displayOwner";
}
```

> [Original] The URI Template "`/owners/{ownerId}`" specifies the variable name `ownerId`. When the controller handles this request, the value of `ownerId` is set to the value found in the appropriate part of the URI. For example, when a request comes in for `/owners/fred`, the value of `ownerId` is `fred`.

URI模板"`/owners/{ownerId}`"指定了一个变量，名为`ownerId`。当控制器处理这个请求的时候，`ownerId`的值就会被URI模板中对应部分的值所填充。比如说，如果请求的URI是`/owners/fred`，此时变量`ownerId`的值就是`fred`.
`

> 为了处理`@PathVariables`注解，Spring MVC必须通过变量名来找到URI模板中相对应的变量。你可以在注解中直接声明：
> ```java
> @RequestMapping(path="/owners/{ownerId}}", method=RequestMethod.GET)
> public String findOwner(@PathVariable("ownerID") String theOwner, Model model) {
>     // 具体的方法代码…
> }
> ```
> 
> 或者，如果URI模板中的变量名与方法的参数名是相同的，则你可以不必再指定一次。只要你在编译的时候留下debug信息，Spring MVC就可以自动匹配URL模板中与方法参数名相同的变量名。
> ```java
> @RequestMapping(path="/owners/{ownerId}", method=RequestMethod.GET)
> public String findOwner(@PathVariable String ownerId, Model model) {
>     // 具体的方法代码…
> }
> ```

> [Original] A method can have any number of `@PathVariable` annotations:

一个方法可以拥有任意数量的`@PathVariable`注解：

```java
@RequestMapping(path="/owners/{ownerId}/pets/{petId}", method=RequestMethod.GET)
public String findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    Pet pet = owner.getPet(petId);
    model.addAttribute("pet", pet);
    return "displayPet";
}
```

> [Original] When a `@PathVariable` annotation is used on a `Map<String, String>` argument, the map is populated with all URI template variables.

当`@PathVariable`注解被应用于`Map<String, String>`类型的参数上时，框架会使用所有URI模板变量来填充这个map。

> [Original] A URI template can be assembled from type and path level _@RequestMapping_ annotations. As a result the `findPet()` method can be invoked with a URL such as `/owners/42/pets/21`.

URI模板可以从类级别和方法级别的 _@RequestMapping_ 注解获取数据。因此，像这样的`findPet()`方法可以被类似于`/owners/42/pets/21`这样的URL路由并调用到：

```java
_@Controller_
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

    @RequestMapping("/pets/{petId}")
    public void findPet(_@PathVariable_ String ownerId, _@PathVariable_ String petId, Model model) {
        // 方法实现体这里忽略
    }

}
```

> [Original] A `@PathVariable` argument can be of _any simple type_ such as int, long, Date, etc. Spring automatically converts to the appropriate type or throws a
`TypeMismatchException` if it fails to do so. You can also register support
for parsing additional data types. See [the section called "Method Parameters
And Type Conversion"](mvc.html#mvc-ann-typeconversion "Method Parameters And
Type Conversion" ) and [the section called "Customizing WebDataBinder
initialization"](mvc.html#mvc-ann-webdatabinder "Customizing WebDataBinder
initialization" ).

`@PathVariable`可以被应用于所有 _简单类型_ 的参数上，比如int、long、Date等类型。Spring会自动地帮你把参数转化成合适的类型，如果转换失败，就抛出一个`TypeMismatchException`。如果你需要处理其他数据类型的转换，也可以注册自己的类。若需要更详细的信息可以参考[“方法参数与类型转换”一节](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-typeconversion "Method Parameters And Type Conversion")和[“定制WebDataBinder初始化过程”一节](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder "Customizing WebDataBinder initialization")

## 带正则表达式的URI模板

> [Original] Sometimes you need more precision in defining URI template variables. Consider the URL `"/spring-web/spring-web-3.0.5.jar"`. How do you break it down into
multiple parts?

有时候你可能需要更准确地描述一个URI模板的变量，比如说这个URL：`"/spring-web/spring-web-3.0.5.jar`。你要怎么把它分解成几个有意义的部分呢？

> [Original] The `@RequestMapping` annotation supports the use of regular expressions in URI template variables. The syntax is `{varName:regex}` where the first part defines the variable name and the second - the regular expression.For example:

`@RequestMapping`注解支持你在URI模板变量中使用正则表达式。语法是`{varName:regex}`，其中第一部分定义了变量名，第二部分就是你所要应用的正则表达式。比如下面的代码样例：

```java
@RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{extension:\\.[a-z]+}")
    public void handle(@PathVariable String version, @PathVariable String extension) {
        // 代码部分省略...
    }
}
```

## Path Patterns（不好翻，容易掉韵味）

> [Original] In addition to URI templates, the `@RequestMapping` annotation also supports Ant-style path patterns (for example, `/myPath/*.do`). A combination of URI
template variables and Ant-style globs is also supported (e.g. `/owners/*/pets/{petId}`).

除了URI模板外，`@RequestMapping`注解还支持Ant风格的路径模式（如`/myPath/*.do`等）。不仅如此，还可以把URI模板变量和Ant风格的glob组合起来使用（比如`/owners/*/pets/{petId}`这样的用法等）。

## 路径样式的匹配(Path Pattern Comparison)

> [Original] When a URL matches multiple patterns, a sort is used to find the most specific match.

当一个URL同时匹配多个模板（pattern）时，我们将需要一个算法来决定其中最匹配的一个。

> [Original] A pattern with a lower count of URI variables and wild cards is considered more specific. For example `/hotels/{hotel}/*` has 1 URI variable and 1 wild card and is considered more specific than `/hotels/{hotel}/**` which as 1 URI
variable and 2 wild cards.

URI模板变量的数目和通配符数量的总和最少的那个路径模板更准确。举个例子，`/hotels/{hotel}/*`这个路径拥有一个URI变量和一个通配符，而`/hotels/{hotel}/**`这个路径则拥有一个URI变量和两个通配符，因此，我们认为前者是更准确的路径模板。

> [Original] If two patterns have the same count, the one that is longer is considered more specific. For example `/foo/bar*` is longer and considered more specific than `/foo/*`.

如果两个模板的URI模板数量和通配符数量总和一致，则路径更长的那个模板更准确。举个例子，`/foo/bar*`就被认为比`/foo/*`更准确，因为前者的路径更长。

> [Original] When two patterns have the same count and length, the pattern with fewer wild cards is considered more specific. For example `/hotels/{hotel}` is more
specific than `/hotels/*`.

如果两个模板的数量和长度均一致，则那个具有更少通配符的模板是更加准确的。比如，`/hotels/{hotel}`就比`/hotels/*`更精确。

> [Original] There are also some additional special rules:

除此之外，还有一些其他的规则：

> [Original] * The **default mapping pattern** `/**` is less specific than any other pattern. For example `/api/{a}/{b}/{c}` is more specific.
> 
> [Original] * A **prefix pattern** such as `/public/**` is less specific than any other pattern that doesn't contain double wildcards. For example `/public/path3/{a}/{b}/{c}` is more specific.

* __默认的通配模式__`/**`比其他所有的模式都更“不准确”。比方说，`/api/{a}/{b}/{c}`就比默认的通配模式`/**`要更准确
* __前缀通配__（比如`/public/**`)被认为比其他任何不包括双通配符的模式更不准确。比如说，`/public/path3/{a}/{b}/{c}`就比`/public/**`更准确

> [Original] For the full details see `AntPatternComparator` in `AntPathMatcher`. Note that the PathMatcher can be customized (see [Section 21.16.11, "Path
Matching"](mvc.html#mvc-config-path-matching "21.16.11 Path Matching" ) in the
section on configuring Spring MVC).

更多的细节请参考这两个类：`AntPatternComparator`和`AntPathMatcher`。值得一提的是，PathMatcher类是可以配置的（见“配置Spring MVC”一节中的[21.16.11 路径的匹配](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/html#mvc-config-path-matching "21.16.11 Path Matching")一节)。


## 带占位符的路径模式（path patterns）

> [Original] Patterns in `@RequestMapping` annotations support ${…} placeholders against local properties and/or system properties and environment variables. This may be useful in cases where the path a controller is mapped to may need to be
customized through configuration. For more information on placeholders, see
the javadocs of the `PropertyPlaceholderConfigurer` class.

`@RequestMapping`注解支持在路径中使用占位符，以取得一些本地配置、系统配置、环境变量等。这个特性有时很有用，比如说控制器的映射路径需要通过配置来定制的场景。如果想了解更多关于占位符的信息，可以参考`PropertyPlaceholderConfigurer`这个类的文档。


#### Suffix Pattern Matching
## 后缀模式匹配

> [Original] By default Spring MVC performs `".*"` suffix pattern matching so that a
controller mapped to `/person` is also implicitly mapped to `/person.*`. This
makes it easy to request different representations of a resource through the
URL path (e.g. `/person.pdf`, `/person.xml`).

Spring MVC默认采用`".*"`的后缀模式匹配来进行路径匹配，因此，一个映射到`/person`路径的控制器也会隐式地被映射到`/person.*`。这使得通过URL来请求同一资源文件的不同格式变得更简单（比如`/person.pdf`，`/person.xml`）。

> [Original] Suffix pattern matching can be turned off or restricted to a set of path extensions explicitly registered for content negotiation purposes. This is
generally recommended to minimize ambiguity with common request mappings such
as `/person/{id}` where a dot might not represent a file extension, e.g.
`/person/joe@email.com` vs `/person/joe@email.com.json)`. Furthermore as explained in the note below suffix pattern matching as well as content negotiation may be used
in some circumstances to attempt malicious attacks and there are good reasons
to restrict them meaningfully.

你可以关闭默认的后缀模式匹配，或者显式地将路径后缀限定到一些特定格式上for content negotiation purpose。我们推荐这样做，这样可以减少映射请求时可以带来的一些二义性，比如请求以下路径`/person/{id}`时，路径中的点号后面带的可能不是描述内容格式，比如`/person/joe@email.com` vs `/person/joe@email.com.json`。而且正如下面马上要提到的，后缀模式通配以及内容协商有时可能会被黑客用来进行攻击，因此，对后缀通配进行有意义的限定是有好处的。

> [Original] See [Section 21.16.11, "Path Matching"](mvc.html#mvc-config-path-matching "21.16.11 Path Matching") for suffix pattern matching configuration and also [Section 21.16.6, "Content Negotiation"](mvc.html#mvc-config-content-negotiation "21.16.6 Content Negotiation") for content negotiation configuration.

关于后缀模式匹配的配置问题，可以参考[第21.16.11小节 "路径匹配"](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/mvc.html#mvc-config-path-matching "21.16.11 Path Matching")；关于内容协商的配置问题，可以参考[第21.16.6小节 "内容协商"](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/mvc.html#mvc-config-content-negotiation "21.16.6 Content Negotiation")的内容。


## 后缀模式匹配与RFD

> [Original] Reflected file download (RFD) attack was first described in a [paper by
Trustwave](https://www.trustwave.com/Resources/SpiderLabs-Blog/Reflected-File-Download---A-New-Web-Attack-Vector/) in 2014. The attack is similar to XSS in that it relies on input (e.g. query parameter, URI variable) being reflected in the response. However instead of inserting JavaScript into HTML, an RFD attack relies on the browser switching to perform a download and treating the response as an executable script if double-clicked based on the file extension (e.g. .bat, .cmd).

RFD(Reflected file download)攻击最先是2014年在[Trustwave的一篇论文](https://www.trustwave.com/Resources/SpiderLabs-Blog/Reflected-File-Download---A-New-Web-Attack-Vector/)中被提出的。它与XSS攻击有些相似，因为这种攻击方式也依赖于某些特征，即需要你的输入（比如查询参数，URI变量等）等也在输出（response）中以某种形式出现。不同的是，RFD攻击并不是通过在HTML中写入JavaScript代码进行，而是依赖于浏览器来跳转到下载页面，并把特定格式（比如.bat，.cmd等）的response当成是可执行脚本，双击它就会执行。

> [Original] In Spring MVC `@ResponseBody` and `ResponseEntity` methods are at risk because they can render different content types which clients can request including via URL path extensions. Note however that neither disabling suffix pattern matching nor disabling the use of path extensions for content negotiation purposes alone are effective at preventing RFD attacks.

Spring MVC的`@ResponseBody`和`ResponseEntity`方法是有风险的，因为它们会根据客户的请求——包括URL的路径后缀，来渲染不同的内容类型。因此，禁用后缀模式匹配或者禁用仅为内容协商开启的路径文件后缀名携带，都是防范RFD攻击的有效方式。

> [Original] For comprehensive protection against RFD, prior to rendering the response body Spring MVC adds a `Content-Disposition:inline;filename=f.txt` header to suggest a fixed and safe download file filename. This is done only if the URL path contains a file extension that is neither whitelisted nor explicitly registered for content negotiation purposes. However it may potentially have side effects when URLs are typed directly into a browser.

> [Original] Many common path extensions are whitelisted by default. Furthermore REST API calls are typically not meant to be used as URLs directly in browsers. Nevertheless applications that use custom `HttpMessageConverter` implementations can explicitly register file extensions for content negotiation and the Content-Disposition header will not be added for such extensions. See [Section 21.16.6, "Content Negotiation"](mvc.html#mvc-config-content-negotiation "21.16.6 Content Negotiation" ).


> [Original] 













