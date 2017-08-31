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
    )
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

### `Classes`注册程序集(只需要三部)

* 扫描当前程序集 `Classes.FromThisAssembly()`
* 设置服务  `WithService.DefaultInterfaces()`
* 实例生命周期 `LifesyleTransient()`

```
    container.Register(
        Classes.FromThisAssembly()
            .WithService.DefaultInterfaces()
            .LifesyleTransient()
    )
```
### 程序集的过滤

* BaseOn<>() (可以多重过滤)

```
    //在当前程序集中只选择集成自 `IController` 接口的类,
    container.Register(
        Classes.FromThisAssembly()
        .BaseOn<IController>()
        .BaseOn<IService>()
    )
```

### 实现服务

1. WithServie.Base()
