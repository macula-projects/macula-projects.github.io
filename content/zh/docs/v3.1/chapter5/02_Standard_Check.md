---
type: docs
title: "代码审计规范"
linkTitle: "代码审计规范"
weight: 2
description: "本文介绍代码审计规范"
---

说明：在代码符合基本的代码规范外，在代码提交到版本控制前，需要进行更为严格的代码审计检查。

下面通过Eclipse IDE下的多个代码检查工具，指定代码必须符合的审计规范。

## Maven体系标准

对项目开发中的模块和项目，其文档结构，必须符合Maven标准，提供标准的pom.xml文件，并可通过Maven命令行执行的打包、测试、运行等。

## Eclipse Code Style

在Eclipse开发环境中，导入指定的代码模版，并按照代码模板格式化代码。

* Clean Up设置

    通过Eclipse菜单 Window -> Preference -> Java -> Clean Up -> Import
    
    选择cleanup.xml文件。
    
* Code Template设置

    通过Eclipse菜单 Window -> Preference -> Java -> Code Template -> Import
    
    选择codetemplates.xml文件。
    
* Code Formatter设置

    通过Eclipse菜单 Window -> Preference -> Java -> Formatter -> Import

    选择formatter.xml文件。
    
## Eclipse Check Style

通过Eclipse的CheckStyle插件，可以更加严格的检查Java代码的规范性。

在Eclipse -> Windows -> Preference -> checkstyle中导入checkstyle.xml ，进行代码检查。

## Eclipse FindBugs

FindBugs作为一种静态分析工具 ，它检查类或者 JAR 文件，将字节码与一组缺陷模式进行对比以发现可能的问题。有了静态分析工具，就可以在不实际运行程序的情况对软件进行分析，起到帮助检测程序潜在的编写问题。

在使用FindBugs检测的结果中，依据错误的等级，不允许出现“严重”级别的错误，并尽可能消除“一般”性错误。

## SVN Check In 规范

1. 绝不要提交没有编译过的代码，如果你不能编译，那别人也不能编译。
2. 避免提交一个未经测试的代码。
3. 提交前作一次更新和测试。如果你修改一个代码一段时间后，并在当前工作环境上进行了测试，并准备提交时，可能另外的开发人员同时也提交了他们的变更在同样的目录上。我们要针对升级后的版本进行测试。
4. 避免垃圾文件提交。 提交前，那些调试文件，中间文件非常容易忘记去除。
5. 一次提交的代码解决一个缺陷或新任务。当我们提交代码变更到仓库时，最好一次提交一个原子变更。
6. 公共分支上不要提交不完整的变更。依据变更集作原子提交是非常聪明的做法，如果一次变更分几次提交，那么原子回退这些变更比较困难。
7. 代码提交必须有本次提交说明，说明可能是开发的功能、解决的Bug等信息。

## JUnit测试覆盖规范
要求对主要的业务逻辑代码（即Service层），提供详细的JUnit测试，并要求测试覆盖率达到一定的百分比（具体可根据项目制定）。



