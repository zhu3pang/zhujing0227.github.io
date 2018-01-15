---
layout: post
title: 利用lomcok减少Java冗余代码
tags: lombok
category: lombok
#eye_catch: http://jekyllrb.com/img/logo-2x.png
---

lombok是在编译期间生成的字节码,不会影响程序的运行性能,**[lombok 官网](https://projectlombok.org/)**

常用的几个注解

@Data   ：注解在类上；提供类所有属性的 getter 和 setter 方法, 此外还提供了equals、canEqual、hashCode、toString 方法

@Accessors(chain=true,fluent=true):链式调用,当chain=true时,只是链式调用;当fluent=true时,getter/setter方法前面没有get/set

@Setter：注解在属性上；为属性提供 setter 方法

@Getter：注解在属性上；为属性提供 getter 方法

@Log4j ：注解在类上；为类提供一个 属性名为log 的 log4j 日志对象

@NoArgsConstructor：注解在类上；为类提供一个无参的构造方法

@AllArgsConstructor：注解在类上；为类提供一个全参的构造方法

---
<!--more-->
<!--more-->

```java
@Data
// @Accessors(chain=true)
@AllArgsConstructor
@NoArgsConstructor(staticName = "of")
public class User {

    private String name;
    private Integer age;

}
```

```java
public class User{

    private String name;
    private Integer age;

    getter/setter...

    public User(String name, Integer age){
        this.name = name;
        this.age = age;
    }

    private User(){}

    public static User of(){
        return new User();
    }
}
```

调用

```java
public class Main{

    public static void main(String[] args){
        //没有@Accessors注解
        User user = User.of();
        user.setName("zhuji");
        user.setAge(26);    //User(name:zhuji, age:26)

        //有@Accessors(chain=true)注解
        User pangzi = User.of()
            .setName("pangzi")
            .setAge(2);     //User(name:pangzi, age:2)

        //有@Accessors(fluent=true)注解
        User shazi = User.of()
            .name("shazi")
            .age(1);        //User(name:shazi, age:1)
        /*
        不建议使用fluent=true,一般此类注解多用于POJO等需要序列化的类上,序列化依赖于getter/setter方法,
        如果使用fluent=true序列化时由于找不到get/set方法而失败,谨记!!
        */
    }
}
```

### @Builder注解

@Singular: 给类的集合属性添加一个单数的set方法,`本注解需要配合@Builder一起使用`

使用建造者模式创建类实例

```java
@Builder
public class BuilderExample {
  private String name;
  private int age;
  @Singular private Set<String> occupations;
}

```

```java
public class BuilderExample {
  private String name;
  private int age;
  private Set<String> occupations;

  BuilderExample(String name, int age, Set<String> occupations) {
    this.name = name;
    this.age = age;
    this.occupations = occupations;
  }

  public static BuilderExampleBuilder builder() {
    return new BuilderExampleBuilder();
  }

  public static class BuilderExampleBuilder {
    private String name;
    private int age;
    private java.util.ArrayList<String> occupations;

    BuilderExampleBuilder() {
    }

    public BuilderExampleBuilder name(String name) {
      this.name = name;
      return this;
    }

    public BuilderExampleBuilder age(int age) {
      this.age = age;
      return this;
    }

    public BuilderExampleBuilder occupation(String occupation) {
      if (this.occupations == null) {
        this.occupations = new java.util.ArrayList<String>();
      }
      this.occupations.add(occupation);
      return this;
    }

    public BuilderExampleBuilder occupations(Collection<? extends String> occupations) {
      if (this.occupations == null) {
        this.occupations = new java.util.ArrayList<String>();
      }
      this.occupations.addAll(occupations);
      return this;
    }

    public BuilderExampleBuilder clearOccupations() {
      if (this.occupations != null) {
        this.occupations.clear();
      }
      return this;
    }

    public BuilderExample build() {
      // complicated switch statement to produce a compact properly sized immutable set omitted.
      // go to https://projectlombok.org/features/Singular-snippet.html to see it.
      Set<String> occupations = ...;
      return new BuilderExample(name, age, occupations);
    }

    @java.lang.Override
    public String toString() {
      return "BuilderExample.BuilderExampleBuilder(name = " + this.name + ", age = " + this.age + ", occupations = " + this.occupations + ")";
    }
  }
}

```

调用

```Java
public class Main{
    public static void main(String[] args){
        BuilderExample example = BuilderExample.builder()
            .name("pangzi")
            .age(2)
            .build();   //BuilderExample(name:pangzi, age:1)

        //example.occupation("test1");    //BuilderExample(name:pangzi, age:2, occupations:[test1])

        example.occupations(List.of("test1","test2"));  //BuilderExample(name:pangzi, age:2, occupations:[test1, test2])
    }
}
```

### @Delegate 利用注解调用字段或方法返回值生成代理方法

`Any field or no-argument method can be annotated with @Delegate to let lombok generate delegate methods that forward the call to this field (or the result of invoking this method).`

```java
public class Main implements RestOperations{

    @Delegate
    private RestTemplate restTemplate;
    /*
    使用@Delegate让lombok使用restTemplate字段自动生成需要实现RestOperations的方法
    */
}
```

- tips

在项目根目录加入`lombok.comfig`配置文件可以自定义一些配置,例如`lombok.var.flagUsage=ALLOW`配置可以使用var,不需要声明类型`var a = 1`.其他的各种配置参考[官网](https://projectlombok.org/)