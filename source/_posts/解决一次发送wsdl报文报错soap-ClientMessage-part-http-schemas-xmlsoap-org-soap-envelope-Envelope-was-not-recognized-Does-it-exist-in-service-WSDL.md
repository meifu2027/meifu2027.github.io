---
title: >-
  解决一次发送wsdl报文报错soap:ClientMessage part
  {http://schemas.xmlsoap.org/soap/envelope/}Envelope was not recognized. (Does
  it exist in service WSDL?)
categories:
  - 原创
toc: true
date: 2022-02-25 10:00:13
tags: [wsdl, WebService, php]
---
我司采用的ESB基于开源产品WSO2进行二次开发，由于二开人员已不知所踪，各种对接异构系统的疑难杂症只能由自己研究。其中与PHP对接过程中，出现了`soap:ClientMessage part
  {http://schemas.xmlsoap.org/soap/envelope/}Envelope was not recognized. (Does
  it exist in service WSDL?)`报错，Google过各种无效信息后，最终由自己解决，特此记录。
<!--more-->
## 背景
**PHP系统发数据** --> **ESB（WSO2）** --> **JAVA业务系统**
## PHP发送wsdl相关代码
```php
<?php

function curl_post($url, $data) {
   //初使化init方法
   $ch = curl_init();
   //指定URL
   curl_setopt($ch, CURLOPT_URL, $url);
   //设定请求后返回结果
   curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
   //设置头信息
   curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type:text/xml;charset=utf-8') );
   //声明使用POST方式来进行发送
   curl_setopt($ch, CURLOPT_POST, 1);
   //发送什么数据呢
   curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
   //忽略证书
   curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
   curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
   //忽略header头信息
   curl_setopt($ch, CURLOPT_HEADER, 0);
   //设置超时时间
   curl_setopt($ch, CURLOPT_TIMEOUT, 10);
   //发送请求
   $output = curl_exec($ch);
   //关闭curl
   curl_close($ch);
   //返回数据
   return $output;
}
$url = "https://192.168.0.1/services/ZTF_XYS_wSBusinessInfoService_v1.ZTF_XYS_wSBusinessInfoService_v1HttpsSoap11Endpoint";
$xmlstr = '<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:mkt="http://mkt.service.ws.paas.ccs.com/">
   <soapenv:Header/>
   <soapenv:Body>
      <mkt:sendBusinessInfo>
         <!--Optional:-->
         <arg0> 
            <businessCode>0000000001</businessCode> 
            <businessName>测试业务名称</businessName> 
            <processName>INITIATE</processName> 
            <origin>5</origin> 
            <level>GS</level> 
            <bpType>21</bpType> 
            <bpName>北京博物馆</bpName> 
            <contactsName>张三</contactsName> 
            <phone>18868868686</phone> 
            <provinceMajorTypeId>1000</provinceMajorTypeId> 
            <estimateInvest>1500000.00</estimateInvest> 
            <areaId>317000</areaId> 
            <text></text> 
            <companyId>100000</companyId> 
            <orgCode>1000000028</orgCode> 
            <createNum>01001001</createNum> 
            <createDate>2022-02-24 08:42:11</createDate> 
            <implDepartment>1000000028</implDepartment> 
            <markManager>01001001</markManager> 
            <orgManagerId>01001001</orgManagerId> 
         </arg0>
      </mkt:sendBusinessInfo>
   </soapenv:Body>
</soapenv:Envelope>';

echo "url:".$url."<br/>";
echo "xml:<br/> ";
echo $xmlstr;
$return = curl_post($url,$xmlstr);
echo "<br/>return:"."<br/>";
print_r($return);
?>
```
## 请求返回信息
```xml
<?xml version='1.0' encoding='UTF-8'?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <soap:Fault>
      <faultcode>soap:Client</faultcode>
      <faultstring>Message part {http://schemas.xmlsoap.org/soap/envelope/}Envelope was not recognized.  (Does it exist in service WSDL?)</faultstring>
    </soap:Fault>
  </soap:Body>
</soap:Envelope>
```
## 问题分析
JAVA业务系统报错信息
```log
10:16:51,328 警告    [org.apache.cxf.phase.PhaseInterceptorChain] (http--0.0.0.0-9089-8) Interceptor for {http://mkt.service.ws.paas.ccs.com/}wSBusinessInfoService has thrown exception, unwinding now: org.apache.cxf.interceptor.Fault: Message part {http://schemas.xmlsoap.org/soap/envelope/}Envelope was not recognized.  (Does it exist in service WSDL?)
	at org.apache.cxf.interceptor.DocLiteralInInterceptor.handleMessage(DocLiteralInInterceptor.java:197) [cxf-rt-core-2.5.2.jar:2.5.2]
	at org.apache.cxf.phase.PhaseInterceptorChain.doIntercept(PhaseInterceptorChain.java:263) [cxf-api-2.5.2.jar:2.5.2]
	at org.apache.cxf.transport.ChainInitiationObserver.onMessage(ChainInitiationObserver.java:123) [cxf-rt-core-2.5.2.jar:2.5.2]
	at org.apache.cxf.transport.http.AbstractHTTPDestination.invoke(AbstractHTTPDestination.java:207) [cxf-rt-transports-http-2.5.2.jar:2.5.2]
	at org.apache.cxf.transport.servlet.ServletController.invokeDestination(ServletController.java:213) [cxf-rt-transports-http-2.5.2.jar:2.5.2]
	at org.apache.cxf.transport.servlet.ServletController.invoke(ServletController.java:193) [cxf-rt-transports-http-2.5.2.jar:2.5.2]
	at org.apache.cxf.transport.servlet.CXFNonSpringServlet.invoke(CXFNonSpringServlet.java:126) [cxf-rt-transports-http-2.5.2.jar:2.5.2]
	at org.apache.cxf.transport.servlet.AbstractHTTPServlet.handleRequest(AbstractHTTPServlet.java:185) [cxf-rt-transports-http-2.5.2.jar:2.5.2]
	at org.apache.cxf.transport.servlet.AbstractHTTPServlet.doPost(AbstractHTTPServlet.java:108) [cxf-rt-transports-http-2.5.2.jar:2.5.2]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:754) [jboss-servlet-api_3.0_spec-1.0.0.Final.jar:1.0.0.Final]
	at org.apache.cxf.transport.servlet.AbstractHTTPServlet.service(AbstractHTTPServlet.java:164) [cxf-rt-transports-http-2.5.2.jar:2.5.2]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:329) [jbossweb-7.0.13.Final.jar:]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:248) [jbossweb-7.0.13.Final.jar:]
	at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:143) [spring-security-web-3.0.3.RELEASE.jar:]
	at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:343) [spring-web-3.2.5.RELEASE.jar:3.2.5.RELEASE]
	at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:260) [spring-web-3.2.5.RELEASE.jar:3.2.5.RELEASE]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:280) [jbossweb-7.0.13.Final.jar:]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:248) [jbossweb-7.0.13.Final.jar:]
	at org.jasig.cas.client.session.SingleSignOutFilter.doFilter(SingleSignOutFilter.java:110) [cas-client-core-3.1.10.jar:]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:280) [jbossweb-7.0.13.Final.jar:]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:248) [jbossweb-7.0.13.Final.jar:]
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:275) [jbossweb-7.0.13.Final.jar:]
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:161) [jbossweb-7.0.13.Final.jar:]
	at org.jboss.as.web.security.SecurityContextAssociationValve.invoke(SecurityContextAssociationValve.java:153) [jboss-as-web-7.1.1.Final.jar:7.1.1.Final]
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:155) [jbossweb-7.0.13.Final.jar:]
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:102) [jbossweb-7.0.13.Final.jar:]
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:109) [jbossweb-7.0.13.Final.jar:]
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:368) [jbossweb-7.0.13.Final.jar:]
	at org.apache.coyote.http11.Http11Processor.process(Http11Processor.java:877) [jbossweb-7.0.13.Final.jar:]
	at org.apache.coyote.http11.Http11Protocol$Http11ConnectionHandler.process(Http11Protocol.java:671) [jbossweb-7.0.13.Final.jar:]
	at org.apache.tomcat.util.net.JIoEndpoint$Worker.run(JIoEndpoint.java:930) [jbossweb-7.0.13.Final.jar:]
	at java.lang.Thread.run(Thread.java:745) [rt.jar:1.7.0_67]

```
定位到对应的代码`org.apache.cxf.interceptor.AbstractInDatabindingInterceptor#findMessagePart`
```java
protected MessagePartInfo findMessagePart(Exchange exchange, Collection<OperationInfo> operations,
                                              QName name, boolean client, int index,
                                              Message message) {
        Endpoint ep = exchange.get(Endpoint.class);
        MessagePartInfo lastChoice = null;
        BindingMessageInfo msgInfo = null;
        BindingOperationInfo boi = null;
        // 代码调试的结果显示 只存在1次遍历
        for (Iterator<OperationInfo> itr = operations.iterator(); itr.hasNext();) {
            OperationInfo op = itr.next();

            boi = ep.getEndpointInfo().getBinding().getOperation(op);
            if (boi == null) {
                continue;
            }
            if (client) {
                msgInfo = boi.getOutput();
            } else {
                msgInfo = boi.getInput();
            }

            if (msgInfo == null) {
                itr.remove();
                continue;
            }
            
            Collection bodyParts = msgInfo.getMessageParts();
            if (bodyParts.size() == 0 || bodyParts.size() <= index) {
                itr.remove();
                continue;
            }

            MessagePartInfo p = (MessagePartInfo)msgInfo.getMessageParts().get(index);
            if (name.getNamespaceURI() == null || name.getNamespaceURI().length() == 0) {
                // message part has same namespace with the message
                name = new QName(p.getMessageInfo().getName().getNamespaceURI(), name.getLocalPart());
            }
            // 经调试此处name的值为”http://schemas.xmlsoap.org/soap/envelope/“
            // 而p.getConcreteName()的值为 ”http://mkt.service.ws.paas.ccs.com/“
            // 导致 p = null 返回
            if (name.equals(p.getConcreteName())) {
                exchange.put(BindingOperationInfo.class, boi);
                exchange.put(OperationInfo.class, boi.getOperationInfo());
                exchange.setOneWay(op.isOneWay());
                return p;
            }

            if (XSD_ANY.equals(p.getTypeQName())) {
                lastChoice = p;
            } else {
                itr.remove();
            }
        }
        if (lastChoice != null) {
            setMessage(message, boi, client, boi.getBinding().getService(), msgInfo.getMessageInfo());
        }
        return lastChoice;
    } 
```

于是问题就来了，明明我php发送的报文有2个namespace（`xmlns:soapenv`,`xmlns:mkt`） 为啥到了Java系统只遍历一次呢？而且刚好遍历的namespace与我们JAVA系统发布的不一致？于是我猜测报文到ESB出了问题。
## ESB报文分析
在ESB中查看我用php发出去的报文。咦?有没有发现，好像跟老子发出去的不太对嘛，这是嘛情况？
```xml
<?xml version='1.0' encoding='utf-8'?>
    <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
        <soapenv:Body>
            <soapenv:Envelope xmlns:mkt="http://mkt.service.ws.paas.ccs.com/">
                   <soapenv:Header/>
                   <soapenv:Body>
                          <mkt:sendBusinessInfo>
                                 <!--Optional:-->
                            <arg0>
                                <businessCode>0000000001</businessCode>
                                <businessName>测试业务名称</businessName>
                                <processName>INITIATE</processName>
                                <origin>5</origin>
                                <level>GS</level>
                                <bpType>21</bpType>
                                <bpName>北京博物馆</bpName>
                                <contactsName>张三</contactsName>
                                <phone>18868868686</phone>
                                <provinceMajorTypeId>1000</provinceMajorTypeId>
                                <estimateInvest>1500000.00</estimateInvest>
                                <areaId>317000</areaId>
                                <text/>
                                <companyId>100000</companyId>
                                <orgCode>1000000028</orgCode>
                                <createNum>01001001</createNum>
                                <createDate>2022-02-24 08:42:11</createDate>
                                <implDepartment>1000000028</implDepartment>
                                <markManager>01001001</markManager>
                                <orgManagerId>01001001</orgManagerId>
                            </arg0>
                          </mkt:sendBusinessInfo>
                   </soapenv:Body>
            </soapenv:Envelope>
        </soapenv:Body>
    </soapenv:Envelope>
```

再看看使用SoapUI工具调用接口发送报文，在ESB中看到的报文信息如下。发现啥子没有？这特么才是正常的报文。
```xml
<?xml version='1.0' encoding='utf-8'?>
    <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:mkt="http://mkt.service.ws.paas.ccs.com/">
        <soapenv:Header/>
        <soapenv:Body>
              <mkt:sendBusinessInfo>
                      <!--Optional:-->
                <arg0>
                    <businessCode>0000000001</businessCode>
                    <businessName>测试业务名称</businessName>
                    <processName>INITIATE</processName>
                    <origin>5</origin>
                    <level>GS</level>
                    <bpType>21</bpType>
                    <bpName>北京博物馆</bpName>
                    <contactsName>张三</contactsName>
                    <phone>18868868686</phone>
                    <provinceMajorTypeId>1000</provinceMajorTypeId>
                    <estimateInvest>1500000.00</estimateInvest>
                    <areaId>317000</areaId>
                    <text/>
                    <companyId>100000</companyId>
                    <orgCode>1000000028</orgCode>
                    <createNum>01001001</createNum>
                    <createDate>2022-02-24 08:42:11</createDate>
                    <implDepartment>1000000028</implDepartment>
                    <markManager>01001001</markManager>
                    <orgManagerId>01001001</orgManagerId>
                </arg0>
              </mkt:sendBusinessInfo>
        </soapenv:Body>
    </soapenv:Envelope>
```

## 问题解决
找到了问题的原因之后，就是去解决为啥在第一个`<soapenv:Envelope>`标签中少了一个namespace的问题。最终，在本人的不懈尝试之下发现将PHP其中的一行代码增加参数即可实现报文到达ESB不变形的目的。
```php
  //设置头信息
   curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type:text/xml;charset=utf-8','SOAPAction:mkt:sendBusinessInfo') );
```
其实本文中PHP发送wsdl报文的方式是模拟soapUI的，包括其中的头信息、报文内容。但是关于`SOAPAction`参数，从soapUI中查看发送的信息，该参数也一直为空的，最原始的PHP代码在直连Java系统的时候，是可以正常访问的，无需加`SOAPAction`,至于为何接入ESB之后没有此参数报文就异常的原因，暂不清楚。

`至此，问题解决。`撒花~~~