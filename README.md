# WKWebView
WKWebView添加cookie方法，自己亲测可用

          // 可以创建多个cookie ，注意，这里cookie一定要创建完整的，越完整越好，切记，只设name 和 value是没用的
         let cookie = HTTPCookie(properties: [HTTPCookiePropertyKey.name : "ga",
                                             HTTPCookiePropertyKey.value: "\(LBConfiguration.enviroment == .Gray ? 1 : 0)",
                                             HTTPCookiePropertyKey.domain: ".aaaaaa.com",
                                             HTTPCookiePropertyKey.path: "/",
                                             HTTPCookiePropertyKey.version: 0,
                                             HTTPCookiePropertyKey.expires: Date(timeIntervalSinceNow: 60*60*24*99)])

        let cookie2 = HTTPCookie(properties: [HTTPCookiePropertyKey.name : "ga",
                                              HTTPCookiePropertyKey.value: "\(LBConfiguration.enviroment == .Gray ? 1 : 0)",
                                              HTTPCookiePropertyKey.domain: ".bbbbbb.com",
                                              HTTPCookiePropertyKey.path: "/",
                                              HTTPCookiePropertyKey.version: 0,
                                              HTTPCookiePropertyKey.expires: Date(timeIntervalSinceNow: 60*60*24*99)])
        // 这里实际上是清理已存在的旧cookie，这一步不做，老的cookie可能会影响新的cookie，现在测试来看，不清也没问题
        if let oldCookies = HTTPCookieStorage.shared.cookies {
            for ck in oldCookies {
                if ck.name == "ga" {
                    HTTPCookieStorage.shared.deleteCookie(ck)
                }
            }
            // 这里仅仅是用来查看是否已清掉老的cookie
            if let temCookies = HTTPCookieStorage.shared.cookies {
                for ck in temCookies {
                    print("cookie = ",ck.name)
                }
            }
        }
          
        if #available(iOS 11.0, *) {
           // iOS11 之后苹果给了API,就不用那么麻烦了，直接先设cookie，再在回调里加载，我这里调二次，是因为工程可能有两个或更多的主域名，如果只有一个域名，就不用这样做了
            mainWeb.configuration.websiteDataStore.httpCookieStore.setCookie(cookie!) { [weak self] in
                self?.mainWeb.configuration.websiteDataStore.httpCookieStore.setCookie(cookie2!) {
                    //这里特别要留意，request一定要在回调内创建，不然会有问题，切记
                    let request = URLRequest(url: URL(string: urlstr)!)
                    self?.mainWeb.load(request)
                }
            }
        } else {
                  // 这里要注意了，如果你是单域名，只需要setValue一次就行，这里之所以这样做，是为了解决第一次请求时没有设上cookie的问题
                  var request = URLRequest(url: URL(string: "http://xx.aaaaaa.com/xxxxxx")!)
                  request.setValue(getCookieString(cookie!), forHTTPHeaderField: "Cookie")
                  request.setValue(getCookieString(cookie2!), forHTTPHeaderField: "Cookie")
                  // 这里作用是通过js注入的方式把cookie种到本地，之后就会一直有了
                  let cookieScript = WKUserScript(source: "document.cookie = '\(getCookieString(cookie!))';document.cookie = '\(getCookieString(cookie2!))';", injectionTime: WKUserScriptInjectionTime.atDocumentStart, forMainFrameOnly: false)
                  mainWeb.configuration.userContentController.addUserScript(cookieScript)
                  mainWeb.load(request)
        }
        
        // cookie转换成字符串，方便写入，
        func getCookieString(_ cookie:HTTPCookie) -> String {
            let str = "\(cookie.name)=\(cookie.value);domain=\(cookie.domain);expiresDate=\(cookie.expiresDate ?? Date(timeIntervalSinceNow: 60*60*24*99));path=\(cookie.path);sessionOnly=\(cookie.isSessionOnly ? "TRUE":"FALSE");isSecure=\(cookie.isSecure ? "TRUE":"FALSE")"
            return str
        }
