---
title: JAVA处理xml
date: 2021-05-10 21:31:35
tags:
- xml
- JAVA
---

# XML

## 1. XML结构

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE note SYSTEM "book.dtd">
<book id="1">
    <name>Java核心技术</name>
    <author>Cay S. Horstmann</author>
    <isbn lang="CN">1234567</isbn>
    <tags>
        <tag>Java</tag>
        <tag>Network</tag>
    </tags>
    <pubDate/>
</book>
```

首行必定是`<?xml version="1.0"?>`，可以加上可选的编码。

`<!DOCTYPE note SYSTEM "book.dtd">`声明的是文档定义类型（DTD：Document Type Definition），DTD是可选的。

一个XML文档有且仅有一个根元素，根元素可以包含任意个子元素，元素可以包含属性，例如，`<isbn lang="CN">1234567</isbn>`包含一个属性`lang="CN"`。

由于使用了`<`、`>`以及引号等标识符，如果内容出现了特殊符号，需要使用`&???;`表示转义。

| 字符 | 表示   |
| :--- | :----- |
| <    | `&lt; `  |
| >    | `&gt;`   |
| &    | `&amp; ` |
| "    | `&quot;` |
| '    | `&apos;` |

DTD文档可以指定一系列规则，例如：

- 根元素必须是`book`
- `book`元素必须包含`name`，`author`等指定元素
- `isbn`元素必须包含属性`lang`
- ...

如何验证XML文件的正确性呢？最简单的方式是通过浏览器验证。可以直接把XML文件拖拽到浏览器窗口，如果格式错误，浏览器会报错。

## 2. DOM

因为XML是一种树形结构的文档，它有两种标准的解析API：

- DOM：一次性读取XML，并在内存中表示为树形结构；
- SAX：以流的形式读取XML，使用事件回调。

如果解析为DOM结构，它大概长这样：

![image-20210510215159229](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210510215159229.png)

Java提供了DOM API来解析XML，它使用下面的对象来表示XML的内容：

- Document：代表整个XML文档；
- Element：代表一个XML元素；
- Attribute：代表一个元素的某个属性。

使用DOM API解析一个XML文档的代码如下：

```java
InputStream input = Main.class.getResourceAsStream("/book.xml");
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(input);
```

`DocumentBuilder.parse()`用于解析一个XML，它可以接收InputStream，File或者URL，如果解析无误，我们将获得一个Document对象，这个对象代表了整个XML文档的树形结构，需要遍历以便读取指定元素的值：

```java
void printNode(Node n, int indent) {
    for (int i = 0; i < indent; i++) {
        System.out.print(' ');
    }
    switch (n.getNodeType()) {
    case Node.DOCUMENT_NODE: // Document节点
        System.out.println("Document: " + n.getNodeName());
        break;
    case Node.ELEMENT_NODE: // 元素节点
        System.out.println("Element: " + n.getNodeName());
        break;
    case Node.TEXT_NODE: // 文本
        System.out.println("Text: " + n.getNodeName() + " = " + n.getNodeValue());
        break;
    case Node.ATTRIBUTE_NODE: // 属性
        System.out.println("Attr: " + n.getNodeName() + " = " + n.getNodeValue());
        break;
    default: // 其他
        System.out.println("NodeType: " + n.getNodeType() + ", NodeName: " + n.getNodeName());
    }
    for (Node child = n.getFirstChild(); child != null; child = child.getNextSibling()) {
        printNode(child, indent + 1);
    }
}
/*
Document: #document
 Element: book
  Text: #text = 
    
  Element: name
   Text: #text = Java核心技术
  Text: #text = 
    
  Element: author
   Text: #text = Cay S. Horstmann
  Text: #text = 
  ...
  */
```

## 3. SAX

SAX解析会触发一系列事件：

- startDocument：开始读取XML文档；
- startElement：读取到了一个元素，例如`<book>`；
- characters：读取到了字符；
- endElement：读取到了一个结束的元素，例如`</book>`；
- endDocument：读取XML文档结束。

如果我们用SAX API解析XML，Java代码如下：

```java
InputStream input = Main.class.getResourceAsStream("/book.xml");
SAXParserFactory spf = SAXParserFactory.newInstance();
SAXParser saxParser = spf.newSAXParser();
saxParser.parse(input, new MyHandler());
```

关键代码`SAXParser.parse()`除了需要传入一个`InputStream`外，还需要传入一个回调对象，这个对象要继承自`DefaultHandler`：

```java
class MyHandler extends DefaultHandler {
    public void startDocument() throws SAXException {
        print("start document");
    }

    public void endDocument() throws SAXException {
        print("end document");
    }

    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        print("start element:", localName, qName);
    }

    public void endElement(String uri, String localName, String qName) throws SAXException {
        print("end element:", localName, qName);
    }

    public void characters(char[] ch, int start, int length) throws SAXException {
        print("characters:", new String(ch, start, length));
    }

    public void error(SAXParseException e) throws SAXException {
        print("error:", e);
    }

    void print(Object... objs) {
        for (Object obj : objs) {
            System.out.print(obj);
            System.out.print(" ");
        }
        System.out.println();
    }
}
```

运行SAX解析代码，可以打印出下面的结果：

```
start document
start element:  book
characters:
     
start element:  name
characters: Java核心技术
end element:  name
characters:
     
start element:  author
...
```

如果要读取`<name>`节点的文本，我们就必须在解析过程中根据`startElement()`和`endElement()`定位当前正在读取的节点，可以使用栈结构保存，每遇到一个`startElement()`入栈，每遇到一个`endElement()`出栈，这样，读到`characters()`时我们才知道当前读取的文本是哪个节点的。可见，使用SAX API仍然比较麻烦。

## 4. Jackson

一个名叫Jackson的开源的第三方库可以轻松做到XML到JavaBean的转换。我们要使用Jackson，先添加两个Maven的依赖：

- com.fasterxml.jackson.dataformat:jackson-dataformat-xml:2.10.1
- org.codehaus.woodstox:woodstox-core-asl:4.4.1

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<book id="1">
    <name>Java核心技术</name>
    <author>Cay S. Horstmann</author>
    <isbn lang="CN">1234567</isbn>
    <tags>
        <tag>Java</tag>
        <tag>Network</tag>
    </tags>
    <pubDate/>
</book>
```

定义好JavaBean

```java
public class Book {
    public long id;
    public String name;
    public String author;
    public String isbn;
    public List<String> tags;
    public String pubDate;
}
```

解析：

```java
InputStream input = Main.class.getResourceAsStream("/book.xml");
JacksonXmlModule module = new JacksonXmlModule();
XmlMapper mapper = new XmlMapper(module);
Book book = mapper.readValue(input, Book.class);
System.out.println(book.id);
System.out.println(book.name);
System.out.println(book.author);
System.out.println(book.isbn);
System.out.println(book.tags);
System.out.println(book.pubDate);
```

注意到`XmlMapper`就是我们需要创建的核心对象，可以用`readValue(InputStream, Class)`直接读取XML并返回一个JavaBean。

