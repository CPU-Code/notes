## XML概述



*   XML（Extensible Markup Language）

    可扩展的标识语言

    数据传输的一种格式

    树形结构（节点）

*   优点

    解析的更快 : 使用标签语言，能够很快定位

    占用空间小 : 纯文本格式

    可读性强 : 带有名字的标签我们都很喜欢

    跨平台性 : 只要有解析的库文件，无论什么系统，什么语言都可以无障碍的通信

    流行 : 现在最流行的一种数据传输格式





XML的格式

*   所有元素都须有关闭标签

*   标签对大小写敏感

*   标签没有被预定义（自己命名）

*   第一句格式必须为 `<?xml version="1.0"?>`

*   文档必须有根元素的单一标签对

*   每个标签都可以有自己的属性

*   通常不需要手动创建XML



XML的应用场合

*   WEB客户端和服务器数据的格式

*   广泛作为网络之间的数据格式

*   可以在互不兼容的系统间交换数据

*   对数据库支持不好的系统中用XML保存数据

*   用来作为配置文档

*   能够用来显示您的技术很高深（~0~）





## XML解析-WEB



![image-20200907101319942](https://gitee.com/cpu_code/picture_bed/raw/master//20200907101327.png)



CGI读取XML

*   open -> read -> printf

JS解析XML

*   获取XML文档

    `var XMLDoc = XMLHttpRequest.responseXML;`

*   获取文档中内容

    `XMLDoc.getElementsByTagName("blackbeetle")；`





##XML解析-C语言



相关类型

*   xmlDocPtr代表一个XML文档的指针类型

*   xmlNodePtr代表一个节点的指针类型

打开XML

*   `xmlDocPtr xmldoc=xmlParseFile(“file_name”);`

获得根节点

*   `xmlNodePtr cur=xmlDocGetRootElement(xmldoc);`

*   `cur=cur->xmlChildrenNode;//子节点集合`

*   `cur=cur->next; //下一个字节点`



其他常用解析函数

*   char* value = (char*)xmlNodeGetContent(cur)；//得到节点中的信息

*   xmlFree(value);	//free得到的数据（必须free）

*   xmlFreeDoc(doc);	//释放xml解析库所用资源

*   xmlKeepBlanksDefault(0);	//去除空格

*   xmlCleanupParser();

*   xmlStrcmp(curNode->name, BAD_CAST "root")；	//字符串比较 BAD_CAST 宏定义 强制转为XML字符串



## XML生成-C语言



生成版本号1.0

*    xmlDocPtr doc = xmlNewDoc(BAD_CAST"1.0");

创建新节点

*    root = xmlNewNode(NULL, BAD_CAST"rfid_class");

配置根节点

*   xmlDocSetRootElement(doc, root);

添加子节点

*   xmlAddChild(root, node);

为节点添加节点名以及内容

*   xmlNewTextChild(node, NULL, BAD_CAST  “class_name”, BAD_CAST(“text”));

保存文件

*    xmlSaveFormatFileEnc(xml_name, doc, "utf-8", 1);

*   xmlSaveFile(xml_name, doc);

释放文档内节点动态申请的内存

*   xmlFreeDoc(doc);



## libXML的移植



libxml 是一个xml的c语言版的解析器



