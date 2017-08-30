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



