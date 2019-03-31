#### 配置文件

+ SpringBOOT使用一个全局的配置文件，配置文件的名称是固定的：
  + application.properties
  + application.yaml/application.yml
+ 配置文件的作用：修改springboot默认的配置。springboot在底层给我们都配置好了。
+ Yml 是YAML语言的文件，以数据为中心，比json、xml更适合做配置文件
+ 全局配置文件可以对默认一些配置进行更改
+ SpringBoot load YAML

```java
Spring Framework provides two convenient classes that can be used to load YAML documents. The YamlPropertiesFactoryBean loads YAML as Properties and the YamlMapFactoryBean loads YAML as a Map.
```

+ yml配置例子

```yaml
server:
  port:8081
```

+ xml配置

```xml
<server>
	<port>8081</port>
</server>
```



#### 语法

+ yaml的基本语法

  + k:（空格） v 表示一对键值对（空格必须有）
  + 用空格的缩进来控制层级关系（左对齐的一列数据都是同一个层级的）

  ```yaml
  server:
  	port: 8081
  	path: /hello 
  ```

  + 属性和值也是大小写敏感的。

+ 值得写法

  + 字面量：普通值（数字、字符串、布尔）

    + K: V : 字面量直接来写
    + 字符串默认不用加上单引号或者双引号
      + "":双引号不会转义字符串里面的特殊字符，字符串会表示自身想表示的意思
        + name： "zhangsan \n lisi"  输出：zhangsan 换行 lisi
      + '':单引号，会转义特殊字符，特殊字符会被转移成特殊的字符串

  + 对象（属性和值）（键值对）

    + K: V ：在下一行写对象的属性和值

    ```20
    friends:
    	lastname: zhangsan
    	age:20
    ```

    + 行内写法：

    ```yaml
    friends: {lastname: zhangsan,age: 18}
    ```

  + 数组（List Set）

    + 用-值表示数组中的元素

    ```yaml
    pets:
    	- cat
    	- dog
    	- pig
    ```

    + 行内写法

    ```yaml
    pets: [cat,dog,pig]
    ```




#### 配置文件值注入

+ 配置文件

```yaml
person:
  name: zhangsan
  age: 12
  boss: false
  birth: 2017/12/12
  maps: {k1: v1,k2: v2}
  lists:
    - zhangsan
    - wangwu
  dog:
    name: xiaogou
    age: 2
```

+ javabean

```java
/*
* 将配置文件中配置的每一个属性的值，映射到这个组件中
* To bind to properties like that by using Spring Boot’s Binder utilities (which is what @ConfigurationProperties does)
* 使用Springboot的绑定器绑定属性（这是@ConfigurationProperties所做的）
* */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private  String name;
    private  Integer age;
    private  boolean boss;
    private  Date birth;

    private Map<String,Object> maps;

    private List<Object> lists;

    private  Dog dog;
```

+ 导入配置文件处理器

```
<!--配置文件处理器依赖，绑定配置文件就会提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```





