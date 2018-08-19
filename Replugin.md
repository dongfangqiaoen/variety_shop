### Replugin

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


### classloader的hook
工程的application要继承RePluginApplication
RePluginApplication.attachBaseContext(Context)
RePlugin.App.attachBaseContext();
PMF.init(app);
PathClassLoaderUtils.path()//对宿主的HostClassLoader做修改。这是RePlugin中唯一需要修改宿主私有属性的位置了。该类只有这一个方法。
1. 首先用反射获取宿主application中的mPackageInfo属性
2. 再反射mPackageInfo中的mClassLoader属性
3. 将反射到的mClassLoader这个属性赋值，从RePluginCallbacks.createClassLoader()方法中获取自定义的classloader，这个方法是可以在初始化Replugin是进行复写的，开发者可以赋值给自己的classloader,但一定要基于RePluginClassLoader类
4. 最后设置线程上下文中的classloader.

### startActivity
1. Replugin.startActivity()
2. Factory.startActivityWithNoInjectCN()
3. PluginCommImpl.startActivity()
4. PluginLibraryInternalProxy.startActivity()
5. PluginCommImpl.loadPluginActivity()
其中getActivityInfo（）Loader中初始化了PluginDexClassLoader(有缓存)，然后allocActivityContainer（）将坑位和activity一起，返回ComponentName
6. PluginProcessPer.allocActivityCopntainer()分配坑位，返回坑位
7. PluginProcessPer.bindActivity()返回坑位
8. PluginContainers.alloc2()真正找坑位，返回ActivityState
9. PluginContainers.allocLocked();映射坑位 找活的，找空白的，重用最老的，挤掉最老的


### 解析加载插件
1. Loader.loadDex()中对插件进行初始化
解析资源到相应目录
注册插件manifest中注册的reciver
创建插件的dexclassloader
给插件全局的context
2. 插件端初始化
Entry.create
初始化一些类，让插件能调用宿主中的方法
3. 插件端启动activity
因为插件中的activity没有在manifest中注册，所以只能靠坑位启动，启动坑位只能在宿主中完成


问题一般都是context造成的，要确定当前使用的是插件中的还是宿主中的context 
gradle配置是放在gradle末尾