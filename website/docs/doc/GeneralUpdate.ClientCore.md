---
sidebar_position: 2
---

### 定义

命名空间：GeneralUpdate.ClientCore

程序集：GeneralUpdate.ClientCore.dll



GeneralUpdate.ClientCore是最核心的组件之一，提供了大量主要功能。本质和Core没有区别，但是有职责上的区别ClientCore用于主程序中，更新升级助手然后关闭主程序启动升级助手。

```c#
public class GeneralClientBootstrap : AbstractBootstrap<GeneralClientBootstrap, IStrategy>
```

nuget安装

```shell
NuGet\Install-Package GeneralUpdate.ClientCore -Version 1.0.0
```



### 示例

![](imgs/muti_donwload.png)

以下示例定义方法，包含GeneralClientBootstrap使用。

```c#
//ClientStrategy该更新策略将完成1.自动升级组件自更新 2.启动更新组件 3.配置好ClientParameter无需再像之前的版本写args数组进程通讯了。
//generalClientBootstrap.Config(baseUrl, "B8A7FADD-386C-46B0-B283-C9F963420C7C").
var configinfo = GetWindowsConfigInfo();
var generalClientBootstrap = await new GeneralClientBootstrap()
//单个或多个更新包下载通知事件
.AddListenerMultiDownloadProgress(OnMultiDownloadProgressChanged)
//单个或多个更新包下载速度、剩余下载事件、当前下载版本信息通知事件
.AddListenerMultiDownloadStatistics(OnMultiDownloadStatistics)
//单个或多个更新包下载完成
.AddListenerMultiDownloadCompleted(OnMultiDownloadCompleted)
//完成所有的下载任务通知
.AddListenerMultiAllDownloadCompleted(OnMultiAllDownloadCompleted)
//下载过程出现的异常通知
.AddListenerMultiDownloadError(OnMultiDownloadError)
//整个更新过程出现的任何问题都会通过这个事件通知
.AddListenerException(OnException)
.Config(configinfo)
.Option(UpdateOption.DownloadTimeOut, 60)
.Option(UpdateOption.Encoding, Encoding.Default)
.Option(UpdateOption.Format, Format.ZIP)
//开启驱动更新
//.Option(UpdateOption.Drive, true)
//开启遗言功能，需要部署GeneralUpdate.SystemService Windows服务。
.Option(UpdateOption.WillMessage, true)
.Strategy<WindowsStrategy>()
//注入一个func让用户决定是否跳过本次更新，如果是强制更新则不生效
//.SetCustomSkipOption(ShowCustomOption)
//注入一个自定义方法集合，该集合会在更新启动前执行。执行自定义方法列表如果出现任何异常，将通过异常订阅通知。（推荐在更新之前检查当前软件环境）
//.AddCustomOption(new List<Func<bool>>() { () => Check1(), () => Check2() })
//默认黑名单文件： { "Newtonsoft.Json.dll" } 默认黑名单文件扩展名： { ".patch", ".7z", ".zip", ".rar", ".tar" , ".json" }
//如果不需要扩展，需要重新传入黑名单集合来覆盖。
//.SetBlacklist(GetBlackFiles(), GetBlackFormats())
.LaunchTaskAsync();

private List<string> GetBlackFiles()
{
  var blackFiles = new List<string>();
  blackFiles.Add("MainApp");
  return blackFiles;
}

private List<string> GetBlackFormats()
{
  var blackFormats = new List<string>();
  blackFormats.Add(".zip");
  return blackFormats;
}

/// <summary>
/// 获取Windows平台所需的配置参数
/// </summary>
/// <returns></returns>
private Configinfo GetWindowsConfigInfo()
{
  //该对象用于主程序客户端与更新组件进程之间交互用的对象
  var config = new Configinfo();
  //本机的客户端程序应用地址
  config.InstallPath = @"D:\packet\source";
  //更新公告网页
  config.UpdateLogUrl = "https://www.baidu.com/";
  //客户端当前版本号
  config.ClientVersion = "1.1.1.1";
  //客户端类型：1.主程序客户端 2.更新组件
  config.AppType = AppType.UpgradeApp;
  //指定应用密钥，用于区分客户端应用
  config.AppSecretKey = "B8A7FADD-386C-46B0-B283-C9F963420C7C";
  //更新组件更新包下载地址
  config.UpdateUrl = $"{baseUrl}/versions/{config.AppType}/{config.ClientVersion}/{config.AppSecretKey}";
  //更新程序exe名称
  config.AppName = "GeneralUpdate.Core";
  //主程序客户端exe名称
  config.MainAppName = "GeneralUpdate.ClientCore";
  //主程序信息
  var mainVersion = "1.1.1.1";
  //主程序客户端更新包下载地址
  config.MainUpdateUrl = $"{baseUrl}/versions/{AppType.ClientApp}/{mainVersion}/{config.AppSecretKey}";
  return config;
}

/// <summary>
/// 获取Android平台所需要的参数
/// </summary>
/// <returns></returns>
private Configinfo GetAndroidConfigInfo()
{
  var config = new Configinfo();
  config.InstallPath = System.Threading.Thread.GetDomain().BaseDirectory;
  //主程序客户端当前版本号
  config.ClientVersion = "1.0.0.0"; //VersionTracking.Default.CurrentVersion.ToString();
  config.AppType = AppType.ClientApp;
  config.AppSecretKey = "41A54379-C7D6-4920-8768-21A3468572E5";
  //主程序客户端exe名称
  config.MainAppName = "GeneralUpdate.ClientCore";
  //主程序信息
  var mainVersion = "1.1.1.1";
  config.MainUpdateUrl = $"{baseUrl}/versions/{AppType.ClientApp}/{mainVersion}/{config.AppSecretKey}";
  return config;
}

/// <summary>
/// 让用户决定是否跳过本次更新
/// </summary>
/// <returns></returns>
private async Task<bool> ShowCustomOption()
{
  return await Task.FromResult(true);
}

private void OnMultiDownloadStatistics(object sender, MultiDownloadStatisticsEventArgs e)
{
  //e.Remaining 剩余下载时间
  //e.Speed 下载速度
  //e.Version 当前下载的版本信息
}

private void OnMultiDownloadProgressChanged(object sender, MultiDownloadProgressChangedEventArgs e)
{
  //e.TotalBytesToReceive 当前更新包需要下载的总大小
  //e.ProgressValue 当前进度值
  //e.ProgressPercentage 当前进度的百分比
  //e.Version 当前下载的版本信息
  //e.Type 当前正在执行的操作  1.ProgressType.Check 检查版本信息中 2.ProgressType.Donwload 正在下载当前版本 3. ProgressType.Updatefile 更新当前版本 4. ProgressType.Done更新完成 5.ProgressType.Fail 更新失败
  //e.BytesReceived 已下载大小
  DispatchMessage($"{e.ProgressPercentage}%");
  //MyProgressBar.ProgressTo(e.ProgressValue, 100, Easing.Default);
}

private void OnException(object sender, ExceptionEventArgs e)
{
  //DispatchMessage(e.Exception.Message);
}

private void OnMultiAllDownloadCompleted(object sender, MultiAllDownloadCompletedEventArgs e)
{
  //e.FailedVersions; 如果出现下载失败则会把下载错误的版本、错误原因统计到该集合当中。
  DispatchMessage($"Is all download completed {e.IsAllDownloadCompleted}.");
}

private void OnMultiDownloadCompleted(object sender, MultiDownloadCompletedEventArgs e)
{
  var info = e.Version as VersionInfo;
  DispatchMessage($"{info.Name} download completed.");
}

private void OnMultiDownloadError(object sender, MultiDownloadErrorEventArgs e)
{
  var info = e.Version as VersionInfo;
  DispatchMessage($"{info.Name} error!");
}

private void DispatchMessage(string message)
{
    ShowMessage(message);
}
```



### 注解

GeneralClientBootstrap提供以下能力。

#### 构造函数

| Constructors             |                                    |
| ------------------------ | ---------------------------------- |
| GeneralClientBootstrap() | 当前GeneralUpdateBootstrap构造函数 |
| base:AbstractBootstrap() | 父类AbstractBootstrap构造函数      |

#### 属性

| Properties   |                                                            |
| ------------ | ---------------------------------------------------------- |
| Packet       | 更新包信息                                                 |
| UpdateOption | 更新操作配置设置枚举                                       |
| Configinfo   | 客户端相关参数类（AppType、AppName、AppSecretKey等字段）。 |

#### 方法

| Method                                 |                                                              |
| -------------------------------------- | ------------------------------------------------------------ |
| LaunchTaskAsync()                      | Task异步启动更新                                             |
| LaunchAsync()                          | 启动更新                                                     |
| SetBlacklist()                         | 设置更新文件黑名单，如果不需要更新文件 名则传入即可。        |
| Option()                               | 设置更新配置。                                               |
| Config()                               | 更新相关内容配置参数，url 服务器地址及 端口号, appSecretKey客户端唯一标识用于 区分产品分支。 |
| GetOption()                            | 获取更新配置。                                               |
| Strategy()                             | 设置当前更新策略，例如：如果是Windows 平台则使用WindowsStrategy， linux...mac...android以此类推。 |
| SetCustomSkipOption()                  | 让用户在非强制更新的状态下决定是否进行更新。                 |
| AddCustomOption()                      | 添加一个异步的自定义操作。理论上，任何自定义操作都可以完成。建议注册环境检查方法，以确保更新完成后存在正常的依赖和环境。 |
| AddListenerMultiAllDownloadCompleted() | 完成所有的下载任务通知。                                     |
| AddListenerMultiDownloadProgress()     | 单个或多个更新包下载通知事件。                               |
| AddListenerMultiDownloadCompleted()    | 单个或多个更新包下载完成事件。                               |
| AddListenerMultiDownloadError()        | 监听每个版本下载异常的事件                                   |
| AddListenerMultiDownloadStatistics()   | 单个或多个更新包下载速度、剩余下载事 件、当前下载版本信息通知事件。 |
| AddListenerException()                 | 整个更新过程出现的任何问题都会通过这个回调函数通知。         |



### 🌴Packet

| 属性                                                         |
| ------------------------------------------------------------ |
| **MainUpdateUrl** string 更新检查api地址。                   |
| **AppType** int 1:ClientApp 2:UpdateApp                      |
| **UpdateUrl** string Update 更新检查api地址。                |
| **AppName**  string 需要启动应用程序的名称。                 |
| **MainAppName** string 需要启动主应用程序的名称。            |
| **Format** string 更新包文件格式（默认格式为Zip）。          |
| **IsUpgradeUpdate** bool 是否需要更新来升级应用程序。        |
| **IsMainUpdate** bool 主应用程序是否需要更新。               |
| **UpdateLogUrl** string 更新日志网页地址。                   |
| **UpdateVersions** List 需要更新的版本信息VersionInfo。      |
| **Encoding** Encoding 文件操作的编码格式。                   |
| **DownloadTimeOut** int 下载超时时间。                       |
| **AppSecretKey** string 应用程序密钥，需要和服务器约定好。   |
| **ClientVersion** string 客户端当前版本号。                  |
| **LastVersion** string 最新版本号。                          |
| **InstallPath** string 安装路径（用于更新文件逻辑）。        |
| **TempPath** string 下载文件临时存储路径（用于更新文件逻辑）。 |
| **ProcessBase64** string 升级终端程序的配置参数。            |
| **Platform** string 当前策略所属的平台。（Windows\linux\Mac） |
| **BlackFiles** List 黑名单中的文件将跳过更新。               |
| **BlackFormats** 黑名单中的文件格式将跳过更新。              |
| **DriveEnabled** bool 是否启用驱动升级功能。                 |
| **WillMessageEnabled** bool 是否开启遗言功能，如果想要启动需要同步部署'GeneralUpdate. SystemService'服务。 |

### 🌴Configinfo

| **属性**                                              |
| ----------------------------------------------------- |
| **AppType** int 1:ClientApp 2:UpdateApp               |
| **AppName**  string 需要启动应用程序的名称。          |
| **AppSecretKey** string 应用程序密钥。                |
| **ClientVersion** string 客户端当前版本。             |
| **UpdateUrl** string 更新检查api地址。                |
| **UpdateLogUrl** string 更新日志网页地址。            |
| **InstallPath** string 安装路径（用于更新文件逻辑）。 |
| **MainUpdateUrl** string  更新检查api地址。           |
| **MainAppName** string  主客户端应用名称              |



### 🍵UpdateOption

| **枚举**                                                     |
| ------------------------------------------------------------ |
| **Format** 更新包的文件格式。                                |
| **Encoding**  压缩编码。                                     |
| **MainApp** 主程序名称。                                     |
| **DownloadTimeOut** 超时时间（单位：秒）。如果未指定此参数，则默认超时时间为30秒。 |
| **Drive** 是否启用驱动升级功能。                             |
| **WillMessage** 是否开启遗言功能，如果想要启动需要同步部署'GeneralUpdate. SystemService'服务。 |



### 🌼LaunchTaskAsync()

**方法**

Task异步启动更新。

```c#
public Task<GeneralUpdateBootstrap> LaunchTaskAsync();
```



### 🌼LaunchAsync()

**方法**

启动更新。

```c#
public virtual TBootstrap LaunchAsync();
```



### 🌼SetBlacklist()

**方法**

设置更新时会忽略的黑名单信息，避免特殊文件二进制差分更新时无法使用导致更新失败。

```c#
public virtual TBootstrap SetBlacklist(List<string> files = null, List<string> fileFormats = null);
```



**参数类型**

```c#
List<string> 黑名单信息集合。
```



**参数**

```c#
files List<string> 黑名单文件名称集合。

fileFormats List<string> 黑名单文件后缀集合。
```



### 🌼Option()

**方法**

设置更新配置。

```c#
public virtual TBootstrap Option<T>(UpdateOption<T> option, T value);
```



**参数类型**

T 要设置更新操作UpdateOption。



**参数**

```c#
option UpdateOption<T> 配置动作枚举。

value T 需要设置的值，值类型根据UpdateOption枚举来。
```



### 🌼Config()

**方法**

Custom Configuration (Recommended : All platforms).

```c#
public GeneralClientBootstrap Config(Configinfo info);
public GeneralClientBootstrap Config(string url, string appSecretKey, string appName = "GeneralUpdate.Upgrade");
```



**参数类型**

Configinfo 

客户端相关参数类（AppType、AppName、AppSecretKey等字段）。



**参数**

**info** Configinfo 客户端相关参数类。

**url** string 远程服务器地址。

**appSecretKey** string  application key(与服务端约定好的密钥，用于区分客户端进行版本管理或指定客户端推送升级).

**appName** string 更新程序的名称不需要包含扩展名。



### 🌼GetOption()

**方法**

```c#
public virtual T GetOption<T>(UpdateOption<T> option);
```

**参数类型**

T 

根据UpdateOption枚举获取结果。



**参数**

```c#
option UpdateOption<T> 具体枚举内容参考本文档中的 🍵UpdateOption。
```



### 🌼Strategy()

**方法**

根据不同的操作系统平台，指定更新策略。

```c#
public virtual TBootstrap Strategy<T>() where T : TStrategy, new();
```

**参数类型**

T 

设置符合当前操作系统的更新策略，例如：Windows操作系统使用WindowsStrategy。



### 🌼SetCustomSkipOption()

**方法**

让用户决定是否在非强制更新状态下进行更新。

```c#
public GeneralClientBootstrap SetCustomSkipOption(Func<bool> func);
public GeneralClientBootstrap SetCustomSkipOption(Func<Task<bool>> func);
```



**参数类型**

```c#
Func<bool> 注入一个同步的自定义回调函数，通常用来做用户决定是否跳过本次版本更新。
Func<Task<bool>> 注入一个Task异步的自定义回调函数。
```



**参数**

```c#
func Func<bool>  注入一个同步的自定义回调函数，通常用来做用户决定是否跳过本次版本更新。
func Func<Task<bool>>  注入一个Task异步的自定义回调函数。
```



### 🌼AddCustomOption()

**方法**

添加一个异步的自定义操作。理论上，任何自定义操作都可以完成。建议注册环境检查方法，以确保更新完成后存在正常的依赖和环境。

```c#
public GeneralClientBootstrap AddCustomOption(List<Func<bool>> funcs);
public GeneralClientBootstrap AddCustomOption(List<Func<Task<bool>>> funcs);
```



**参数类型**

```c#
List<Func<bool>> 注入一组同步的带bool返回值的自定义回调函数。

List<Func<Task<bool>>> 注入一组Task异步的带bool返回值的自定义回调函数。
```



**参数**

```c#
funcs List<Func<bool>> 注入一组同步的自定义回调函数，通常用来做环境检查工作（例如：检查是否缺少VC++的环境库、或硬件设备驱动程序打印机、摄像头等）。

funcs List<Func<Task<bool>>> 注入一组Task异步的自定义回调函数。
```



### 🌼AddListenerMultiAllDownloadCompleted()

**方法**

```c#
public TBootstrap AddListenerMultiAllDownloadCompleted(Action<object, MultiAllDownloadCompletedEventArgs> callbackAction);
```



**参数类型**

**sender** object

操作句柄。

**args** MultiAllDownloadCompletedEventArgs 

所有版本下载完成通知参数。



**参数**

```c#
callbackAction Action<object, MultiAllDownloadCompletedEventArgs> 
```

监听所有更新版本下载完成的事件回传参数。



### 🌼AddListenerMultiDownloadProgress()

**方法**

```c#
public TBootstrap AddListenerMultiDownloadProgress(Action<object, MultiDownloadProgressChangedEventArgs> callbackAction);
```



**参数类型**

**sender** object 

操作句柄。

**args** MultiDownloadProgressChangedEventArgs 

进度通知参数。



**参数**

```c#
callbackAction Action<object, MultiDownloadProgressChangedEventArgs> 
```

监听每个版本下载进度事件回传参数。



### 🌼AddListenerMultiDownloadCompleted()

**方法**

```c#
public TBootstrap AddListenerMultiDownloadCompleted(Action<object, MultiDownloadCompletedEventArgs> callbackAction);
```



**参数类型**

sender object 

操作句柄。

MultiDownloadCompletedEventArgs

监听每个版本更新包下载完成回传参数。



**参数**

```c#
callbackAction Action<object, MultiDownloadCompletedEventArgs>
```

监听每个版本下载异常回传参数。



### 🌼AddListenerMultiDownloadError()

**方法**

```c#
public TBootstrap AddListenerMultiDownloadError(Action<object, MultiDownloadErrorEventArgs> callbackAction);
```



**参数类型**

**sender** object 

操作句柄。

**args** MultiDownloadErrorEventArgs

下载异常通知参数。



**参数**

```c#
callbackAction Action<object, MultiDownloadErrorEventArgs>
```

监听每个版本下载异常回传参数。



### 🌼AddListenerMultiDownloadStatistics()

**方法**

```c#
public TBootstrap AddListenerMultiDownloadStatistics(Action<object, MultiDownloadStatisticsEventArgs> callbackAction);
```



**参数类型**

**sender** object 

操作句柄。

**args** MultiDownloadStatisticsEventArgs

下载信息统计（下载速度、下载大小等）参数。



**参数**

```c#
callbackAction Action<object, MultiDownloadStatisticsEventArgs>
```

监听每个版本下载统计信息事件。



### 🌼AddListenerException()

**方法**

```c#
public TBootstrap AddListenerException(Action<object, ExceptionEventArgs> callbackAction);
```



**参数类型**

**sender** object 

操作句柄。

**args** ExceptionEventArgs

异常参数。



**参数**

```c#
callbackAction Action<object, ExceptionEventArgs>
```

监听更新组件内部的所有异常。



### 适用于

| 产品           | 版本          |
| -------------- | ------------- |
| .NET           | 5、6、7、8、9 |
| .NET Framework | 4.6.1         |
| .NET Standard  | 2.0           |
| .NET Core      | 2.0           |
