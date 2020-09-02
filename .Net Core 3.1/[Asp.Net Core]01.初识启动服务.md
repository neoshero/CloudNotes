
## 目录
### 启动运行三部曲
* 服务配置
* 服务构建
* 服务托管

我们创建第一个<em>.Net Core</em> 项目,我们会看到工程项目下提供了一个运行的<em>Program</em>类,代码如下

```C#
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args)
        .Build()
        .Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```
Main方法中的三个步骤,CreateHostBuilder、Build、Run它代表运行的三个步骤
从方法名称可以推测三个方法作用
* Host.CreateDefaultBuilder:Host的默认配置信息(IWebHostBuilder)
* Build:创建服务器宿主(IWebHost)
* Run:启动宿主服务
  
##服务配置

首先我们需要关心它到底有哪些配置服务,查询返回的接口是IHostBuilder
我们查看一下接口包含的内容
```C#
public interface IHostBuilder
{
    //构建服务器宿主
    IHost Build();
    //应用服务配置文件设置
    IHostBuilder ConfigureAppConfiguration(Action<HostBuilderContext, IConfigurationBuilder> configureDelegate);
    //宿主主机配置
    IHostBuilder ConfigureHostConfiguration(Action<IConfigurationBuilder> configureDelegate);
    //替换默认DI容器的注入方式 (例如 AutoFac)
    IHostBuilder ConfigureContainer<TContainerBuilder>(Action<HostBuilderContext, TContainerBuilder> configureDelegate);
    //服务注入配置(DI)
    IHostBuilder ConfigureServices(Action<HostBuilderContext, IServiceCollection> configureDelegate);
    //提供服务需要工厂 例如IOC相关的容器
    IHostBuilder UseServiceProviderFactory<TContainerBuilder>(IServiceProviderFactory<TContainerBuilder> factory);
}
```
IHostBuilder 本身为我们了提供了这几种文件的配置信息,我们先不关心具体的操作,只需要了解大概有哪些功能
我们在看一下ConfigureWebHostDefaults这个方法,他的委托对象是IWebHostBuilder
由此我们可以看出,.Net Core 默认为我们创建了一个实现IWebHostBuilder接口的实例
它提供的Api跟IHostBuilder基本类似,只是多了一个UseSetting重要方法,主要用来设置配置文件的信息
它拥有非常丰富的拓展方法
    WebHostBuilderExtensions: 服务与中间件相关的Api 
    HostingAbstractionsWebHostBuilderExtensions: 服务器运行环境相关的Api
我列举出两个拓展类几个重要的方法
```C#
public static class WebHostBuilderExtensions
{
    //配置运行的中间件
    public static IWebHostBuilder Configure(this IWebHostBuilder hostBuilder,Action<IApplicationBuilder> configureApp){}
    //设置配置文件
    public static IWebHostBuilder ConfigureAppConfiguration(this IWebHostBuilder hostBuilder,Action<IConfigurationBuilder> configureDelegate){}
    //配置日志
    public static IWebHostBuilder ConfigureLogging(this IWebHostBuilder hostBuilder, Action<ILoggingBuilder> configureLogging){}
    //提供默认的服务注入容器(DI)
    public static IWebHostBuilder UseDefaultServiceProvider(this IWebHostBuilder hostBuilder, Action<ServiceProviderOptions> configure){}
    //使用分离的配置类
    public static IWebHostBuilder UseStartup<TStartup>(this IWebHostBuilder hostBuilder)｛｝
}

public static class HostingAbstractionsWebHostBuilderExtensions
{
    //设置默认的启动运行路径 默认当前运行项目的路径
    public static IWebHostBuilder UseContentRoot(this IWebHostBuilder hostBuilder, string contentRoot){}
    //设置运行环境 默认Development
    public static IWebHostBuilder UseEnvironment(this IWebHostBuilder hostBuilder, string environment){}
    //设置监听端口的服务器 (IIS、IISExpress、Kestrel)
    public static IWebHostBuilder UseServer(this IWebHostBuilder hostBuilder, IServer server){}
    //设置监听端口号
    public static IWebHostBuilder UseUrls(this IWebHostBuilder hostBuilder, params string[] urls){}
    //设置静态文件的访问路径 默认当前运行项目wwwroot文件夹
    public static IWebHostBuilder UseWebRoot(this IWebHostBuilder hostBuilder, string webRoot){}
}
```
我们在看一下UseStartup<>中Startup这个配置类,看看这个方法里面包含了哪些方法
```C#
public class Startup
{
    //服务注册
    public void ConfigureServices(IServiceCollection services)
    {
        //添加Controller服务,让系统支持Controler访问
        services.AddControllers();
    }

    //这个是Http请求管道,添加需要的中间件
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        //使用路由系统
        app.UseRouting();
        //使用静态文件
        app.UseStaticFiles();
        //路由终结点配置
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapGet("/", async context =>
            {
                await context.Response.WriteAsync("Hello World!");
            });
        });
    }
}
```
这些方法刚好是我们上面IHostBuilder和WebHostBuilderExtensions,说明这个文件不是必须的,我们完全可以在CreateDefaultBuilder上进行配置
将StartUp文件提取出来的理由
* ConfigureServicese跟我们实际的业务有关系,我们会需要注册一些自己的服务,跟不属于服务器配置范围
* Congiure是请求中间件管道,我们需要对中间件进行灵活配置,配置分离到这个文件中方便我们的管理

## 服务构建

上面我们初步了解了HostBuilder可以配置哪些信息,我们先了解服务构建的顺序

```C#
    public static void Main(string[] args)
    {
        IHostBuilder hostBuilder = CreateHostBuilder(args);
        System.Console.WriteLine("-------------------------------");
        System.Console.WriteLine("Pre Build");
        IHost host = hostBuilder.Build();
        System.Console.WriteLine("End Build");
        System.Console.WriteLine("-------------------------------");
        host.Run();
    }

   public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureHostConfiguration(builder=>
            {
                System.Console.WriteLine("ConfigureHostConfiguration");
            })
            .ConfigureAppConfiguration(builder=>
            {
                System.Console.WriteLine("ConfigureAppConfiguration");
            })
            .ConfigureLogging(logging=>
            {
                System.Console.WriteLine("Logging");
            })
            .ConfigureServices(svcs=>
            {
                System.Console.WriteLine("ConfigureServices");
            })
            .ConfigureWebHostDefaults(webBuilder =>
            {
                System.Console.WriteLine("ConfigureWebHostDefaults");
                webBuilder.UseStartup<Startup>();
            });

    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();
            System.Console.WriteLine("Startup.ConfigureServices");
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            System.Console.WriteLine("Startup.Configure");
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseRouting();

            app.UseStaticFiles();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapGet("/", async context =>
                {
                    await context.Response.WriteAsync("Hello World!");
                });
            });
        }
    }
```
我们来看一下它运行的结果
```
ConfigureWebHostDefaults
-------------------------------
Pre Build
ConfigureHostConfiguration     
ConfigureAppConfiguration
Logging
ConfigureServices
Startup.ConfigureServices
End Build
-------------------------------
Startup.Configure
``` 

ConfigureWebHostDefaults:程序首先运行的方法
Startup.Configure Http请求管道是配置的最后执行方法

