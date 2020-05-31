# 奇异果

<a name="DCl3L"></a>
# 一、简介
<a name="xZCiV"></a>
## 1、解决的问题
所有测试能力服务化后，通过http协议作为基础，书写所有的业务用例。<br />用例内容采用json作为输入输出。<br />用例结果采用json结构diff+过滤的方式进行断言。<br />在这个情境下，用例管理需要一个方便的交互执行界面，此工具应运而生。<br />另外降低研发同学执行复现的成本。
<a name="B5O06"></a>
## 2、为什么使用idea插件的形式
（1）idea是一个普遍使用的ide工具平台。<br />（2）集团的代码检查插件等，在idea上有了极佳的实践<br />（3）idea插件市场有很高的下载量，方便更广泛的推广。如，之前发布的一个非常简单的小工具就达到了2w+的下载量，足见其传播力还是很惊人的。<br />![](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/8773/1589130243679-7eb3357b-0407-47a3-8734-cfc29454952d.png?x-oss-process=image%2Fresize%2Cw_1492#align=left&display=inline&height=790&margin=%5Bobject%20Object%5D&originHeight=790&originWidth=1492&status=done&style=none&width=1492)<br /> 
<a name="lG3Lk"></a>
# 二、插件安装
<a name="XwYXW"></a>
## 1、若采用本地安装

<a name="4nPfD"></a>
## 2、idea市场安装
打开链接：[https://plugins.jetbrains.com/plugin/14283-actinidia](https://plugins.jetbrains.com/plugin/14283-actinidia)
<a name="HoXhm"></a>
## 3、样例数据
见附件demo.zip
<a name="fnX1N"></a>
# 三、交互使用方法
<a name="rhX83"></a>
## 1、视频版本
见附件：actinidia.mp4
<a name="Uiyp6"></a>
## 2、截图版本

<a name="lDpWv"></a>
# 三、用例书写要求
<a name="yjYwP"></a>
## 1、命令文件
<a name="AfNK2"></a>
### （1）文件名
为“{序号}-{命令}-{用例说明等}.json”；<br />第一个短横线与第二个短横线之间的“命令”为约定的单词，如httpj、sql、poi等，用于触发对应的解析器；<br />当前版本，暂时只开放httpj命令。
<a name="84glQ"></a>
### （2）内容
json格式，说明当前http请求的相关信息，包括：
```
{
  ////http方法post或get，文件可以加注释，四个斜线开头
  "method": "post",
  ////操作日常环境数据库：使用域名poi-test.amap.test
  "url": "http://poi-test.amap.test/moc/command",
  ////header
  "header":{
    "cookie": "session=zhangyi"
  }
  ////queryString参数，json格式，可以为空
  "queryString": {
    "command": "sql"
  },
  ////body体，可以为字符串，或者json
  "body": {
   "xxxx":"xxxx"
  }
}
```
<a name="J2MtG"></a>
### （3）变量引用
场景：<br />-场景1，调用“A命令文件”返回json中有x变量，希望在调用“B命令文件”的时候使用到；<br />-场景2，A/B/C/D/E/F命令文件希望共用一个变量，希望统一提出来配置到一个地方；<br />方法：<br />-在“xxxJSON格式的文件”中设置一个变量<br />-在需要引用的文件中配置：“@文件名jsonpath@”<br />如：<br />@0000-caseId.json$.case_id@<br />其中，“0000-caseId.json”为文件名（目前仅支持当前目录下），“$.case_id”为要引用的变量的jsonpath。<br />可以配置在命令文件的任务位置，包括在字符串中间。程序处理的时候，通过正则进行匹配替换。<br />![](https://cdn.nlark.com/lark/0/2020/png/13080/1579591710954-5099c1b7-1798-4e3f-8ff6-6f229aadf92e.png#align=left&display=inline&height=1206&margin=%5Bobject%20Object%5D&originHeight=1206&originWidth=2962&status=done&style=none&width=2962)
<a name="SNXrP"></a>
## 2、结果文件
每个命令文件在执行后生成同名的".new.json/.old.json/.diff.json"文件；

<a name="kqMxS"></a>
### （1）".old.json"文件
如果已经存在此文件，不会生成，也不会被覆盖；<br />此文件用于存储“命令”的执行结果的对比“基线”；<br />文件可以手工提前配置；
<a name="d2dHt"></a>
### （2）".new.json"文件
每次命令被执行，都会生成此文件，如果存在则覆盖；<br />此文件用于跟".old.json"进行对比
<a name="rROMT"></a>
## 3、diff结果文件
<a name="HMbyd"></a>
### （1）内容来源
如果之前没有“.old.json”文件，会跑通过，因为对比使用的是最新生成的.old.json和.new.json文件；
<br />如果用自己的".old.json"文件，或者之前跑时生成的“.old.json”文件，两份json文件可能会有差异，差异会当作diff报出，写入到".diff.json"文件中。
<br />其中，“filter”是一个数组，每条都是一个diff，
<a name="B119T"></a>
### （2）字段含义
operation：新json相对于老json，发生的变化，取值为update、remove、add<br />path：新json相对于老json，发生变化的json路径（目前不是标准jsonpath格式，仅供参考）<br />value：新json中的结果<br />oldValue：老json中的结果
<a name="K4qFw"></a>
### （3）快速过滤掉符合预期的diff
如果（2）中的diff结果，即filter中的diff符合预期，比如是时时变化的时间戳等，可以将此filter数组，copy到当前报错的“命令”json根节点下即可，下次执行变会过滤掉，copy方法，如：

<a name="w8AFC"></a>
# 四、其他计划
1、支持模板文件创建<br />2、支持更多的报表输出
<a name="fvkRu"></a>
# 五、其他测试工具推荐


<a name="F11Xg"></a>
# 六、技术要点
1、idea的插件开发技术<br />2、其他ui
<br />所有ui控件中难度最大的应该是jtree，是数模分离的一个比较复杂的插件。
<br />核心在于对DefaultTreeModel的拓展。<br />3、jsondiff的能力<br />
参考：[https://github.com/eBay/bsonpatch](https://github.com/eBay/bsonpatch)
