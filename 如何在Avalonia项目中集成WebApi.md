1.  首先需要再Avalonia项目中 编辑项目文件，添加 Microsoft.AspNetCore.App的引用

   ```
   <ItemGroup>
   		<FrameworkReference Include="Microsoft.AspNetCore.App" />
   </ItemGroup>
   ```

   ![image-20251215180429386](https://raw.githubusercontent.com/zl-aric/MyImage/main/images/image-20251215180429386.png)

2. 安装swagger包

   Swashbuckle.AspNetCore

3. 添加webApi的拓展

```c#
 public static class WebApiExtensions
 {
     public static WebApplication Build(string[] args, IServiceCollection sharedServices)
     {
         var builder = WebApplication.CreateBuilder(args);

         // 👇 关键：把 Avalonia 的服务合进来
         foreach (var sd in sharedServices)
         {
             builder.Services.Add(sd);
         }

         // ===== 标准 WebApi =====
         builder.Services.AddControllers();
         builder.Services.AddEndpointsApiExplorer();
         builder.Services.AddSwaggerGen();

         var app = builder.Build();

         if (app.Environment.IsDevelopment())
         {
             app.UseSwagger();
             app.UseSwaggerUI();
         }

         app.UseHttpsRedirection();
         app.UseAuthorization();
         app.MapControllers();

         return app;
     }
 }
```

4.   最后在Avalonia服务注册后，启动WebApi

   ```c#
   public partial class App : Application
   {
       public static IServiceProvider? Services { get; private set; }
   
       public override void Initialize()
       {
           var collection = new ServiceCollection();
           collection.AddEmpServices();
   
           var webApp = WebApiExtensions.Build(Environment.GetCommandLineArgs(), collection);
           webApp.Start();
           Services = webApp.Services;
           if (!Design.IsDesignMode)
               webApp.Services.Mirate();
   
           AvaloniaXamlLoader.Load(this);
       }
   
       public override void OnFrameworkInitializationCompleted()
       {
           var vm = Services!.GetRequiredService<MainWindowViewModel>();
           if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
           {
               // Avoid duplicate validations from both Avalonia and the CommunityToolkit.
               // More info: https://docs.avaloniaui.net/docs/guides/development-guides/data-validation#manage-validationplugins
               DisableAvaloniaDataAnnotationValidation();
   
               desktop.MainWindow = new MianWindowView()
               {
                   DataContext = vm
               };
           }
   
           base.OnFrameworkInitializationCompleted();
       }
   
       private void DisableAvaloniaDataAnnotationValidation()
       {
           // Get an array of plugins to remove
           var dataValidationPluginsToRemove =
               BindingPlugins.DataValidators.OfType<DataAnnotationsValidationPlugin>().ToArray();
   
           // remove each entry found
           foreach (var plugin in dataValidationPluginsToRemove)
           {
               BindingPlugins.DataValidators.Remove(plugin);
           }
       }
   }
   ```

   

