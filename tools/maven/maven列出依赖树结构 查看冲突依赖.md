
## mvn dependency:tree maven列出依赖树结构 查看冲突依赖 解决依赖冲突
摘抄自：https://blog.csdn.net/torpidcat/article/details/130881488

### 一、列出依赖树结构

命令行列出依赖结构


```
 mvn dependency:tree
```

![](assets/644e63e1fdf0e3be57f383d667bfd11e.png)


将依赖树结构打印到txt文件


```
 mvn dependency:tree>tree.txt
```

![](assets/9861fbf435bc9c666646fb064fabcad1.png)


将依赖树结构打印到txt文件 并对比出冲突


```
 mvn dependency:tree>tree.txt -Dverbose
```

![](assets/0fe269905f1edb08dba4cb3654439612.png)


### 二、解决 依赖包冲突问题


![](assets/b8c03fb2f08b8cf878b2b65b2a00a355.png)



参考


[https://www.cnblogs.com/kingsonfu/p/11800375.html](https://www.cnblogs.com/kingsonfu/p/11800375.html "https://www.cnblogs.com/kingsonfu/p/11800375.html")


[https://blog.csdn.net/qq_24712507/article/details/123658596](https://blog.csdn.net/qq_24712507/article/details/123658596 "https://blog.csdn.net/qq_24712507/article/details/123658596")

# idea 查看引入某个依赖的依赖树

![](assets/markdown-img-paste-20250611095546533.png)

![](assets/markdown-img-paste-20250611095715393.png)

