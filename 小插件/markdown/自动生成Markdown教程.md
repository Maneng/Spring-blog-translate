# 自动生成Markdown教程


## 安装


需要已经安装node.js

命令行输入
> npm install -g doctoc


## 使用

`cd`到markdown文件(例如ex1.md)所在目录，然后命令行使用

> $ doctoc ex1.md

如果想给这个目录和所有子目录的文件都添加文档目录，

> $ doctoc .

## 使用效果

![](/img/toc.png)


## 注意事项

> 标题如果有'.'等符号，生成的目录可能无法正确链接！