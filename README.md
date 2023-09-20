<p align="center">
	<img alt="logo" src="readme/img/Title.png" width="150%">
</p>
<p align="center">
    <center><strong>基于APT(Annotation Processing Tool)实现业务实体功能增强.提供包装类,便于自定义代码增强相关逻辑开发</strong></center>
</p>

## 业务痛点:

后端程序返回的数据中, 包含跳转Url / 资源Url等内容. 

- 我方网络环境与客户端网络环境存在差异, 我方可正确访问的资源,客户端不一定可以正确访问.

## 功能性需求:

- Request 中持有用户的访问域名
- 无侵入式的将该域名扩散到所有Response Url中

### 非功能需求:

- 可用性: 该修改过程必须对前端不可见
- 可维护性: 该修改过程不应占用过多系统资源
- 可扩展性: 该修改过程不易过度繁琐

## 方案对比:

经过调研, 最终可选方案为: Aop / JDKProxy / CglibProxy / Apt

### 各方案优缺点

| 方案       | 优点                                         | 缺点                                                         |
| ---------- | -------------------------------------------- | ------------------------------------------------------------ |
| Aop 增强   | 代码结构紧凑,修改过程由Spring维护,易读易扩展 | 替换逻辑复杂:Response结构复杂,需要深度遍历所有遍历,逻辑开销大 |
| JDK 代理   | 只需增强需要修改的对象,替换逻辑简单;         | 无形中多维护一套实体,需要修改所有初始化代码;需要定义额外接口; |
| Cglib 代理 | 不定义额外接口,只需修改特定对象,替换逻辑简单 | 存在额外性能开销, 需要修改所有初始化代码                     |
| Apt 增强   | 编译时期增强, 性能优越                       | 需要标注要修改的属性, 不可动态修改;存在学习成本              |

### 各方案性能

前置条件:

- 假设目标实体类中包含且仅包含唯一需要修改的字段
- 请求过程中没有额外业务逻辑 (创建对象后立即返回)
- 不考虑网络IO/文件IO等开销
- 每种方案指定 1000,000 次

> 实际业务场景中, 实体类相关操作逻辑只占了整个请求过程的一部分, 对比结果可能没有那么明显.

对比结果: 

<img src="readme/img/bar-background.svg" alt="bar-background" style="zoom:80%;" />

<center>各增强方案性能对比</center>

| 指标          | 无修改 | APT  | Aop  | JdkProxy | CglibProxy |
| ------------- | ------ | ---- | ---- | -------- | ---------- |
| 时间(单位:ms) | 149    | 148  | 3982 | 1359     | 1292       |

<font color=purple>**从上图我们可以发现, APT在性能上具有明显的优势**</font>

域名相关资源通过 ThreadLocal 封装在静态资源中, 满足编译期间获取; 遍历所有实体类, 为对应属性添加注解的开发成本在可容忍的范围内;

最终选择 APT 作为我方域名替换实现方案.

## 项目结构:

### APT设计类图:

![APTClassDesign](readme/img/APTClassDesign.png)

### 各模块功能:

- base_structure: 工具包, 负责具体域名替换逻辑 & 业务逻辑, 作为lib集成到业务程序中
- class_reinforce: 各方案实现代码
  - aop_scaffold: Aop增强方案相关代码(目前仅支持对最外层class增强)
  - apt_scaffold: APT增强方案相关代码(不支持对内部类进行增强, 感兴趣可以自己实现)
  - proxy_cglib_scaffold: cglib 代理方案相关代码
  - proxy_java_scaffold: jdk 代理方案相关代码

## 参考资料:

[Hannes Dorfmann--APT实战与精析](http://hannesdorfmann.com/annotation-processing/annotationprocessing101/)

[The Java Community Process(SM) Program - JSRs: Java Specification Requests - detail JSR# 199 (jcp.org)](https://jcp.org/en/jsr/detail?id=199)

[动态的Java - 无废话JavaCompilerAPI中文指南 - Meta-Interpretation (pfmiles.github.io)](http://pfmiles.github.io/blog/dynamic-java/))

[square/javapoet：用于生成.java源文件的Java API。 (github.com)](https://github.com/square/javapoet)

[projectlombok/lombok: Very spicy additions to the Java programming language. (github.com)](https://github.com/projectlombok/lombok)

[mplushnikov/lombok-intellij-plugin: Lombok Plugin for IntelliJ IDEA (github.com)](https://github.com/mplushnikov/lombok-intellij-plugin)

[jflex-de/jflex：Java的™快速扫描仪生成器，具有完整的Unicode支持 (github.com)](https://github.com/jflex-de/jflex)

IDEA插件在 plugin 分支

