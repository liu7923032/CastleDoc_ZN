# Castle Windsor

## 初始化

```
    //非常简单
    var container=new WindsorContainer();
```
## 注册组件

1. 注册多个服务

```
    //注册 IService 接口服务,它的具体实现类是Service
    container.Register(
        Component.For<IService>().ImplementedBy<Service>(),
        Component.For<IRepository>().ImplementedBy<IUserRepository>()
    );
   
```

2. 注册一个服务多个实例

```
    // 如果一个服务注册多个实例的情况,那么默认实现第一个
    container.Register(
        Component.For<IService>().ImplementedBy<MyService>(),
        Component.For<IService>().ImplementedBy<OtherService>(),
        //IsDefault() 方法指定默认实现实例
        Component.For<IService>().ImplementedBy<ThreeService>().IsDefault(),
    );
```

3. 注册泛型服务,需要用到For(typeof(IInterface<>))

```
    container.Register(
        Component.For(typeof(IRepository<>)).ImplementedBy(typeof(EFRepository<>))
    );
```

4. 设定服务周期
    
    * 默认的实现的是单例模式,如果实现每次都生成新的实例,需要用到 LifeStye.Transient
```
    container.Register(
        Component.For<IService>()
            .ImplementedBy<Service>()
            .LifeStyle.Transient
    );

```

5. 注册现有实例(ps:这个不知道用在哪里)

```
    ISerivce instance=new Service();
    //此时设定的生命周期是无效的
    container.Register(
        Component.For<IService>().Instance(instance)
    )
```

6. 给实例设定名称

```
    //实现动态的获取对象实例
    container.Register(
        Component.For<IService>().ImplementedBy<MyService>()
        .Named("service.myservice"),
        Component.For<IService>().ImplementedBy<OtherService>()
        .Named("service.otherservice"),
    )
```

7. 使用委托注册服务

```
    //将实例交给其他方法,动态生成实例
    container.Register(
        Component.For<IService>()
            .UsingFactoryMethod(()=>xxx.CreateMethod())
    );

    //重载方法
    container.Register(
        Component.For<IService>()
            .UsingFactoryMethod((kernel)=>kernel.Resolve<IService>().Create())
    );
```
8. 注册多服务到同一实例

```
    //一个实例由实现了多个接口时候使用
    container.Register(
        Component.For<IService,IExtendService>().ImplementedBy<Service>()
    )
    //上面最多只能使用四个接口,假如有超过四个接口使用 `ForWord`
    container.Register(
        Component.For<IUserRepository>()
            .ForWard<IRepository,IRepository<User>>()
            .ImplementedBy<MyRepository>()
    );
```

9. 服务重写

如下例子,解决在特定的类中,想要实例特定的具体实现
```
    //1：默认实例MyService
    container.Register(
        //默认的实现
        Component.For<IService>()
            .ImplementedBy<MyService>(),
        Component.For<IService>()
            .ImplementedBy<OtherService>(),
        
    )
    //2：想要要实现OtherServce
    class ProductController{
       
        private IService _service;

        public ProductController(IService service){
            this._service=service;
        }
    }

    //3：
    container.Register(
        //默认的实现
        Component.For<IService>()
            .ImplementedBy<MyService>()
            .Named("myservice"),
        Component.For<IService>()
            .ImplementedBy<OtherService>()
            .Named("otherservice"),
        Component.For<ProductController>()
            .ServiceOverrides(
                ServiceOverride.ForKey("Service").Eq("otherservice")
            )
    )

```

## 注册程序集(精华)

* 在一个程序里面,接口和服务可能会有几百个甚至更多,如果程序员每添加一个类都要设置一下,那估计要😭了,所以此处程序集注册出现了.

## 通过 `Classes` 或者 `Types` 注册 `Assembly`

### `Classes`注册程序集(只需要三步)

* 扫描当前程序集 `Classes.FromThisAssembly()`
* 注入服务  `WithService.DefaultInterfaces()`
* 设置生命周期 `LifesyleTransient()`

```
    container.Register(
        Classes.FromThisAssembly()
            .WithService.DefaultInterfaces()
            .LifesyleTransient()
    );
```
### 程序集的过滤

* BaseOn,Where,Pick (可以多重过滤) 

```
    //在当前程序集中只选择集成自 `IController` 接口的类,
    container.Register(
        Classes.FromThisAssembly()
        .BaseOn<IController>()
        .Where(Component.IsInNamespace("Acme.Crm.MessagerDTOS"))
    );
```

### 为组件选择服务

1. `WithService.Base()`

```
    container.Register(
        Classes.FromThisAssembly()
        .BaseOn<IController>()
        .WithService.Base()
    )
```

2. `WithService.DefaultInterfaces()`

这个方法执行匹配是依据类型名称和接口的名称.

```
    container.Register(
        Classes.FromThisAssembly()
        .InNamespace("Acme.Crm.Services)
        .WithService.DefaultInterfaces()
    );
```

3. `FromInterface()`

另一个非常常见的场景是能够注册共享公共接口的所有类型，但它们之间没有关系。

```
    public interface IService   {}

    public interface ICalculatorService:IService{}

    public class CalculatorService:ICalculatorService{

    }


    //1：普通注册
    container.Register(
        Component.For<ICalculatorService>()
        .ImplementedBy<CalculatorService>()
    )
    //2: FromInterface()
    container.Register(
        Classes.FromThisAssembly()
        .BaseOn<IService>()
        .WithService.FromInterface()
    );
```

4. `AllInterfaces()`

解决多接口的问题,当一个实现实现了多个接口,你想每个接口都注册实例的化,用这个方法

5. `Self()`

注册组件将明确类型作为服务使用`WithService.Self()`

```
    //1. 普通祖册
    container.Register(
        Component.For<Foo>()
        .ImplementedBy<Foo>()
    );

    //2 等价于
    container.Register(
        Classes.FromThisAssembly()
        .WithService.Self()
    );
```

6. `Select()`

如果上面没有一项满足你,那么你可以提供自己的子的委托实现逻辑,通过`WithService.Select()`

```
    //1:
    container.Register(
        Classes.FromThisAssembly()
            .BasedOn<IFoo>()
            .WithService.Self()
            .WithService.Base()
    );
    //2:
    Component.For<IFoo, FooImpl>().ImplementedBy<FooImpl>();
```
7. 注册非公共类型

默认的,容易只会注册可见的类型和服务,如果你想注册包含非公共类型,你需要首先指定程序集后接着使用`IncludeNonpublicTypes`

```
    container.Register(
        Classes.FromThisAssemby()
            .IncludeNoPublicTypes()
            .BaseOn<NoPublicComponent>()
    );
```

8. `Configure`

```
    container.Register(
        Classes.FromAssembly(Assembly.GetExecutingAssembly())
            .BasedOn<ICommon>()
            .LifestyleTransient()
            .Configure(component => component.Named(component.Implementation.FullName + "XYZ"))
    );
```
9. `ConfigureFor<T>`

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


10 `ConfigureIf`

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