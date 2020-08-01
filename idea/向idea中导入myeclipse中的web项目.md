# 向idea中导入myeclipse中的web项目

## 导入
![](assets/markdown-img-paste-2020080111392568.png)

![](assets/markdown-img-paste-20200801114000997.png)

![](assets/markdown-img-paste-20200801114644996.png)

![](assets/markdown-img-paste-20200801114736564.png)

在Modules项的Sources标签页中配置项目结构：
![](assets/markdown-img-paste-20200801114923511.png)

在path中配置编译后文件输出路径，选择“Use module compile out path” -> 将Outputpath 和Test output path 都设置为classes文件夹：
![](assets/markdown-img-paste-20200801115057519.png)

编译快点：
![](assets/markdown-img-paste-20200801140208627.png)
![](assets/markdown-img-paste-20200801140225467.png)


->Artifacts->+号->WebApplication:Exploded->From Modules..
![](assets/markdown-img-paste-20200801140651448.png)
选择自己的项目

使用默认设置即可->Apply->ok
![](assets/markdown-img-paste-2020080114072299.png)

## 配置Tomcat容器
在菜单栏中Run -> Edit Configurations...
![](assets/markdown-img-paste-20200801140849185.png)

->+号->Tomcat Server->选择Local
![](assets/markdown-img-paste-20200801140905326.png)

![](assets/markdown-img-paste-20200801141129874.png)

在“Server”面板中，不勾选“After Launch”，设置“HTTP port”和“JMX port”（默认值即可），点击Apply -> OK，（左边列表中tomcat图标上小红叉是未部署项目的提示，部署项目后就会消失）。
![](assets/markdown-img-paste-20200801141245734.png)

## 需要运行
![](assets/markdown-img-paste-20200801142958395.png)

![](assets/markdown-img-paste-20200801143025882.png)

![](assets/markdown-img-paste-20200801143051408.png)

修改了再重新编译
![](assets/markdown-img-paste-2020080114313279.png)

![](assets/markdown-img-paste-20200801143459560.png)



