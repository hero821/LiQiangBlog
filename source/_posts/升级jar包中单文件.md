---
title: 升级jar包中单文件
date: 2020-10-01 14:00:00
tags:
- Java
categories:
- [Java]
---

# 发现问题

开发springboot项目的时候，我们经常会以jar包的形式进行部署。这个时候，如果需要修改某一个文件，那么该怎么升级呢？
- 本地重新打包，全包上传到服务器
- 只上传单文件到服务器，在服务器上把单文件合并到原来的jar包中

显然，第一种方案在网速较慢的时候，是影响工作效率的。

<!-- more -->

# 解决问题

## 过程

查看help：

```shell
[security@localhost jars]$ jar
用法: jar {ctxui}[vfm0Me] [jar-file] [manifest-file] [entry-point] [-C dir] files ...
选项包括: 
    -c  创建新的归档文件
    -t  列出归档目录
    -x  从档案中提取指定的 (或所有) 文件
    -u  更新现有的归档文件
    -v  在标准输出中生成详细输出
    -f  指定归档文件名
    -m  包含指定清单文件中的清单信息
    -e  为捆绑到可执行 jar 文件的独立应用程序
        指定应用程序入口点
    -0  仅存储; 不使用情况任何 ZIP 压缩
    -M  不创建条目的清单文件
    -i  为指定的 jar 文件生成索引信息
    -C  更改为指定的目录并包含其中的文件
如果有任何目录文件, 则对其进行递归处理。
清单文件名, 归档文件名和入口点名称的指定顺序
与 'm', 'f' 和 'e' 标记的指定顺序相同。

示例 1: 将两个类文件归档到一个名为 classes.jar 的归档文件中: 
       jar cvf classes.jar Foo.class Bar.class 
示例 2: 使用现有的清单文件 'mymanifest' 并
           将 foo/ 目录中的所有文件归档到 'classes.jar' 中: 
       jar cvfm classes.jar mymanifest -C foo/ .
```

## 结论

* 查看文件在jar包中的路径

```shell
[security@localhost jars]$ jar tvf service.jar |grep todo
  304 Fri Aug 23 12:00:00 CST 2019 BOOT-INF/classes/templates/todo.ftl
```

* 解压出文件

```shell
[security@localhost jars]$ jar xvf service.jar BOOT-INF/classes/templates/todo.ftl
  已解压: BOOT-INF/classes/templates/todo.ftl
```

* 覆盖解压出的文件

```shell
mv todo.ftl BOOT-INF/classes/templates/todo.ftl
```

* 合并文件到jar包中

```shell
[security@localhost jars]$ jar uvf service.jar BOOT-INF/classes/templates/todo.ftl
  正在添加: BOOT-INF/classes/templates/todo.ftl(输入 = 304) (输出 = 195)(压缩了 35%)
```

* 删除解压出的文件

```shell
[security@localhost jars]$ rm -rf BOOT-INF
```

* 重启服务
