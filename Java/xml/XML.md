# XML

> 注意：每个xml文件有且只有一个根节点

## 使用dmo4j包**xml**文件

​		book.xml文件

~~~xml
<?xml version="1.0" encoding="utf-8" ?>
<books>
    <book sn="1001">
        <name>时间简史</name>
        <price>56.3</price>
        <author>霍金</author>
    </book>
    <book sn="1002">
        <name>红楼梦</name>
        <price>88.5</price>
        <author>曹雪芹</author>
    </book>
</books>
~~~

​		Book类

~~~java
package xml.demo.code;


public class Book {
    private String sn;
    private String name;
    private double price;
    private String author;

    public Book() {
    }

    public Book(String sn, String name, double price, String author) {
        this.sn = sn;
        this.name = name;
        this.price = price;
        this.author = author;
    }

    //设置属性

    public String getSn() {
        return sn;
    }

    public void setSn(String sn) {
        this.sn = sn;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    // 设置tostring方法
    @Override
    public String toString() {
        return "book{" +
                "sn='" + sn + '\'' +
                ", name='" + name + '\'' +
                ", price=" + price +
                ", author='" + author + '\'' +
                '}';
    }
}
~~~



​		Dom4j测试文件

~~~~java
package xml.demo.code;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;
import org.junit.Test;
import java.util.List;

public class Dom4jTest {

    /**
     * 将xml文件的数据，装换为book类
     */
    @Test
    public void Test2() throws Exception {

        // 1.创建SAXReader流，读取xml文件
        SAXReader saxReader = new SAXReader();
        // 2.设置要读取xml文件的路径
        // 注：在junit测试中，相对路径要从模板名开始算
        Document read = saxReader.read("src/book.xml");

        // 3.拿到根节点
        Element rootElement = read.getRootElement();
        // 4.拿到根节点中所有的子节点，并遍历
        // 注：elements()是拿到节点集合，element()是拿到单个节点
        List<Element> elementList = rootElement.elements();
        for (Element element:elementList) {
            // 5.拼装Book类，并输出
            // 拿到element节点sn属性
            String sn = element.attributeValue("sn");
            // element("name")：拿到name节点元素
            // getStringValue()：拿到节点元素的值
            String name = element.element("name").getStringValue();
            String price = element.element("price").getStringValue();
            String author = element.element("author").getStringValue();
            Book book = new Book(sn, name, Double.parseDouble(price), author);
            System.out.println(book);
        }
    }
}
~~~~

