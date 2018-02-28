很多人用mybatis时候写过resultMap来定义表字段和model属性之间的映射。

(一般我们习惯把表字段定义成带下划线的英文单词， 如：user_name，而model属性定义为驼峰，如：userName)

mybatis提供一个很方便的配置项，可以省略这部分代码，自动帮你完成映射。

在mybatis的config文件中添加以下配置内容：

```xml
<settings>
    <!-- http://www.mybatis.org/mybatis-3/configuration.html#settings -->
    <setting name="mapUnderscoreToCamelCase" value="true"/> <!-- 下划线转驼峰 -->
    <setting name="autoMappingBehavior" value="FULL"/> <!-- 自动映射 -->
  </settings>
```

除此之外还有很多方便的配置项。

参考：http://www.mybatis.org/mybatis-3/configuration.html#settings
