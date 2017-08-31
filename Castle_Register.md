# IOC 学习

    准备学习ABP框架,看源码的时候,发现代码中到处都用到依赖注入(DI)技术,我原来的项目由于使用的都是Autofac,所以准备花点时间来系统的学习 Castle 框架使用

译文[https://github.com/castleproject/Windsor/blob/master/docs/registering-components-one-by-one.md#type-forwarding]

## Castle 使用

1. 安装Castle Windsor

```csharp
    Install Package Castle-Windsor
```
2. 初始化Windsor Container

```csharp
    var container=new Windsor();
```

# 注册组件

## 在 container 中注册类型
```
    container.Register(
        Component.For(MyServiceImpl)()
    );
```
这会注册类型 ```MyServiceImpl``` 作为服务```MyServiceImpl```,它拥有默认的生命周期(Singleton)

## 注册一个非默认类型的服务
```
    container.Register(
        Component.For(IMyService)()
            .ImplementedBy<MyService>()
    );
```

## 注册一个泛型

假如你有一个 ```IRepository<T>```接口,它有一个实现```EFRepository<T>```.
你可以注册每一个实体的仓储,但是它不是非必需的

```
    // 注册一个每一个实体的仓储不是必须的
    container.Register(
        Component.For<IRepository<User>>()
            .ImplementedBy<EFRepository<User>>(),
        Component.For<IRepository<Post>>()
            .ImplementedBy<EFRepository<Post>>()
    );
```
上面的代码实际上有很多重复,其实只需要注册一个```IRepository<>```就可以了
```
    // 这种不会工作（编译不容许）
    container.Register(
        Component.For<IRepository<>>()
            .ImplementedBy<EFRepository<>>()
    );
```
像上面那种不合法,并且上面的代码无法编译,相反你需要用到 ```typeof()```

```
    // 用 typeof() 不需要指定具体的实体
    container.Register(
        Component.For(typeof(IRepository<>))
            .ImplementedBy(typeof(EFRepository<>))
    );
```
当 ```lifeStyle```没有设置清晰，他会使用默认的 Singleton lifeStyle 

## 配置组件的 lifeStyle

```
    container.Register(
        Component.For<IMyService>()
            .ImplementedBy<MyService>()
            .LifeStyle.Transient
    );
```
当 ```lifeStyle``` 没有设置明确,组件会使用默认的 Singleton lifeStye

## 注册一个服务,多个实现

你可以很简单的实现同一个服务多个注册

```
    container.Register(
        Component.For<IMyService>().ImplementedBy<MyService1>(),
        Component.For<IMyService>().ImplementedBy<MyService2>(),
    )
```
当一个服务有多个从属组件的时候,它会默认的获取第一个注册的组件(在上面会解析到 ```MyService1```),这个不同于 Autufac(默认获取最后一个从属组件)

当然你也可以强制将最后一个组件变为默认实例用到一个方法```IsDefault```
```
    container.Register(
        Component.For<IMyService>().ImplementedBy<MyService1>(),
        Component.For<IMyService>().Named("OtherServce").ImplementedBy<MyService2>()
        .IsDefault()
    )
```
上面的例子,任何组件都依赖于IMyservice,它会默认获取```MyService2```实例，即使它是最后注册的。

当然，你可以重写你需要用到的实现。这用到服务重写。

当你明确使用 ```container.Resolve<IMyService>()```(不指定名字),容器会返回第一个组件

为重复的组件提供唯一的名字：如果你想注册多次同一个实现，务必为注册的组件提供不同的名字。

## 注册已经存在的实例

将先用的对象注册为服务是可能的.

```
    var customer=new Customer();
    container.Register(
        Component.For<ICustomer>().Instance(customer)
    );
```

注意：将现有的对象注册到服务,即使你写明liftStyle,也会被忽略

## 使用委托作用组件工厂
你可以使用委托为组件提供一个轻量级工厂:

```
    container.Register(
        Component.For<IMyService>().UsingFactoryMethod(
            ()=>MyLegacyServiceFactory.CreateMyService()
        )
    );
```

```UsingFactoryMethod```方法有两个重载，他可以给你提供访问 kernel,如果你需要可以创建context.

基于Kernel重载<Convert<IKernel,IMyService>> UsingFactoryMethod 用例
```
    container.Register(
        Component.For<IMyFactory>().ImplementedBy<MyFactory>(),
        Component.For<IMyFactory>()
                    .UsingFactoryMethod(kernel=>kernel.Resolve<IMyfacory>().Create())
    );

```

除了```UsingFactoryMethod```方法,还有一个```UsingFactory```(不需要```method```后缀),它能视为特殊版本的```UsingFactoryMethod```方法,用于从container中解析已经存在的的工厂和让你在服务中创建实例。

避免使用```UsingFacotry```：建议使用```UsingFactoryMethod```,在你穿件服务的时候避免```UsingFactory```，```UsingFactory```在将来的发布版本中会被过期/删除.


## OnCreate

有事需要检查或者修改创建的实例，在用之前,你可以使用```OnCreate```方法去处理

```
    container.Register(
        Component.For<IServcie>().ImplementedBy<MyService>()
            .OnCreate(kernel,instance)=>instance.Name+="a")
    );
```
这个方法有两个重载,一个工作配合委托，有两个实例IKernel和新创建的实例,另外一个
只有一个新创建的实例.

```OnCreate```只有工作在container创建组件中:这个方法不会被触发在外部提供实例(像用实例方法时).只有在容器创建组件的时候才会被触发.这当然包含某些工具穿件组件时.

## 给组件指定一个名字

注册组件默认的名字是类型实例类型的全名,你可以用```Named()```方法制定一个不同的名称.
```
    container.Register(
        Component.For<IService>().ImplementedBy<MyServiceImpl>()
                    .Named("mservice.default")
    );
```

## 向组件提供使用依赖(服务重写)

如果一个组件需要或者想要其他组件的功能,这叫依赖.在注册时,使用服务重写你可以明确的设置.

```
    container.Register(
        Component.For<IMyService>()
            .ImplementedBy<MyService>()
            .Named("myservice.default),
        Component.For<IMyService>()
            .ImplementedBy<OtherService>()
            .Named("myservice.other),
        Component.For<ProductController>()
            .ServiceOverrides(
                ServiceOverride.ForKey("myService").Eq("myservice.other")
            )
    );

    public class ProductController
    {
        public ProductController(IMyservice myService){

        }
    }
```

## 使用多个服务注册组件

一个组件有多个服务是有可能的.比如如果你有一个类```FooBar```实现```IFoo```和```IBar```接口,你可以配置你的容器,当```IFoo```和```IBar```需要的时候,返回同样的服务,这种能力叫类型转发.

## 类型转发

使用```Component.For```的泛型重载方法,可以很简单的实现指定类型转发
```
    container.Register(
        Component.For<IUserRepository,IRepository>()
            .ImplementedBy<MyRepository>()
    );
```
上面可以最高重载四种类型转发服务,这个已经够了.如果你发现自己需要更多,你很坑你违反了单一职责原则,你可能需要将巨型组件分解更多的组件,每个组件只做一件事.

上面当然还有一个非泛型重载.

更多,你可以哦那个```ForWard```方法,他会暴露相同行为和重载```For```方法.

```
    container.Register(
        Component.For<IUserRepository>()
            .ForWard<IRepository,IRepository<User>>()
            .ImplementedBy<MyRepository>()
    );
```

