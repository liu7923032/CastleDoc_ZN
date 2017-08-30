翻译[https://github.com/castleproject/Windsor/blob/master/docs/registering-components-by-conventions.md]
# 通过约定注册组件

## 一次注册多种类型

一个一个的注册组件是一个非常重复的工作.当然记住每一个新增加的类型是很令人挫折的.幸运的,在大部分情况下,你不需要那样做,你也不要.通过使用 `Classes` 或者`Types` 入口,基于一些你指定的特定的字符你可以执行一组类型的注册.你会发现在使用Windsor写应用程序时最长使用的方式.

## 三步

注册多类型通常采用一下形式

```
    container.Register(
        Classes.FromThisAssembly()
            .IsSameNamespaceAs<RootComponent>()
            .WithService.DefaultInterfaces()
            .LifestyleTrasnsient()
    );
```

在注册调用过程中识别三个明显的步骤.

## 选择`Assembly`

第一步指示Windsor扫描你要的assembly,你用它做:

```
    Classes.FromThisAssembly()...
```
上面时她姐妹方法之一.

我需要用`Classes`或`Types`?:这有两种方式去开始依照约定注册.一种时使用 `Classes` 静态类,像上面的例子.第二种使用 `Types` 静态类. 他们暴露的相同的方法.他们之间的不同的是,`Types` 会允许你在`assembly`注册所有(或者暴露清晰,如果你使用默认设定,是所有 `public`)类型,其他 Classes,interfaces,structs,delegates和枚举。`Classes` 在其他方面过滤类型只考虑非抽象类.大多数时间你会使用`Classes`,但是摩羯高级场景中,`Types`非常有用,比如基于类型工厂注册接口.

注意 `AllTypes` 是什么?:`Classes`和`Types`在Windsor 3中都是新的.

## 选择 基类型/条件

一旦你选择一个程序集,基本步骤是你要过滤你要注册的类型.这将以以下方式缩小丢向的范围.

1. 通过基类/接口,如下例:
  
  * `Classes.FromThisAssembly().BasedOn<IMessage>()`
2. 通过命名空间,如下例:

  * `Classes.FromAssemblyInDirectory)new AssemblyFilter("bin")).InNamespace("Acme.Crm.Extensions")`

3. 通过条件,如下例:

  * `Classes.FromAssemblyContaining<MyController>().Where(u=>Attribute.IsDefined(t,typeof(CacheAttribute))`

4. 在这一点没有限制
  * `Classes.FromAssemblyNamed("Acme.Crm.Services").Pick()`

## 附加过滤和配置

一旦你选择源经过类型和基础条件你当然也配置类型或者附加过滤。

```
    container.Register(
        Classes.FromThisAssembly()
            .BaseOn<IMessage>()
            .BaseOn(typeof(IMessageHandler<>)).WithService.Base()
            .Where(Component.IsInNamespace("Acme.Crm.MessageDTOs"))
    )
```

### 注册给定类型的子代(像在MVC程序的所有控制器)

这有一个单轨配置的示例:

```
    container.Register(
        Classes.FromThisAssembly()
            .BaseOn<IController>()
            .Configure(u=>u.Lifestyle.Transient)
    );
```

## 默认服务

记住其他筛选只是只是缩小了我们要注册的程序集，它们没有指定服务,会使用缺省值.

## 给组件选择服务

默认的服务是组件类型本身，有几种情况是不够的.Windsor需要你指定清晰的类型.

### Base()
```
    container.Register(
        Classes.FromThisAssembly()
            .BasedOn(typeof(ICommand<>)).WithService.Base(),
        Classes.FromThisAssembly()
            .BasedOn(typeof(IValidator<>)).WithService.Base()
    )
```

### DefaultInterfaces()

这个方法在Windsor 3中改名为从`DefaultInterface`,他强调可以匹配更多的不仅仅是一个组件.

```
    container.Register(
        Classes.FromThisAssembly()
            .InNamespace("Acme.Crm.Services")
            .WithService.DefaultInterfaces()
    );
```

这个方法执行匹配依据类型名称和接口名称.通常你会发现你接口/具体的匹配像这样:`ICustomerRepository/CustomerRepository`, `IMessageSender/SmsMessageSender`, `INotificationService/DefaultNotificationService`,这种状况是你可能想要使用DefaultInterfaces方法去匹配服务.它将查看选定类型实现的所有接口，并将其用作具有匹配名称的类型的服务。匹配名称，意味着实现类在其名称中包含接口的名称（没有前面的i）。

### FromInterface()

另一个非常常见的场景是能够注册共享公共接口的所有类型，但它们之间没有关系。

```
    container.Register(
        Classes.FromThisAssembly()
            .BasedOn<IService>()
            .WithService.FromInterface()
    );
```

这里我们从正在执行的程序集注册服务类。在这种情况下，IService接口可能是一个标记接口识别系统中的一个组成部分的作用。不同的服务。不像```WithService.Base()```，选定此注册服务类型是从接口，扩展服务选择。下面是一个例子来帮助说明这一点。
下面是一个例子来帮助说明这一点。

可以说你有一个标记接口服务在您指定的所有服务组件。

```
    public interface IService {}

    public interface ICalculatorService : IService
    {
        float Add(float op1, float op2);
    }

    public class CalculatorService : ICalculatorService
    {
        public float Add(float op1, float op2)
        {
            return op1 + op2;
        }
    }
```
上面的例子等同于

```
    container.Register(
        Component.For<ICalculatorService>()
            .ImplementedBy<CalculatorService>()
    );
```
你可以看到，实际的服务接口是不是IService，而是扩展接口IService，这是在这种情况下ICalculatorService.

### AllInterfaces()

当一个组件实现多个接口并且你想他们都实现它,用 `WidthService.AllInterfaces()方法.

### Self()

注册组件实现类型明确作为服务使用self()服务。

### Select()

如果以上选项不适合您可以提供您自己的选择逻辑为代表，通过它的委托可以通过Select()方法

服务可是累积:可以多次使用WidthService.Something()并且可以累加,意味着:
```
    container.Register(
        Classes.FromThisAssembly()
            .BasedOn<IFoo>()
            .WithService.Self()
            .WithService.Base()
    );
```

上面的写法等同于：

```
    Component.For<IFoo, FooImpl>().ImplementedBy<FooImpl>();
```

## 注册非公共类型

默认情况下，只能从程序集外部可见的类型将被注册。如果你想包括非公有制类型，你必须从指定的组件，然后调用IncludeNonPublicTypes

```
    container.Register(
        Classes.FromThisAssembly()
            .IncludeNonPublicTypes()
            .BasedOn<NonPublicComponent>()
    );
```

注意:尽量不要用该方法.

## 配置注册

### Configure 方法

注册类型时，还可以配置它们来设置与注册类型一个接一个时的所有相同属性。为此，您使用配置方法。最常见的情况是为您的组件分配一种生活方式，而不是默认的单例。

```
    container.Register(
        Classes.FromAssembly(Assembly.GetExecutingAssembly())
            .BasedOn<ICommon>()
            .Configure(component => component.LifestyleTransient())
    );

    //windsor 3 above
    container.Register(
        Classes.FromAssembly(Assembly.GetExecutingAssembly())
            .BasedOn<ICommon>()
            .LifestyleTransient()
    );
```

除了指定生活方式之外，还可以设置许多其他配置选项：

```
    container.Register(
        Classes.FromAssembly(Assembly.GetExecutingAssembly())
            .BasedOn<ICommon>()
            .LifestyleTransient()
            .Configure(component => component.Named(component.Implementation.FullName + "XYZ"))
    );
```

在我们这里注册了实现ICommon，设置了他的生命周期和名称

### ConfigureFor<T> method

```
    container.Register(
        Classes.FromThisAssembly()
            .BasedOn<ICommon>()
            .LifestyleTransient()
            .Configure(
                component => component.Named(component.Implementation.FullName + "XYZ")
            )
            .ConfigureFor<CommonImpl1>(
                component => component.DependsOn(Property.ForKey("key1").Eq(1))
            )
            .ConfigureFor<CommonImpl2>(
                component => component.DependsOn(Property.ForKey("key2").Eq(2))
            )
    );
```

### ConfigureIf method

```
    container.Register(
        Classes.FromThisAssembly()
            .BasedOn<ICommon>()
            .LifestyleTransient()
            .Configure(
                component => component.Named(component.Implementation.FullName + "XYZ")
            )
            .ConfigureIf(
                x => x.Implementation.Name.StarsWith("Common"),
                component => component.DependsOn(Property.ForKey("key1").Eq(1))
            )
    );
```

