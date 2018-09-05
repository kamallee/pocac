title: junit4测试运用
date: 2015-09-07 14:45:30
tags: junit
categories: junit
---
毋庸置疑，程序员要对自己编写的代码负责，您不仅要保证它能通过编译，正常地运行，而且要满足需求和设计预期的效果。单元测试正是验证代码行为是否满足预期的有效手段之一。但不可否认，做测试是件很枯燥无趣的事情，而一遍又一遍的测试则更是让人生畏的工作。幸运的是，单元测试工具 JUnit 使这一切变得简单艺术起来。
JUnit 是 Java 社区中知名度最高的单元测试工具。它诞生于 1997 年，由 Erich Gamma 和 Kent Beck 共同开发完成。其中 Erich Gamma 是经典著作《设计模式：可复用面向对象软件的基础》一书的作者之一，并在 Eclipse 中有很大的贡献；Kent Beck 则是一位极限编程（XP）方面的专家和先驱。
麻雀虽小，五脏俱全。JUnit 设计的非常小巧，但是功能却非常强大。Martin Fowler 如此评价 JUnit：在软件开发领域，从来就没有如此少的代码起到了如此重要的作用。它大大简化了开发人员执行单元测试的难度，特别是 JUnit 4 使用 Java 5 中的注解（annotation）使测试变得更加简单。
<!--more-->
{% link 普通测试方法参考 http://albertchen.top/2015/05/27/JUnit4-%E4%BD%BF%E7%94%A8/ 测试方法参考 %}

JUnit4 支持的Annotation有:
@Test：测试方法 返回值必须为void
    A) (expected=XXEception.class)
    B) (timeout=xxx)
@Ignore: 被忽略的测试方法
@Before: 每一个测试方法执行之前运行。
@After : 每一个测试方法执行之后运行。
@BefreClass 所有测试开始之前运行。
@AfterClass 所有测试结果之后运行。

现在项目很多都是界面和接口完全分离，比如android app分为android前端和android后端，后端一般都会使用Api方式来调用需要的方法。这样能使前端和后端完全解耦，使开发人员能够更加专注于自己擅长的领域并且不需要依赖于其他的任何代码。

这里有三个接口需要测试，分别为增删查操作。
1.[post]http://localhost:8080/api/add.json 表单数据为id，该数据不能小于1，并且只能是整数
2.[post]http://localhost:8080/api/delete.json 表单数据为name，name不能重复，不能空
3.[get]http://localhost:8080/api/get.json 一定有返回值
现在我们需要对这个三个接口进行单元测试，尽量保证api的可用性，保证api满足要求。

首先需要引入junit4 jar包
{% codeblock lang:xml %}
<dependency>
     <groupId>junit</groupId>
     <artifactId>junit</artifactId>
     <version>4.10</version>
     <scope>test</scope>
</dependency>
{% endcodeblock %}

参数化测试代码如下：
{% codeblock lang:java %}
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.net.URLConnection;
import java.util.Arrays;
import java.util.Collection;
import java.util.List;
import java.util.Map;
import static org.junit.Assert.assertEquals;

import org.junit.*;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameters;

/**
 * Created by kamal on 15/9/6.
 */
@RunWith(Parameterized.class)
public class TestAction {
    private String param;
    private String result;

    //@Parameters
    public static Collection getParameters(){
        return Arrays.asList(new Object[][]{
                {"id=1","1"},
                {"id=0","0"},
                {"id=-1","0"},
                {"id=x","0"}//不能为非数字
        });
    }

    @Parameters
    public static Collection parameters(){
        return Arrays.asList(new Object[][]{
                {"name=lpp","1"},
                {"name=lpp","0"},//名字不能重复
                {"name=","0"},
                {"","0"},
                {"names=","0"}
        });
    }

    public TestAdImage(String expected,String result){
        this.param = expected;
        this.result = result;
    }

    @BeforeClass
    public static void beforeClass(){
        System.out.println("beforeClass...");
    }

    @Before
    public void before(){
        System.out.println("before");
    }

    public void _testAdd(String params){
        try {
            String url = "http://localhost:8080/api/add.json";
            URLConnection connection = HttpUtils.getConnection(url,HttpUtils.HTTP_POST);
            OutputStreamWriter out = new OutputStreamWriter(connection.getOutputStream(), "utf-8");
            out.write(params); //post表单数据
            //out.write("name=");
            out.flush();
            out.close();
            String sCurrentLine;
            String sTotalString;
            sTotalString = "";
            InputStream l_urlStream;
            l_urlStream = connection.getInputStream();
            BufferedReader l_reader = new BufferedReader(new InputStreamReader(
                    l_urlStream));
            while ((sCurrentLine = l_reader.readLine()) != null) {
                sTotalString += sCurrentLine + "\r\n";

            }
            System.out.println(sTotalString);
            String ifSuccess = JSONUtils.getSuccessValue(sTotalString);
            //assertEquals(ifSuccess,"1");
        }catch(Exception e){
            e.printStackTrace();
        }
    }

    @Test public void testAdd(){
        _testAdd(param);
    }

    //删除
    public void _testDelete(String params){
        try {
            String url = "http://localhost:8080/api/delete.json";
            URLConnection connection = HttpUtils.getConnection(url,HttpUtils.HTTP_POST);
            OutputStreamWriter out = new OutputStreamWriter(connection.getOutputStream(), "utf-8");
            out.write(params); //post表单数据
            //out.write("id=");
            out.flush();
            out.close();
            String sCurrentLine;
            String sTotalString;
            sTotalString = "";
            InputStream l_urlStream;
            l_urlStream = connection.getInputStream();
            BufferedReader l_reader = new BufferedReader(new InputStreamReader(
                    l_urlStream));
            while ((sCurrentLine = l_reader.readLine()) != null) {
                sTotalString += sCurrentLine + "\r\n";

            }
            System.out.println(sTotalString);
            String ifSuccess = JSONUtils.getSuccessValue(sTotalString);
            //assertEquals(ifSuccess,"1");
        }catch(Exception e){
            e.printStackTrace();
        }
    }

    @Ignore
    @Test public void testDelete(){
        _testDelete(param);
    }

    @Ignore
    @Test public void testGet(){
        String result = "";
        BufferedReader in = null;
        try {
            String url = "http://127.0.0.1:8080/api/get.json";
            URLConnection connection = HttpUtils.getConnection(url,HttpUtils.HTTP_GET);
            Map<String, List<String>> map = connection.getHeaderFields();
            in = new BufferedReader(new InputStreamReader(
                    connection.getInputStream()));
            String line;
            while ((line = in.readLine()) != null) {
                result += line;
            }
            System.out.println("rlt:" + result);
            String ifSuccess = JSONUtils.getSuccessValue(result);
            assertEquals(ifSuccess,"1");
        } catch (Exception e) {
            e.printStackTrace();
        }
        finally {
            try {
                if (in != null) {
                    in.close();
                }
            } catch (Exception e2) {
                e2.printStackTrace();
            }
        }
    }

    @After
    public void after(){
        System.out.println("after");
    }

    @AfterClass
    public static void afterClass(){
        System.out.println("afterClass");
    }
}
{% endcodeblock %}

一个一个测试显得很繁琐，可以将测试进行打包测试
在实际项目中，随着项目进度的开展，单元测试类会越来越多，可是直到现在我们还只会一个一个的单独运行测试类，这在实际项目实践中肯定是不可行的。为了解决这个问题，JUnit 提供了一种批量运行测试类的方法，叫做测试套件。这样，每次需要验证系统功能正确性时，只执行一个或几个测试套件便可以了。测试套件的写法非常简单，您只需要遵循以下规则：
创建一个空类作为测试套件的入口。
使用注解 org.junit.runner.RunWith 和 org.junit.runners.Suite.SuiteClasses 修饰这个空类。
将 org.junit.runners.Suite 作为参数传入注解 RunWith，以提示 JUnit 为此类使用套件运行器执行。
将需要放入此测试套件的测试类组成数组作为注解 SuiteClasses 的参数。
保证这个空类使用 public 修饰，而且存在公开的不带有任何参数的构造函数。
{% codeblock lang:java %}
import org.junit.runner.RunWith;
import org.junit.runners.Suite;

/**
 * Created by kamal on 15/9/7.
 */
@RunWith(Suite.class)
@Suite.SuiteClasses({
        TestAction.class
})
public class TestWanted {

}
{% endcodeblock %}
HttpUtils类如下所示
{% codeblock lang:java %}
import java.net.URL;
import java.net.URLConnection;

/**
 * Created by kamal on 15/8/14.
 */
public class HttpUtils {
    public static final String HTTP_POST = "post";
    public static final String HTTP_GET = "get";

    public static URLConnection getConnection(String url,String httpType){
        try {
            URL realUrl = new URL(url);
            URLConnection connection = realUrl.openConnection();
            connection.setRequestProperty("accept", "*/*");
            connection.setRequestProperty("connection", "Keep-Alive");
            connection.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            if( httpType.equals(HTTP_POST)){
                connection.setDoOutput(true);
            }
            connection.connect();
            return connection;
        }catch(Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
{% endcodeblock %}

JSONUtils如下所示
{% codeblock lang:java %}
import java.net.URL;
import java.net.URLConnection;

/**
 * Created by kamal on 15/8/14.
 */
public class HttpUtils {
    public static final String HTTP_POST = "post";
    public static final String HTTP_GET = "get";

    public static URLConnection getConnection(String url,String httpType){
        try {
            URL realUrl = new URL(url);
            URLConnection connection = realUrl.openConnection();
            connection.setRequestProperty("accept", "*/*");
            connection.setRequestProperty("connection", "Keep-Alive");
            connection.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            if( httpType.equals(HTTP_POST)){
                connection.setDoOutput(true);
            }
            connection.connect();
            return connection;
        }catch(Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
{% endcodeblock %}
参考：{% link 单元测试利器junit http://www.ibm.com/developerworks/cn/java/j-lo-junit4/ 单元测试利器 %}

