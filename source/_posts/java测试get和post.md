title: java测试get和post
date: 2015-08-13 16:10:06
tags: java
categories: java
---
有时候需要测试表单的提交或者获取，一个是通过在命令行输入
http get: curl http://localhost:8888/api/testget 
http post: curl -d "name=lpp&passwd=123456" http://localhost:8888/api/testpost"
post当参数很多的时候就不是很好办了，也不能够单元测试，所以还是写成代码比较漂亮。
<!--more-->
http post java代码：

{% codeblock lang:java %}
public static void main(String args[]) {
        try {
            URL url = new URL("http://localhost:8888/api/testpost");//接收方法
            URLConnection connection = url.openConnection();
            connection.setDoOutput(true);
            OutputStreamWriter out = new OutputStreamWriter(connection.getOutputStream(), "utf-8");
            out.write("name=lpp&passwd=123456"); //post表单数据
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
        }catch(Exception e){
            e.printStackTrace();
        }
    }
{% endcodeblock %}

http get java代码:
{% codeblock lang:java %}
public static void main(String args[]) {
        String result = "";
        BufferedReader in = null;
        try {
            String url = "http://localhost:8888/api/testget?id=1";
            URL realUrl = new URL(url);
            URLConnection connection = realUrl.openConnection();
            connection.setRequestProperty("accept", "*/*");
            connection.setRequestProperty("connection", "Keep-Alive");
            connection.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            connection.connect();
            Map<String, List<String>> map = connection.getHeaderFields();
            for (String key : map.keySet()) {
                System.out.println(key + "--->" + map.get(key));
            }
            in = new BufferedReader(new InputStreamReader(
                    connection.getInputStream()));
            String line;
            while ((line = in.readLine()) != null) {
                result += line;
            }
            System.out.println("rlt:" + result);
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

{% endcodeblock %}
