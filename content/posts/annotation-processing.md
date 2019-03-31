---
title: "java注解处理器"
date: 2018-03-11T22:11:54+08:00
draft: false
---
最近发现周围的一些小伙伴开始像下面这样写DTO了
```java
import lombok.AccessLevel;
import lombok.Setter;
import lombok.Data;
import lombok.ToString;

@Data public class ExampleDTO {
  private final String name;
  @Setter(AccessLevel.PACKAGE) private int age;
  private double score;
  private String[] tags;
  
  @ToString(includeFieldNames=true)
  @Data(staticConstructor="of")
  public static class Exercise<T> {
    private final String name;
    private final T value;
  }
}
```
顺着引用的包路径找到lombok这个包，看了下[官网](https://projectlombok.org/)的介绍

Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.

看了下官网的介绍确实可以给开发者减少代码量（对于考核代码行的公司的开发这可能不喜欢它）

为了搞清楚Lombok到底是怎么工作的，借助谷歌大神找到了它底层的实现机制“注解处理器”，并找到两篇高质量的文章

[annotationprocessing101](http://hannesdorfmann.com/annotation-processing/annotationprocessing101)

[eliminate-boilerplate](https://academy.realm.io/posts/360andev-ryan-harter-eliminate-boilerplate/)

感兴趣的可以看看这两篇文章，基本也就可以实现自己的注解处理器了。

补充:[mapstruct](http://mapstruct.org/)这个也很好用，可以在实体类，DTO，VO之间转换，非常方便，同时可以集成spring。
[Notes-on-Java-Annotation-Processor](https://lotabout.me/2017/Notes-on-Java-Annotation-Processor/)






