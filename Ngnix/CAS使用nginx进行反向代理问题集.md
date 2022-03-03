## 1、无效的Cookie

CAS打印日志：

~~~shell
2022-01-18 14:57:00,919 WARN [org.apereo.cas.web.support.gen.CookieRetrievingCookieGenerator] - <InvalidCookieException>
~~~

CAS打印的错误栈信息：

~~~shell
2022-01-18 14:57:00,919 WARN [org.apereo.cas.web.support.gen.CookieRetrievingCookieGenerator] - <InvalidCookieException>
org.apereo.cas.web.support.InvalidCookieException: null
	at org.apereo.cas.web.support.mgmr.DefaultCasCookieValueManager.obtainValueFromCompoundCookie(DefaultCasCookieValueManager.java:95) ~[cas-server-core-cookie-api-6.4.0-RC6.jar:6.4.0-RC6]
	at org.apereo.cas.web.support.mgmr.EncryptedCookieValueManager.obtainCookieValue(EncryptedCookieValueManager.java:51) ~[cas-server-core-cookie-api-6.4.0-RC6.jar:6.4.0-RC6]
	at org.apereo.cas.web.cookie.CookieValueManager.obtainCookieValue(CookieValueManager.java:35) ~[cas-server-core-api-cookie-6.4.0-RC6.jar:6.4.0-RC6]
	at org.apereo.cas.web.support.gen.CookieRetrievingCookieGenerator.lambda$retrieveCookieValue$0(CookieRetrievingCookieGenerator.java:152) ~[cas-server-core-cookie-api-6.4.0-RC6.jar:6.4.0-RC6]
	at java.util.Optional.map(Optional.java:265) ~[?:?]
	at org.apereo.cas.web.support.gen.CookieRetrievingCookieGenerator.retrieveCookieValue(CookieRetrievingCookieGenerator.java:152) ~[cas-server-core-cookie-api-6.4.0-RC6.jar:6.4.0-RC6]
......略
~~~

源码报错位置：

~~~java
# 所在包<org.apereo.cas:cas-server-core-cookie-api:6.4.0-RC6>
# DefaultCasCookieValueManager#obtainValueFromCompoundCookie(...)

if (!cookieIpAddress.equals(clientInfo.getClientIpAddress())) {
    if (StringUtils.isBlank(cookieProperties.getAllowedIpAddressesPattern())
        || !RegexUtils.find(cookieProperties.getAllowedIpAddressesPattern(), clientInfo.getClientIpAddress())) {
        throw new InvalidCookieException("Invalid cookie. Required remote address "
                                         + cookieIpAddress + " does not match " + clientInfo.getClientIpAddress());
    }
    LOGGER.debug("Required remote address [{}] does not match [{}], but it's authorized proceed",
                 cookieIpAddress, clientInfo.getClientIpAddress());
}
~~~

解决办法，在`location`块中添加如下几项配置：

~~~shell
location / {
    proxy_pass https://192.168.1.35;
    # 添加以下3项配置
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Real-Port $remote_port;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
~~~

