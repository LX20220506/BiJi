# **插件汇总**

## GenerateAllSetter



快速生成get/set

实际的开发中，可能会经常为某个对象中多个属性进行 set 赋值，尽管可以用BeanUtil.copyProperties()方式批量赋值，但这种方式有一些弊端，存在属性值覆盖的问题，所以不少场景还是需要手动 set。如果一个对象属性太多 set 起来也很痛苦，GenerateAllSetter可以一键将对象属性都 set 出来。

快捷键：Alt+Enter



------

## Maven Helper



maven依赖解决

Maven Helper 是解决Maven依赖冲突的利器，可以快速查找项目中的依赖冲突。安装后打开pom文件，底部有 Dependency Analyzer 视图。显示红色表示存在依赖冲突，点进去直接在包上右键Exclude排除，pom文件中会做出相应排除包的操作。



- Conflicts(冲突)
- All Dependencies as List(列表形式查看所有依赖)
- All Dependencies as Tree(树结构查看所有依赖)，并且这个页面还支持搜索。



------

## Codota



编程提示

用了Codota 后不再怕对API不会用，举个栗子：当我们用stream().filter()对List操作，可是对filter()用法不熟，按常理我们会百度一下，而用Codota 会提示很多filter()用法，节省不少查阅资料的时间。



------

## Free MyBatis Plugin



mapper跳转到xml

在使用MyBatis 作为持久框架时有一个尴尬的问题：SQL xml文件和定义的Java接口无法相互跳转，不能像Java接口间调用那样，只能全局搜索稍显麻烦。Free MyBatis Plugin将两者之间进行关联。

![img](https://cdn.nlark.com/yuque/0/2021/png/2442600/1613877742910-8e8b7974-d154-4e1d-9f0a-3c3ecb8fe694.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_15%2Ctext_aGVsbG_nn6Xor4blupM%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

------

## Properties to YAML Converter



配置文件格式转换

将Properties 配置文件一键转换成YAML 文件，很实用的一个插件。**「注意：要提前备份原Properties 文件」**







## GsonFormatPlus

一个非常实用的插件，它可以将JSON字符串自动转换成Java实体类。

在和其他系统对接时，往往以`JSON`格式传输数据，而我们需要用`Java`实体接收数据入库或者包装转发，如果字段太多一个一个编写那就太麻烦了。

![img](https://cdn.nlark.com/yuque/0/2021/png/2442600/1626687694369-1addff68-4b40-4afd-828f-d65e9560f623.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_57%2Ctext_aGVsbG_nn6Xor4blupM%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)





## Properties to YAML Converter

这个插件可以将Properties 配置文件一键转换成YAML 文件，很实用的一个插件。

![img](https://cdn.nlark.com/yuque/0/2021/png/2442600/1626687721014-83b2c199-77e9-4103-9106-1c10a4f6ed22.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_27%2Ctext_aGVsbG_nn6Xor4blupM%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)







## Translation

Translation是一款非常好用的翻译插件，可以随时随地翻译单词、甚至一段话，不再需要额外打开浏览器搜索翻译网站了！

![img](https://cdn.nlark.com/yuque/0/2021/png/2442600/1626687773664-f8a2ad22-7453-4b57-b707-af4f84607b77.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_43%2Ctext_aGVsbG_nn6Xor4blupM%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)