MSpider
==========
基于词频密度过滤、利用百度、谷歌、搜搜、360搜索4个引擎为种子来源的多线程爬虫，结果存入mysql。用到了jsoup和webclient。

- 原理****
-
**1. 过滤算法**

　　过滤关联度不大的网址，避免爬虫盲目搜索。目前只用到词频密度对网址和域名进行打分，在任务堆积较多(超过总队列长度90%)时，过滤掉相对评价分数小的网址。打算下一步得到关键词在全文的分布向量，用熵权系数法来比较各个关键词的分布情况，进一步优化过滤算法。原理和把纸撕成碎片然后对边缘进行拼合求关联度一样，关键词分布就像纸片的边缘，可以当做特征向量来求两两之间的商权系数，判断各关键词之间存在什么关系。(不过目前用词频密度过滤结果还算不错，大家可以试试看)


**2. 整体结构**

　　分为引擎搜索线程、爬虫线程和垃圾回收线程、数据库模块和任务队列4部分。
　　![](https://raw.githubusercontent.com/wo4li2wang/MSpider/master/pic/pic2.jpg)


　　**引擎搜索线程**采用单例模式，可以利用上述4中搜索引擎输入关键字得到结果，并将结果以一种特殊的数据结构放入任务队列中。引擎搜索线程用于获得爬虫种子，它受垃圾回收线程的控制。在队列任务不多的情况下利用site:host语句在各个引擎中搜索模型中host评分最高的网址，获得关联度更强的种子供爬虫使用。

　　**爬虫线程**的数量是可修改的(下一步打算结合内存和算法让其数量动态变化)。它们不断向任务队列里申请任务，得到分配后爬取网页，调用过滤算法，将有价值的结果放入队列并调用数据库模块写入数据库。

　　**垃圾回收线程**是后台线程，单例模式，时刻监控任务队列的情况，并在任务较多时清理垃圾，任务较少时启动引擎搜索线程获得更多爬虫种子。平时则关闭引擎搜索线程。

　　**数据库模块**采用单例模式，我没有用ThreadLocal(因为我的小电脑顶多运行8只爬虫，读写操作没那么频繁)。用同步块来执行读写操作。接受引擎搜索线程和爬虫线程的读写请求。它以host作为分类准则，对连接进行分类处理。不同的host存入不同的表内。

　　**任务队列**其实不是队列，它是对TreeSet的改造，还是采用单例模式╮(╯▽╰)╭，存放特殊数据结构的任务，供爬虫们使用。同时它也负责爬虫请求的任务分配。


**3. 执行过程**

　　1)调用引擎搜索线程run方法(注意不是start启动)初始化，从4个引擎上得到一定数量的种子

　　2)分别启动垃圾回收线程和所有爬虫线程，后续线程控制完全归垃圾回收线程控制。当然可以通过暂停按钮让所有线程阻塞。

 
 
- 使用方法
-
启动mysql，输入create database mspider;

启动 com.td1madao.gui.MyFrame 即可启动GUI程序，也可以用
com.td1madao.gui.NoGui命令行启动

gui效果如图

![](https://raw.githubusercontent.com/wo4li2wang/MSpider/master/pic/pic.jpg)

在数据库里的效果如图(这里用MYSQL FRONT显示)

![](https://raw.githubusercontent.com/wo4li2wang/MSpider/master/pic/pic3.jpg)


如果觉得功能不够，可以在com.td1madao.global.GlobalVar下修改默认配置，配置里提供了host黑名单、线程数量、算法等级、请求次数、任务队列长度等等，因为我不是很会做GUI，所以就没有在图形界面里实现这些功能。
<br /><br />这个引擎的制作完全是出自个人的兴趣，程序结构、模型设计，算法设计和代码实现均个人完成，有的地方代码还不是很规范，很多设计模式也没有灵活运用上，希望大家不要笑话我o(╯□╰)o<br /><br /><br /><br />
　　ps：在github上无耻的做个小广告。我是大三本科生，想在这个暑假去公司实习，希望哪位学长或学姐愿意给我一些建议或者传授我一些技巧。JAVA WEB、安卓或者其他后台方向我的实习我都应该没问题。我参加过数学建模的全国赛和全美赛，获过奖，算法相关的职务我也应该没问题。(我不会PHP和PYTHON，Linux也是新手，如果技能要求特别苛刻的公司实习就算了……)
我的<p>邮箱:wo4li2wang@163.com<p>qq:360810498<p>希望大神们能给我提供个机会，或者对我毕业提供一些学习指导，先感谢了。
