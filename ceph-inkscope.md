# inkscope
inkscope是一个ceph的管理软件，界面还是挺好看的！  
但目前只支持到jewel之前的版本，已经几乎不更新了，换句话说，项目已经黄了。。。  

inkscope的主页：  
https://github.com/inkscope/inkscope
## 软件安装
centos 7上的安装过程参考如下网页：  
http://www.zphj1987.com/2016/04/19/inkscope%E5%AE%8C%E6%95%B4%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE/  
各节点上主要安装包（有些包会因为依赖关系自动安装好，不然也会在启动任务时报错，提示所需的依赖包）：  

|节点|rpm|
|:---|:---|
|inkscope web节点|inkscope-admviz, inkscope-monitor, mongodb, flask|
|mon|inkscope-cephprobe和inkscope-cephrestapi(一台上安装即可), inkscope-sysprobe|
|osd|inkscope-sysprobe|

对配置的修改参见上面的网页，没有问题

## 问题
- ceph status界面显示的信息不全
这是因为ceph使用的版本是Jewel，inkscope还不兼容！还和浏览器有关！经过合入master tag后的代码，firefox可以正常显示，而chrome则不行，不再搞下去了。。。问题说明详见下面的issue: 
https://github.com/inkscope/inkscope/issues/83

