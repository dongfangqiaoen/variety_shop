### Replugin
1. startActivity


2. 四个module
1. replutin-host-gradle  宿主的编译插件（groovy）
2. replugin-host-library  宿主库
核心module ,负责插件的初始化，加载，启动，管理等
3. replugin-plugin-gradle  插件的编译插件（groovy）
4. replugin-plugin-library  插件库
用来反射调用主程序中的相关接口，提供双向通信的能力。
其中的RePlugin、RePluginInternal、PluginServiceClient都是反射宿主App ：replugin-host-library 中的 RePlugin 、 RePluginInternal 、PluginServiceClient 类方法。

1. classloader
2. BootClassLoader 系统启动时创建，一般不需要用到
2. PathClassLoader 应用启启动时创建，只能加载内部dex
3. DexClassLoader 可以加载外部dex

1. RePluginClassLoader 宿主中的loader extends PathClassLoader 也是唯一hook住系统的classloader.
2. PluginDexClassLoader 加载插件的Loader extends DexClassLoader 用来做一些高级特性



RePluginApplication.attachBaseContext(Context)
RePlugin.App.attachBaseContext();
PMF.init(app);
PathClassLoaderUtils.path()//对宿主的HostClassLoader做修改。这是RePlugin中唯一需要修改宿主私有属性的位置了。该类只有这一个方法。