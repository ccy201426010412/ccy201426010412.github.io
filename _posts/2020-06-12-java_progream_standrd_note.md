
---
layout:     post
title:      Java开发规范学习笔记
subtitle:   Java开发规范华山版学习笔记
date:       2020-06-12
author:     王鹏程
header-img: img/post-bg-ios10.jpg
catalog: true
tags:
    - Java
    - 编译管理
    - 读书笔记
---

# Java开发规范学习笔记

_参考链接:_
- [阿里云官网课程链接](https://edu.aliyun.com/course/417?spm=5176.10731460.0.0.280639f4xpKm4n)
- [Git 官方手册](https://github.com/chjw8016/alibaba-java-style-guide)
- [在线书籍下载地址](https://edu.aliyun.com/course/417/material/24016/download?spm=5176.10731542.0.0.4a8c52bb9jM9Ji)

## 一 编程规范

### 1.1 命名风格
1. 【强制】代码中的命名不能以下划线或者`$`符号开头或者结束
2. 代码中禁止使用拼音和英文混合方式
3. 类名强制使用驼峰风格；但是**DO/BO/DTO/VO/AO/PO/UID**等；([阿里巴巴Java开发手册中的DO、DTO、BO、AO、VO、POJO定义](https://www.cnblogs.com/EasonJim/p/7967999.html))
    - DO(Data Object):与数据库表结构一一对应，通过DAO层向上传输数据源对象。
    - DTO（ Data Transfer Object）：数据传输对象，Service或Manager向外传输的对象。
    - BO（ Business Object）：业务对象。 由Service层输出的封装业务逻辑的对象。
    - AO（ Application Object）：应用对象。 在Web层与Service层之间抽象的复用对象模型，极为贴近展示层，复用度不高。
    - VO（ View Object）：显示层对象，通常是Web向模板渲染引擎层传输的对象。
    - POJO（ Plain Ordinary Java Object）：在本手册中， POJO专指只有setter/getter/toString的简单类，包括DO/DTO/BO/VO等。
    - Query：数据查询对象，各层接收上层的查询请求。 注意超过2个参数的查询封装，禁止使用Map类来传输。
4. 方法名、参数名、成员变量、局部变量统一使用驼峰式命名方法
5. `static final`的常量命名全部大写，单词间用下划线隔开；力求语义表达完整清楚，不要嫌名字长；例如`MAX_COUNT`
6. 抽象类名使用Abstract或者Base开头；异常类命名使用Exception 结尾；测试以Test结尾
7. 数组中括号需要前置
8. POJO类中布尔类型变量都不要加is前缀。否则设置set/get函数的时候容易引起冲突。
9. 包名统一小写，自然语义之间使用`.`号进行隔开；类名统一首字母大写；可以使用复数形式
10. 避免在子类父类的成员变量之间或者不同的代码块的局部变量之间采用相同的命名，使可读性降低。
11. 在常量和变量的命名时，表示类型的名词放在词尾，以提升辨识度。
12. 如果模块、接口、类、方法使用了设计模式，在命名时需要体现；例如`OrderFactory`
13. 接口类中的方法和属性不要添加任何修饰符号。尽量不要在接口里面定义变量
14. 对于Service和DAO类，基于SOA的理念，暴露出来的服务一定是接口，内部的实现类使用Impl的后缀与接口区别
15. 枚举类名带上Enum后缀，枚举成员名需要全大写单词间使用下划线隔开
16. 各层命名约规
    - Service/DAO层方法命名规约 
        1. 获取单个对象的方法使用get做前缀
        2. 获取多个对象的方法使用list做前缀，复数形式如结尾，如：listObjects;
        3. 获取统计值的方法使用count做前缀
        4. 插入的方法用save/insert做前缀
        5. 删除的方法使用remove/delete做前缀
        6. 修改的方法使用update做前缀
    - 领域模型命名规约
        1. 数据对象：xxxDo,xxx为数据表名；如UserDO
        2. 数据传输对象:xxxDTO,xxx为业务领域相关的名称。如: userDTO
        3. 展示对象:xxxVO,xxx一般为网页名称
        4. POJO 时DO/DTO/BO/VO的统称，禁止命名使用

## 1.2 常量定义:
1. **不允许任何魔法值（即未经预先定义的常量）直接出现在代码中。**
2. **在long或者Long赋值时，数值后使用大写的L，不能是小写的l，小写容易跟数 字1混淆，造成误解。**
3. 常量复用的层次结构:常量的复用层次有5层:
    - 跨应用共享常量:放置在二方库中，通常在constant目录下
    - 应用内共享常量:放置在一方库中，通常时子模块中的constant目录下
    - 子工程内部共享常量：在当前子工程的constant目录下
    - 包内共享常量:在当前包下单独的constant目录下
    - 类共享常量:直接在类内部private static final 定义。
    - 如果变量值仅在一个固定范围内变化用enum类型来定义。 

### 1.3 代码格式
1. **如果是大括号内为空，则简洁地写成{}即可，大括号中间无需换行和空格；如果是非 空代码块则**： 
    1. 左大括号前不换行。  
    2. 左大括号后换行。
    3. 右大括号前换行。  
    4. 右大括号后还有 else 等代码则不换行；表示终止的右大括号后必须换行。
2. 左右括号和字符之间不出现空格。左大括号前需要空格；



