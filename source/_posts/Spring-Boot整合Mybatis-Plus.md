---
title: Spring Boot整合Mybatis Plus
author: 争夕
date: 2018-08-04 16:33:24
categories:
- 后端
- SpringBoot
tags:
- SpringBoot
- Mybatis
---
> 前面演示了很多整合案例，今天笔录下整合Mybatis Plus的过程，后面文章涉及整合框架组件的文章，会大部分基于Spring Boot来做，因为上一篇文章我们也理解了，Spring Boot已经将整个生态链集成达到了比较规范的程度。

Mybatis Plus
------------
我们知道Mybatis是一款非常优秀的JDBC持久化框架，利用它我们可以很方便的搭建DAO实现，而Mybatis Plus则是对Mybatis的更进一步增强，在 Mybatis 的基础上只做增强不做改变，为简化开发、提高效率而生。这是官方的解释。

gradle配置
--------
mybatis plus团队对集成Spring Boot作了适配，制作了一个starter，starter jar中包含了自动配置，以及依赖了其它 mybatis plus相关包、mybatis、Spring-boot-starter-jdbc、Spring-boot-autoconfigure。
而只需引入以下即可：
```
compile "com.baomidou:mybatis-plus-boot-starter:2.3"
```
starter很强大，这也是 Spring boot面向特定功能，快速引入一系列jar广泛使用的方式。

application.yml
---------------

```
mybatis-plus:
  # 如果是放在src/main/java目录下 classpath:/com/yourpackage/*/mapper/*Mapper.xml
  # 如果是放在resource目录 classpath:/mapper/*Mapper.xml
  mapper-locations: classpath:/**/*Mapper.xml
  #实体扫描，多个package用逗号或者分号分隔
  typeAliasesPackage: org.wbw.**.entity
  global-config:
    #主键类型  0:"数据库ID自增", 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
    id-type: 3
    #字段策略 0:"忽略判断",1:"非 NULL 判断"),2:"非空判断"
    field-strategy: 2
    #驼峰下划线转换
    db-column-underline: true
    #mp2.3+ 全局表前缀 mp_
    #table-prefix: mp_
    #刷新mapper 调试神器
    #refresh-mapper: true
    #数据库大写下划线转换
    #capital-mode: true
    # Sequence序列接口实现类配置
    key-generator: com.baomidou.mybatisplus.incrementer.OracleKeyGenerator
    #逻辑删除配置（下面3个配置）
    logic-delete-value: 1
    logic-not-delete-value: 0
    sql-injector: com.baomidou.mybatisplus.mapper.LogicSqlInjector
    #自定义填充策略接口实现
    #meta-object-handler: com.baomidou.springboot.MyMetaObjectHandler
  configuration:
    #配置返回数据库(column下划线命名&&返回java实体是驼峰命名)，自动匹配无需as（没开启这个，SQL需要写as： select user_id as userId）
    map-underscore-to-camel-case: true
    cache-enabled: false
    #配置JdbcTypeForNull, oracle数据库必须配置
    jdbc-type-for-null: 'null'
  type-enums-package: org.wbw.**.type
```

额外的javaConfig定义扫描Mapper包，或根据需要使用分布插件：
-------------------------------------

```java
package org.wbw.mybatisplus.config;

import com.baomidou.mybatisplus.plugins.PaginationInterceptor;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@MapperScan(basePackages = "org.wbw.**.mapper")
public class MybatisPlusConfig {

    /*
     * 分页插件，自动识别数据库类型
     * 多租户，请参考官网【插件扩展】
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

启动类：
----

```java
package org.wbw.mybatisplus;


import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.wbw.mybatisplus.entity.Blog;
import org.wbw.mybatisplus.mapper.BlogMapper;

@SpringBootApplication
public class MybatisPlusSpringBootApp {

    public static void main(String[] args) {

        // 启动Spring boot应用
        ApplicationContext applicationContext = SpringApplication.run(MybatisPlusSpringBootApp.class, args);

        // 启动完成之后，拿到Mapper访问下数据库
        testQuery(applicationContext);

    }

    private static void testQuery(ApplicationContext applicationContext) {
        BlogMapper blogMapper = applicationContext.getBean(BlogMapper.class);
        Blog blog = blogMapper.selectBlog(6);
        System.out.println(blog);
    }

}
```

OK大功造成，是不是比之前用Spring配置简单多了？
