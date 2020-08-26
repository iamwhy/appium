## iOS的UIAutomation Driver

> **注意**: 这个driver已被弃用（_DEPRECATED_），除非特殊情况，建议不再使用此driver。
> 本文档中信息或将不再更新，此driver将在未来的Appium版本中被移除。如需使用Appium进行iOS的
> 自动化，请改用[XCUITest Driver](/docs/cn/drivers/ios-xcuitest.md)。


Appium以往的iOS应用自动化方案是基于`UIAutomation`实现，它是由Apple公司提供的框架，随iOS SDK发布，iOS 10后被移除。`UIAutomation`是AppleInstruments分析系统中提供的工具之一，它提供了一个在单个应用程序上下文中同步运行的JavaScript的API。Appium UIAutomation driver为此API建立了一个异步的、基于session的WebDriver前端。


UIAutomation driver的开发已完成，见[appium-ios-driver](https://github.com/appium/appium-ios-driver) repo.

### 要求和支持

除Appium的一般要求：

* Xcode 7或更低版本。
* iOS模拟器或设备系统版本为9.3或更低。
* 此driver附带Appium的所有版本。
* 为了使此driver正常工作，请参见下面的其他设置。

### 用途

使用UIAutomation driver新建一个session，需要将[new session request](#TODO)中的`platformName` [capability](#TODO)值设为`iOS`。当然，想要正常使用，您至少需要为`platformVersion`，`deviceName`以及`app`赋值。


### Capabilities

UIAutomation driver支持一系列标准的[Appium capabilities](/docs/cn/writing-running-appium/caps.md)，另外还有一些仅适用于此driver的capabilities（请参阅文档中的[iOS部分](/docs/cn/writing-running-appium/caps.md#ios-only)）。

如果想要自动化Safari，需要将`app`置为空，同时将`browserName`设为`Safari`。

### 命令

查看Appium支持的各种命令，以及与UIAutomation driver相关的命令及其行为的信息，请参见[API Reference](#TODO)。


### 模拟器设置

（请注意由于Xcode和iOS模拟器的限制，在任一时间段，只允许打开一个模拟器进行自动化操作。如果需要支持多个模拟器，请升级到[XCUITest driver](ios-xcuitest.md)）。


1. 为了使iOS模拟器能够通过Instruments实现自动化，你需要修改系统的授权数据库。Appium提供了一个简单的方法，您只需要安装并执行一个授权脚本：

    ```
    sudo authorize-ios
    ```

1. 默认的，基于Instruments实现的自动化有一个硬性限制，即命令之间存在着1秒的延迟，这是被Apple的工程师出于一些不知名的原因写死的。要绕过这个限制您可以参考[instruments-without-delay](https://github.com/facebookarchive/instruments-without-delay)
(IWD)。在Xcode 7之前，IWD是随Appium发布的。在7.x即更新版本上，IWD需要由用户手动安装。方法如下：

    * Clone [appium-ios-driver](https://github.com/appium/appium-ios-driver) repository。
    * 进入repo，执行`bin`目录下的`xcode-iwd.sh`脚本，需要提供参数：(1)Xcode的路径。(2)appium-instruments目录的路径。例如：

        ```
        sh ./bin/xcode-iwd.sh /Applications/Xcode.app /Users/me/appium-instruments/
        ```

1. 为了获得最佳结果，请启动您要使用的每个模拟器，并确保以下各项：

    * 启用软键盘（模拟器app中执行Command+K）
    * 开发者设置菜单里打开UIAutomation
    * 确认Xcode的"Devices"中没有重名的模拟器

### 真机配置

在真机上执行测试相对比较复杂，因为涉及到代码签名以及绕过Apple限制的额外操作。要成功的运行自动化测试需要遵循以下基本流程：


1. 使用Debug设置来build你的应用，针对特定的测试设备，要确保你的应用已经被签名允许在你的设备上运行。例如：

    ```
    xcodebuild -sdk <iphoneos> -target <target_name> -configuration Debug \
        CODE_SIGN_IDENTITY="iPhone Developer: Mister Smith" \
        PROVISIONING_PROFILE="XXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXX"
    ```

1. 在测试设备上安装已经build好的应用（通常编好的应用会产生在Xcode的编译目录下），确保能够在设备上找到已安装的应用，并且没有签名报错。关于如何往测试设备上安装应用由几种方法。其中一种是直接使用Xcode来安装。另一种是通过`libimobiledevice`组件提供的`ideviceinstaller`工具来安装。第三种方式是通过[ios-deploy](https://npmjs.org/package/ios-deploy)。以下是一个使用`ideviceinstaller`的例子：

    ```
    # first install ideviceinstaller, using Homebrew (http://brew.sh)
    brew install libimobiledevice
    ideviceinstaller -u <UDID of your device> -i <path to your built app>
    ```

1. 将应用的bundle ID配置为`app`capability的值。
1. 将设备的UDID设置为`udid` capability的值。
1. 和前述设置一样，确保UI Automation在开发者设置中是打开的。

遵循这些步骤即可成功使用此driver！如果您使用的是新版本的Xcode（如7.x），您可以参考[XCUITest Driver Real Device Docs](/docs/cn/drivers/ios-xcuitest-real-devices.md)，这里或许会有相关的信息。

### 真机Hybrid/Web测试

对于混合应用及web的测试，Appium需要用到远程调试协议发送JavaScript来操作web视图。对于iOS真机，这个协议是加密的，且必须要由一个第三方的工具来简化访问，这个工具由Google提供，叫做[ios-webkit-debug-proxy](https://github.com/google/ios-webkit-debug-proxy)
(IWDP)。如何在Appium上安装和使用IWDP，请参考[IWDP doc](/docs/cn/writing-running-appium/web/ios-webkit-debug-proxy.md)。

对于web测试，也就是说在Safari上运行的测试，我们有另外一个需要克服的问题。在真机上，一个应用如果没有被开发者签名则不能被UIAutomation进行插桩。而Safari就是这种应用。因此我们需要用到一个辅助应用`SafariLauncher`，这个应用可以（_can_ ）被开发者签名。这个应用唯一的目的就是在它启动时去启动Safari，然后通过远程调试器结合IWDP实现自动化。然而，在这种情况下，您将不能进入native的上下文或者对浏览器本身做自动化。

`SafariLauncher`的配置请参考[SafariLauncher doc](/docs/cn/drivers/ios-uiautomation-safari-launcher.md)。

### iOS测试产生的文件

iOS测试后产生的文件有时可能会比较大。包括日志文件、临时文件以及Xcode运行产生的数据。通常来说这些文件会存在于如下位置，您可手动删除：

```
$HOME/Library/Logs/CoreSimulator/*
/Library/Caches/com.apple.dt.instruments/*
```

### 使用Jenkins进行iOS测试

首先下载`jenkins-cli.jar`并确认Mac能够成功连接到Jenkins的master节点。需确保你已经执行了前文提到的`authorize-ios`命令。

```
wget https://jenkins.ci.cloudbees.com/jnlpJars/jenkins-cli.jar

java -jar jenkins-cli.jar \
 -s https://team-appium.ci.cloudbees.com \
 -i ~/.ssh/id_rsa \
 on-premise-executor \
 -fsroot ~/jenkins \
 -labels osx \
 -name mac_appium
```

接下来为Jenkins定义一个LaunchAgent，以在登录时自动启动。由于守护进程没有GUI权限，LaunchDaemon将无法工作。要确保plist内没有`SessionCreate`或者`User`字段，因为它们可能会影响测试的运行。如果配置出错了，您将会看到一个`Failed to authorize rights`报错。。

```
$ sudo nano /Library/LaunchAgents/com.jenkins.ci.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.jenkins.ci</string>
    <key>ProgramArguments</key>
    <array>
        <string>java</string>
        <string>-Djava.awt.headless=true</string>
        <string>-jar</string>
        <string>/Users/appium/jenkins/jenkins-cli.jar</string>
        <string>-s</string>
        <string>https://instructure.ci.cloudbees.com</string>
        <string>on-premise-executor</string>
        <string>-fsroot</string>
        <string>/Users/appium/jenkins</string>
        <string>-executors</string>
        <string>1</string>
        <string>-labels</string>
        <string>mac</string>
        <string>-name</string>
        <string>mac_appium</string>
        <string>-persistent</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/appium/jenkins/stdout.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/appium/jenkins/error.log</string>
</dict>
</plist>
```

最后，设置好owner、权限等，就可以启动agent了。

```
sudo chown root:wheel /Library/LaunchAgents/com.jenkins.ci.plist
sudo chmod 644 /Library/LaunchAgents/com.jenkins.ci.plist

launchctl load /Library/LaunchAgents/com.jenkins.ci.plist
launchctl start com.jenkins.ci
```
