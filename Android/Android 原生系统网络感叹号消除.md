# Android 原生系统网络感叹号消除

原生系统Android8.1上，WiFi上出现感叹号，此时WiFi可正常访问。

## **原因**


这是Android 5.0引入的网络评估机制：就是当你连上网络后，会给目标产生204响应的服务器发送给一个请求，如果服务器返回的是状态码为204的响应，那么就被认为网络可以访问；否则，如返回的是其他状态码，那么将被视为网络访问需要登录操作等；没有响应的话，就被认为是网络不可访问。这里的情况就是，目标服务器不能正常访问。

## 产生204响应的服务器

```
https://www.google.cn/generate_204
http://connect.rom.miui.com/generate_204
http://www.v2ex.com/generate_204
https://captive.v2ex.co/generate_204
http://www.noisyfox.cn/generate_204

```


## 修改&恢复默认

测试系统：**Android 8.1**。**默认使用https来验证**，如要使用http，需要*先写入关闭https验证的配置，再填写http服务器*。然后**开启飞行模式**，再打开感叹号即可消失。其中，xxxxx即服务器的URL。

```
# 查看所有配置
adb shell settings list global
# 使用https
adb shell settings put global captive_portal_https_url xxxxx
# 使用http
adb shell settings put global captive_portal_use_https 0
adb shell settings put global captive_portal_http_url xxxxx
# 使用默认，即删除配置
adb shell settings delete global captive_portal_http_url
adb shell settings delete global captive_portal_https_url
```



## 禁用此功能

按照上述方法，设置`captive_portal_mode`的值如下：

- 0：彻底禁用检测
- 1：检测到需要登录则弹窗提醒（默认值）
- 2：检测到需要登录则自动断开此热点并不再自动连接

## Android8.0相关源码代码：

frameworks/base/services/core/java/com/android/server/connectivity/NetworkMonitor.java

``` java
public class NetworkMonitor extends StateMachine {
    private static final String TAG = NetworkMonitor.class.getSimpleName();
    private static final boolean DBG  = true;
    private static final boolean VDBG = false;

    // Default configuration values for captive portal detection probes.
    // TODO: append a random length parameter to the default HTTPS url.
    // TODO: randomize browser version ids in the default User-Agent String.
    private static final String DEFAULT_HTTPS_URL     = "https://www.google.com/generate_204";
    private static final String DEFAULT_HTTP_URL      =
            "http://connectivitycheck.gstatic.com/generate_204";
    private static final String DEFAULT_FALLBACK_URL  = "http://www.google.com/gen_204";
    private static final String DEFAULT_OTHER_FALLBACK_URLS =
            "http://play.googleapis.com/generate_204";
    private static final String DEFAULT_USER_AGENT    = "Mozilla/5.0 (X11; Linux x86_64) "
                                                      + "AppleWebKit/537.36 (KHTML, like Gecko) "
                                                      + "Chrome/52.0.2743.82 Safari/537.36";

    private static final int SOCKET_TIMEOUT_MS = 10000;
    private static final int PROBE_TIMEOUT_MS  = 3000;

    static enum EvaluationResult {
        VALIDATED(true),
        CAPTIVE_PORTAL(false);
        final boolean isValidated;
        EvaluationResult(boolean isValidated) {
            this.isValidated = isValidated;
        }
    }
```
