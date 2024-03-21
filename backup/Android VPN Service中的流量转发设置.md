## 建立VPN链接代码
```
        val builder = Builder()
        builder.addAddress("10.1.10.2", 32)
        //builder.addRoute("0.0.0.0", 0)
        builder.setSession("MyVpnSession")
        // builder.setHttpProxy(ProxyInfo.buildDirectProxy(proxyHost, proxyPort))

        val vpnInterface = builder.establish()
```

1. `builder.setHttpProxy(ProxyInfo.buildDirectProxy(proxyHost, proxyPort))`: 设置系统代理，如果流量被转发到VPN中，则不会被转发到代理服务器中*
2.  `builder.addRoute("0.0.0.0", 0)`：根据路由匹配到的数据包都将转发到VPN中，优先级高于Http代理的设置
3.  `builder.addAllowedApplication`，`builder.addDisallowedApplication`：这两个方法互斥，优先级高于路由addRoute的设置

## `builder.addAllowedApplication`和`builder.addDisallowedApplication`使用
`builder.addAllowedApplication`方法用于指定哪些应用程序的流量可以通过VPN。这个方法的优先级相对较高，因为它直接控制了哪些应用程序的流量可以通过VPN。

当你调用`builder.addAllowedApplication`方法时，你指定的应用程序的流量将会被路由到VPN，而其他应用程序的流量则不会。这个设置会覆盖你通过`builder.addRoute`设置的路由。

例如，如果你调用了`builder.addRoute("0.0.0.0", 0)`，那么所有的流量都会被路由到VPN。但是，如果你接着调用了`builder.addAllowedApplication("com.example.app")`，那么只有"com.example.app"这个应用程序的流量会被路由到VPN，其他应用程序的流量则不会。

所以，`builder.addAllowedApplication`的优先级是高于`builder.addRoute`的。如果你同时设置了这两个方法，那么`builder.addAllowedApplication`的设置会覆盖`builder.addRoute`的设置。

需要注意的是，`builder.addAllowedApplication`和`builder.addDisallowedApplication`这两个方法是互斥的，你只能选择其中一个来使用。如果你同时调用了这两个方法，那么最后调用的那个方法的设置会覆盖前面的设置。

## `allowBypass`应用程序绕过VPN的流量转发
`allowBypass`方法是在构建VPN连接时使用的。当调用此方法时，应用程序可以使用`ConnectivityManager.bindProcessToNetwork`等方法绕过VPN连接直接发送/接收数据。默认情况下，所有应用程序的流量都会通过VPN接口转发，应用程序无法绕过VPN。如果调用了此方法，应用程序可以选择绕过VPN，直接使用底层网络或者它们有权限的任何其他网络进行数据传输。