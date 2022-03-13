---
title: 抓取豆瓣编程书籍
date: 2017-12-22T14:07:46+08:00
lastmod: 2020-11-11T16:31:07+08:00
slug: 47e563a8378c52adb3d211e42474beaa
draft: false
categories: [java]
tags: [spider]
keywords: spider, java
description: spider douban books by java
---
# Something needed before action
```text
    注意:
        lombok不仅需要导入包,还需要idea安装lombok插件
```
<!-- more -->

```xml
    <dependencies>
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>18.0</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.18</version>
        </dependency>

        <dependency>
            <groupId>org.jsoup</groupId>
            <artifactId>jsoup</artifactId>
            <version>1.8.3</version>
        </dependency>

        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.3</version>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.17</version>
        </dependency>
    </dependencies>
```
# mind mapping

- 进行IP代理(未使用代理,http://www.xicidaili.com/ 找不到稳定可用的代理)
- 通过HttpClient获取到请求页面的String字符串
- 通过jsoup解析
- (解析需要自己在页面查看源代码,分析DOM结构)
- (通过使用jsoup的类似于css选择器的函数,获取元素,元素集,或者文本和属性值)
- 每一本书的值set进Book实体,并添加进List集合
- 获取页面底部的总页码数
- 循环创建线程(一个页面,一个线程)
- List集合通过构造方法共享
- 运行结束后,应该获取到的是一个拥有所有页面的书的集合
- 根据score属性及num属性,实现Comparator接口,完成排序
- 遍历当前这个List集合,顺序为每个元素设置id属性
- 调用poi,遍历List,将每个元素按行写入excel文件

# In action
```text
    Book实体以及Comparator实现类
```
```java
package individual.cy.douban.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.ToString;

/**
 * Created with IntelliJ IDEA.
 *
 * @author: mystic
 * @date: 2017/12/21 8:54
 * @since: JDK1.8.0_144
 * @version: X
 * Description:
 */
@Data
@ToString
@AllArgsConstructor
public class Book {
    /**
     * 序号
     */
    private String id;
    /**
     * 书籍名称
     */
    private String name;
    /**
     * 书籍评分
     */
    private String score;
    /**
     * 评价人数
     */
    private String num;
    /**
     * 作者
     */
    private String author;
    /**
     * 作者
     */
    private String press;
    /**
     * 出版日期
     */
    private String date;
    /**
     * 价格
     */
    private String price;

}
```
```java
package individual.cy.douban.pojo;

import java.util.Comparator;

/**
 * Created with IntelliJ IDEA.
 *
 * @author: mystic
 * @date: 2017/12/21 20:06
 * @since: JDK1.8.0_144
 * @version: X
 * Description:
 */
public class BookComparator implements Comparator<Book> {
    /**
     *      降序: 优先评分,人数次之
     * @param book2
     * @param book1
     * @return
     */
    @Override
    public int compare(Book book2, Book book1) {
        String temp1 = book1.getScore();
        String temp2 = book2.getScore();
        if (temp1 == null || "".equals(temp1)) {
            temp1 = "0";
        }
        if (temp2 == null || "".equals(temp2)) {
            temp2 = "0";
        }
        // 评分排序优先
        double num1 = Double.parseDouble(temp1);
        double num2 = Double.parseDouble(temp2);
        int result = Double.compare(num1, num2);
        if (result == 0) {
            String temp3 = book1.getNum();
            String temp4 = book2.getNum();
            if (temp3 == null || "".equals(temp3)) {
                temp3 = "0";
            }
            if (temp4 == null || "".equals(temp4)) {
                temp4 = "0";
            }
            // 评分相同,则以评价人数排序
            double num3 = Double.parseDouble(temp3);
            double num4 = Double.parseDouble(temp4);
            result = Double.compare(num3, num4);
        }
        return result;
    }
}
```
```text
    Spider,抓取指定url的页面字符串
```
```java
package individual.cy.douban.utils;

import org.apache.http.HttpEntity;
import org.apache.http.HttpHost;
import org.apache.http.ParseException;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.IOException;

/**
 * Created with IntelliJ IDEA.
 *
 * @author: mystic
 * @date: 2017/12/21 8:33
 * @since: JDK1.8.0_144
 * @version: X
 * Description:
 */
public class Spider {
    public static String pickData(String url) {
        try (CloseableHttpClient client = HttpClients.createDefault()) {
            HttpGet httpGet = new HttpGet(url);
            CloseableHttpResponse response = client.execute(httpGet);
            // 获取响应实体
            HttpEntity entity = response.getEntity();
            // 打印响应状态
            if (entity != null) {
                return EntityUtils.toString(entity);
            }
        } catch (ParseException | IOException e) {
            e.printStackTrace();
            return "";
        }
        return "";
    }

    /**
     * 使用本机ip进行获取数据
     * @param url
     * @return
     */
    public static String pick4data(String url) {
        //设置超时处理
        RequestConfig config = RequestConfig.custom().setConnectTimeout(3000).
                setSocketTimeout(3000).build();
        HttpGet httpGet = new HttpGet(url);
        return grab(httpGet,config);
    }

    /**
     * 使用代理ip进行获取数据
     * @param url
     * @param ip
     * @param port
     * @return
     */
    public static String pick4data(String url, String ip, String port) {
        //设置代理访问和超时处理
        System.out.println("此时线程: " + Thread.currentThread().getName() + " 爬取所使用的代理为: "
                + ip + ":" + port);
        HttpHost proxy = new HttpHost(ip, Integer.parseInt(port));
        RequestConfig config = RequestConfig.custom().setProxy(proxy).setConnectTimeout(3000).
                setSocketTimeout(3000).build();
        HttpGet httpGet = new HttpGet(url);
        return grab(httpGet,config);
    }

    private static String grab(HttpGet httpGet, RequestConfig config){
        httpGet.setConfig(config);
        httpGet.setHeader("Accept", "text/html,application/xhtml+xml,application/xml;" +
                "q=0.9,image/webp,*/*;q=0.8");
        httpGet.setHeader("Accept-Encoding", "gzip, deflate, sdch");
        httpGet.setHeader("Accept-Language", "zh-CN,zh;q=0.8");
        httpGet.setHeader("Cache-Control", "no-cache");
        httpGet.setHeader("Connection", "keep-alive");
        httpGet.setHeader("Host", "www.xicidaili.com");
        httpGet.setHeader("Pragma", "no-cache");
        httpGet.setHeader("Upgrade-Insecure-Requests", "1");
        httpGet.setHeader("User-Agent", "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 " +
                "(KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36");

        try (CloseableHttpClient httpClient = HttpClients.createDefault();
             //客户端执行httpGet方法，返回响应
             CloseableHttpResponse httpResponse = httpClient.execute(httpGet)) {
            //得到服务响应状态码
            int status = 200;
            if (httpResponse.getStatusLine().getStatusCode() == status) {
                HttpEntity entity = httpResponse.getEntity();
                if (entity != null) {
                    return EntityUtils.toString(entity, "utf-8");
                }
            }
        } catch (ParseException | IOException e) {
            e.printStackTrace();
            return "";
        }
        return "";
    }
}
```
```text
    解析页面数据,并添加至List<Book>集合
```
```java
package individual.cy.douban.web;

import individual.cy.douban.pojo.Book;
import individual.cy.douban.utils.Spider;
import lombok.Getter;
import lombok.Setter;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

import java.util.List;
import java.util.Vector;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * Created with IntelliJ IDEA.
 *
 * @author: mystic
 * @date: 2017/12/21 9:29
 * @since: JDK1.8.0_144
 * @version: X
 * Description:
 */
public class GrabDouban implements Runnable {

    @Getter
    @Setter
    private List<Book> books = new Vector<>();

    @Getter
    @Setter
    private String url;

    private GrabDouban(){}

    public GrabDouban(List<Book> books){
        this.books = books;
    }

    @Override
    public void run() {
        System.out.println("url = " + url);
        System.out.println("Thread.currentThread().getName() = " + Thread.currentThread().getName());
        parse(url);
    }

    private void parse(String url) {
        String html = Spider.pickData(url);
        /*String html = Spider.pick4data(url,"220.249.185.178","9999");*/
        Document doc = Jsoup.parse(html);
        Elements elements = doc.select("ul.subject-list li.subject-item div.info");
        for (Element element : elements) {
            String name = element.select("h2 a").attr("title");
            // pub和books变量需要被锁
            synchronized (GrabDouban.class){
                String[] pub = element.select("div.pub").text().split("/");
                // 译者或审校,不一定有;所以只能反向获取值
                // 并将作者和审校或译者拼接,都算作author值
                String price = pub[pub.length - 1];
                String date = pub[pub.length - 2];
                String press = pub[pub.length - 3];
                StringBuilder author = new StringBuilder();
                int loop = 3;
                for (int i = 0; i < pub.length - loop; i++) {
                    author.append(pub[i]);
                }
                String score = element.select("div.star span.rating_nums").text();
                String num = element.select("div.star span.pl").text();
                // 截取评价人数
                String regEx = "[^0-9]";
                Pattern p = Pattern.compile(regEx);
                Matcher m = p.matcher(num);
                num = m.replaceAll("").trim();
                Book book = new Book("", name, score, num, author.toString(), press, date, price);
                books.add(book);
            }
        }
    }

    public static void main(String[] args) {
        // ?start=20&type=T
        // 这里是单线程执行的,结果正常返回,已打印输出
        GrabDouban gd = new GrabDouban();
        gd.parse("https://book.douban.com/tag/%E7%BC%96%E7%A8%8B");
        System.out.println("gd.getBooks() = " + gd.getBooks());
        // 获取总页数
//        String html = Spider.pickData("https://book.douban.com/tag/%E7%BC%96%E7%A8%8B");
//        Document doc = Jsoup.parse(html);
//        int totalPage = Integer.parseInt(doc.select("div.paginator > a").last().text());
//        StringBuilder sb;
//        for (int i = 0; i < totalPage; i++) {
//            sb = new StringBuilder("https://book.douban.com/tag/%E7%BC%96%E7%A8%8B?start=");
//            sb.append(i * 20);
//            sb.append("&type=T");
//        }
    }
}
```
```text
    多线程抓取多个页面数据,并保存值excel中
```
```java
package individual.cy.douban.utils;

import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hssf.usermodel.HSSFSheet;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.util.CellRangeAddress;

import java.io.FileOutputStream;
import java.io.OutputStream;
import java.lang.reflect.Method;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
import java.util.Map;
import java.util.UUID;

/**
 * Created with IntelliJ IDEA.
 *
 * @author: mystic
 * @date: 2017/12/21 17:21
 * @since: JDK1.8.0_144
 * @version: X
 * Description:
 */
public class ExportExcel {
    /***
     * 构造方法
     */
    private ExportExcel() {

    }

    /***
     * 工作簿
     */
    private static HSSFWorkbook workbook;

    /***
     * sheet
     */
    private static HSSFSheet sheet;
    /***
     * 标题行开始位置
     */
    private static final int TITLE_START_POSITION = 0;

    /***
     * 时间行开始位置
     */
    private static final int DATEHEAD_START_POSITION = 1;

    /***
     * 表头行开始位置
     */
    private static final int HEAD_START_POSITION = 2;

    /***
     * 文本行开始位置
     */
    private static final int CONTENT_START_POSITION = 3;


    /**
     * @param dataList  对象集合
     * @param titleMap  表头信息（对象属性名称->要显示的标题值)[按顺序添加]
     * @param sheetName sheet名称和表头值
     */
    public static void excelExport(List<?> dataList, Map<String, String> titleMap, String sheetName) {
        // 初始化workbook
        initHSSFWorkbook(sheetName);
        // 标题行
        createTitleRow(titleMap, sheetName);
        // 时间行
        createDateHeadRow(titleMap);
        // 表头行
        createHeadRow(titleMap);
        // 文本行
        createContentRow(dataList, titleMap);
        //设置自动伸缩
        //autoSizeColumn(titleMap.size());
        // 写入处理结果
        try {
            //生成UUID文件名称
            UUID uuid = UUID.randomUUID();
            String display = uuid + ".xls";
            //如果web项目，1、设置下载框的弹出（设置response相关参数)；2、通过httpservletresponse.getOutputStream()获取
            OutputStream out = new FileOutputStream(display);
            workbook.write(out);
            out.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /***
     *
     * @param sheetName
     *        sheetName
     */
    private static void initHSSFWorkbook(String sheetName) {
        workbook = new HSSFWorkbook();
        sheet = workbook.createSheet(sheetName);
    }

    /**
     * 生成标题（第零行创建）
     *
     * @param titleMap  对象属性名称->表头显示名称
     * @param sheetName sheet名称
     */
    private static void createTitleRow(Map<String, String> titleMap, String sheetName) {
        CellRangeAddress titleRange = new CellRangeAddress(0, 0, 0, titleMap.size() - 1);
        sheet.addMergedRegion(titleRange);
        HSSFRow titleRow = sheet.createRow(TITLE_START_POSITION);
        HSSFCell titleCell = titleRow.createCell(0);
        titleCell.setCellValue(sheetName);
    }

    /**
     * 创建时间行（第一行创建）
     *
     * @param titleMap 对象属性名称->表头显示名称
     */
    private static void createDateHeadRow(Map<String, String> titleMap) {
        CellRangeAddress dateRange = new CellRangeAddress(1, 1, 0, titleMap.size() - 1);
        sheet.addMergedRegion(dateRange);
        HSSFRow dateRow = sheet.createRow(DATEHEAD_START_POSITION);
        HSSFCell dateCell = dateRow.createCell(0);
        dateCell.setCellValue(new SimpleDateFormat("yyyy年MM月dd日").format(new Date()));
    }

    /**
     * 创建表头行（第二行创建）
     *
     * @param titleMap 对象属性名称->表头显示名称
     */
    private static void createHeadRow(Map<String, String> titleMap) {
        // 第1行创建
        HSSFRow headRow = sheet.createRow(HEAD_START_POSITION);
        int i = 0;
        for (String entry : titleMap.keySet()) {
            HSSFCell headCell = headRow.createCell(i);
            headCell.setCellValue(titleMap.get(entry));
            i++;
        }
    }

    /**
     * @param dataList 对象数据集合
     * @param titleMap 表头信息
     */
    private static void createContentRow(List<?> dataList, Map<String, String> titleMap) {
        try {
            int i = 0;
            for (Object obj : dataList) {
                HSSFRow textRow = sheet.createRow(CONTENT_START_POSITION + i);
                int j = 0;
                for (String entry : titleMap.keySet()) {
                    String method = "get" + entry.substring(0, 1).toUpperCase() + entry.substring(1);
                    Method m = obj.getClass().getMethod(method, null);
                    String value = m.invoke(obj, null).toString();
                    HSSFCell textcell = textRow.createCell(j);
                    textcell.setCellValue(value);
                    j++;
                }
                i++;
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 自动伸缩列（如非必要，请勿打开此方法，耗内存）
     *
     * @param size 列数
     */
    private static void autoSizeColumn(Integer size) {
        for (int j = 0; j < size; j++) {
            sheet.autoSizeColumn(j);
        }
    }
}
```
```java
package individual.cy.douban.main;

import com.google.common.util.concurrent.ThreadFactoryBuilder;
import individual.cy.douban.pojo.Book;
import individual.cy.douban.pojo.BookComparator;
import individual.cy.douban.utils.ExportExcel;
import individual.cy.douban.utils.Spider;
import individual.cy.douban.web.GrabDouban;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;

import java.util.*;
import java.util.concurrent.*;

/**
 * Created with IntelliJ IDEA.
 *
 * @author: mystic
 * @date: 2017/12/21 11:23
 * @since: JDK1.8.0_144
 * @version: X
 * Description:
 */
public class Main {
    /**
     * 创建线程池
     */
    private static ThreadFactory threadFactory = new ThreadFactoryBuilder().setNameFormat("pool-%d").build();
    private static ExecutorService executorService = new ThreadPoolExecutor(5, 200, 0L,
            TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>(1024), threadFactory, new ThreadPoolExecutor.AbortPolicy());

    public static void main(String[] args) throws InterruptedException {
        List<Book> books = new Vector<>();
        // 实现每一页一个线程获取数据
        // 获取总页数
        String html = Spider.pickData("https://book.douban.com/tag/%E7%BC%96%E7%A8%8B");
        /*String html = Spider.pick4data("https://book.douban.com/tag/%E7%BC%96%E7%A8%8B","220.249.185.178","9999");*/
        Document doc = Jsoup.parse(html);
        int totalPage = Integer.parseInt(doc.select("div.paginator > a").last().text());
        StringBuilder sb;
        for (int i = 0; i < totalPage; i++) {
            GrabDouban douban = new GrabDouban(books);
            sb = new StringBuilder("https://book.douban.com/tag/%E7%BC%96%E7%A8%8B?start=");
            sb.append(i * 20);
            sb.append("&type=T");
            douban.setUrl(sb.toString());
            executorService.execute(douban);
            // 防止爬取速度太快,IP被封
            Thread.sleep(3000L);
        }
        executorService.shutdown();
        // 排序
        books.sort(new BookComparator());
        // 添加编号
        List<Book> noBooks = new ArrayList<>();
        Integer no = 0;
        for (Book book : books) {
            book.setId((++no).toString());
            noBooks.add(book);
        }
        // 截取前40个
        noBooks = noBooks.subList(0,40);
        Map<String, String> title = new LinkedHashMap<>(16);
        title.put("id", "序号");
        title.put("name", "书名");
        title.put("score", "评分");
        title.put("num", "评价人数");
        title.put("author", "作者");
        title.put("press", "出版社");
        title.put("date", "出版日期");
        title.put("price", "价格");
        String sheet = "豆瓣编程书籍排行";
        ExportExcel.excelExport(noBooks, title, sheet);
    }
}
```
[Github Source Code](https://github.com/pplmx/Spider)