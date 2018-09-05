title: thrift框架简介
date: 2015-08-07 11:44:04
tags: thrift 
categories: thrift
---
本文使用环境macbook，intellij和thrift。
Apache Thrift 是facebook实现的一种高效的、支持多种编程语言的远程服务调用框架。目前流行的服务调用方式有很多种，如基于SOAP 消息格式的Web Service，基于JSON消息格式的RESTfull服务等。用到的数据传输方式包括XML，JSON等，然而XML相对体积太大，传输效率低，JSON体积较小。Thrift采用接口描述语言定义并创建服务，支持可扩展的跨语言服务开发，所包含的代码生成引擎可以在多种语言中，如C++，JAVA，Python等创建高效的，无缝的服务，传输数据采用二进制格式，相对XML和JSON体积更小，对于高并发、大数据量和多语言的环境更有优势。
<!--more-->
为什么需要RPC：由于各服务部署在不同机器，服务间的调用免不了网络通信过程，服务消费方每调用一个服务都要写一坨网络通信相关的代码，不仅复杂而且极易出错。我们需要有一种方式能让我们像调用本地服务一样调用远程服务，而让调用者对网络通信这些细节透明，那么将大大提高生产力。

Thrift特点：Thrift是IDL(interface definition language)描述性语言的一个具体实现，Thrift适用于程序对程序静态的数据交换，需要先确定好他的数据结构，他是完全静态化的，当数据结构发生变化时，必须重新编辑IDL文件，代码生成，再编译载入的流程，跟其他IDL工具相比较可以视为是Thrift的弱项，Thrift适用于搭建大型数据交换及存储的通用工具，对于大型系统中的子系统间数据传输相对于JSON和xml无论在性能、传输大小上有明显的优势，对于高并发、大数据量和多语言的环境更有优势。

<h2>RPC调用流程</h2>
{% img /img/thriftflow.png RPC调用流程 %}

{% blockquote %}
1）服务消费方（client）调用以本地调用方式调用服务；
2）client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；
3）client stub找到服务地址，并将消息发送到服务端；
4）server stub收到消息后进行解码；
5）server stub根据解码结果调用本地的服务；
6）本地服务执行并将结果返回给server stub；
7）server stub将返回结果打包成消息并发送至消费方；
8）client stub接收到消息，并进行解码；
9）服务消费方得到最终结果。
{% endblockquote %}

IDL支持的数据类型
{% blockquote %}
bool 布尔型  
byte ８位整数  
i16  16位整数   对应Java 的short
i32  32位整数   对应Java 的short
i64  64位整数   对应Java 的long
double 双精度浮点数  
string 字符串  
binary 字节数组  
list<i16> List集合，必须指明泛型   对应 Java 的 ArrayList
map<string, string> Map类型，必须指明泛型   对应 Java 的 HashSet
set<i32> Set集合，必须指明泛型    对应 Java 的 HashMap
结构体类型：struct：定义公共的对象，类似于 C 语言中的结构体定义，在 Java 中是一个 JavaBean
异常类型：exception：对应 Java 的 Exception
服务类型：service：对应服务的类
{% endblockquote %}

支持传输格式
{% blockquote %}
（1）支持的传输格式
    TBinaryProtocol – 二进制格式.
    TCompactProtocol – 压缩格式
    TJSONProtocol – JSON格式
    TSimpleJSONProtocol –提供JSON只写协议, 生成的文件很容易通过脚本语言解析。
    TDebugProtocol – 使用易懂的可读的文本格式，以便于debug
（2） 支持的数据传输方式
    TSocket -阻塞式socker
    TFramedTransport – 以frame为单位进行传输，非阻塞式服务中使用。
    TFileTransport – 以文件形式进行传输。
    TMemoryTransport – 将内存用于I/O. java实现时内部实际使用了简单的ByteArrayOutputStream。
    TZlibTransport – 使用zlib进行压缩， 与其他传输方式联合使用。当前无java实现。
（3）支持的服务模型
    TSimpleServer – 简单的单线程服务模型，常用于测试
    TThreadPoolServer – 多线程服务模型，使用标准的阻塞式IO。
    TNonblockingServer – 多线程服务模型，使用非阻塞式IO（需使用TFramedTransport数据传输方式）
 {% endblockquote %}
<h2>thrift架构</h2>
{% img /img/thriftarch.png thrift架构 %}
图中黄色部分是用户实现的业务逻辑，褐色部分是根据 Thrift 定义的服务接口描述文件生成的客户端和服务器端代码框架，红色部分是根据 Thrift 文件生成代码实现数据的读写操作。红色部分以下是 Thrift 的传输体系、协议以及底层 I/O 通信，使用 Thrift 可以很方便的定义一个服务并且选择不同的传输协议和传输层而不用重新生成代码。
<h2>添加thrift依赖</h2>
{% codeblock swift依赖 lang:xml %}       
 <dependency>
   <groupId>org.apache.thrift</groupId>
      <artifactId>libthrift</artifactId>
      <version>0.9.2</version>
   </dependency>
   <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
   <version>1.7.7</version>
   <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>
</dependency>
{% endcodeblock %}
添加进来0.9.2如果显示红色即没有找到，可以先update maven indices(更新maven索引，maven在刚使用的时候会先去远程服务器下载索引文件nexus-maven-repository-index.gz,后面不会再下载，所有时候索引不是最新的，需要更新）
<h2>创建helloworld.thrift文件</h2>
{% codeblock thrift文件 lang:java %}
namespace java com.lpp.thrift.hello

service  IHelloWorldService {
  string sayHello(1:string username)
}
{% endcodeblock %}
<h2>intellij terminal 中切换到helloworld.thrift文件夹，执行命令</h2>
thrift -r -gen java helloworld.thrift
<h2>将生成的文件移动到你要放得位置</h2>
生成的文件可能会有问题：@Override is not allowed when implementing interface method
原因：项目语言级别过低
排除方法：File->Project Structure->Project->Project language level设置到6以上
这样后面编译可能还会有问题：Error:java: Compilation failed: internal java compiler error
原因：jdk的版本不一致
排除方法：进入preference->Compiler->Java Compiler->Project bytecode version设为你的jdk版本，per-module bytecode version同样设置
<h2>实现生成的文件中的Iface</h2>
{% codeblock 实现Iface lang:java %}
public class HelloWorldServiceImple implements IHelloWorldService.Iface {
    @Override
    public String sayHello(String username) throws TException {
        return "hello," + username + ",welcome to thrift world!";
    }
}
{% endcodeblock %}
<h2>实现thrift服务器端</h2>
{% codeblock swift服务器端 lang:java %}
public class SimpleServer {
    public static int SERVER_PORT = 8712;
    public void startServer(){
        try{
            System.out.println("TSimple Server start...");
            TProcessor p = new IHelloWorldService.Processor<IHelloWorldService.Iface>(new HelloWorldServiceImple());
            TServerSocket soc = new TServerSocket(SERVER_PORT);
            TServer.Args args = new TServer.Args(soc);
            args.processor(p);
            args.protocolFactory(new TBinaryProtocol.Factory());
            TServer server = new TSimpleServer(args);
            server.serve();
        }catch(Exception e){
            e.printStackTrace();
        }
    }
    public static void main(String args[]){
        SimpleServer server = new SimpleServer();
        server.startServer();
    }
}
{% endcodeblock %}
<h2>实现thrift客户端</h2>
{% codeblock swift客户端 lang:java %}
public class SimpleClient {
    public static final String SERVER_IP = "127.0.0.1";
    public static final int SERVER_PORT = 8712;
    public static final int TIMEOUT = 30000;
    public void startClient(String name){
        TTransport transport = null;
        try{
            transport = new TSocket(SERVER_IP,SERVER_PORT,TIMEOUT);
            TProtocol pro = new TBinaryProtocol(transport);
            IHelloWorldService.Client client = new IHelloWorldService.Client(pro);
            transport.open();
            String rlt = client.sayHello(name);
            System.out.println("client:" + rlt);
        }catch(Exception e){
            e.printStackTrace();
        }
    }
    public static void main(String args[]){
        SimpleClient client = new SimpleClient();
        client.startClient("lpp");
    }
}
{% endcodeblock %}
<h2>先执行服务器端，然后执行客户端</h2>

<h2>部署服务</h2>
thrift框架生成的服务包括服务端和客户端，部署模型如下所示
{% img /img/thriftpack.png thrift部署 %}
从图可以看出，服务端和客户端部署时，需要用到公共的jar包和java文件，其中IHelloWorldService.java由IHelloWorldService.thrift编译而来。在服务端，服务必须实现IHelloWorldService.Iface接口和服务器启动代码SimpleServer.java。在客户端调用服务的代码SimpleClient.java。服务端和客户端通过IHelloWorldService.java提供的API实现远程服务调用。

<h2>客户端优化</h2>
上面的例子只是简单的入门级demo，与生产环境还有比较大的不同。客户端每次使用的时候都要打开Socket连接，并且耦合服务端IP、端口等。那么如何对客户端进行封装，使使用者可以不用写大量的客户端代码，二只需要跟调用本地接口一样？

这里我们使用动态代理。
动态代理都要实现InvocationHandler，并且重写invoke方法，该方法为回调方法，可以在里面写客户端链接逻辑，并且调用方法。
{% codeblock lang:java %}
import com.google.common.collect.Lists;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.protocol.TProtocol;
import org.apache.thrift.transport.TSocket;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.List;

/**
 * @author lipengpeng
 * @desc
 * @date 2016-07-21 下午4:42
 */
public class DynamicClientProxy<T> implements InvocationHandler {
    private  Class<T> tc = null;
    public Object createProxy(Class<T> tc){
        this.tc = tc;
        return Proxy.newProxyInstance(tc.getClassLoader(),tc.getInterfaces(),this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        List<ServerNode> list = Lists.newArrayList();
        list.add(new ServerNode("127.0.0.1",8712));
        int timeout = 4000;
        //failover失败重试
        for( ServerNode serverNode : list ){
            String ip = serverNode.getIp();
            int port = serverNode.getPort();

            //后面优化可以在这里设置连接池，提高建立Socket链接的速度
            TSocket tSocket = null;
            try{
                tSocket = new TSocket(ip,port);
                tSocket.setTimeout(timeout);
                TProtocol tProtocol = new TBinaryProtocol(tSocket);
                Class[] argsClass = new Class[]{
                        TProtocol.class
                };
                Constructor<T> constructor = tc.getConstructor(argsClass);
                T client = constructor.newInstance(tProtocol);
                tSocket.open();
                return method.invoke(client,args);
            }catch(Exception e){

            }
        }
        return null;
    }

    class ServerNode{
        private String ip;
        private int port;

        public ServerNode(String ip,int port){
            this.ip = ip;
            this.port = port;
        }

        public String getIp() {
            return ip;
        }

        public void setIp(String ip) {
            this.ip = ip;
        }

        public int getPort() {
            return port;
        }

        public void setPort(int port) {
            this.port = port;
        }
    }
}

//使用方法
DynamicClientProxy<IHelloWorldService.Client> clientDynamicClientProxy = new DynamicClientProxy<>();
IHelloWorldService.Iface service = (IHelloWorldService.Iface)clientDynamicClientProxy.createProxy(IHelloWorldService.Client.class);
System.out.print(service.sayHello("kamal"));
{% endcodeblock %}

从使用方法可以看出，这种封装还不够彻底，这里也可以使用spring进行配置，将生产代理交给spring来做，这样就可以达到调用接口就像调用本地接口一样
{% codeblock lang:java %}
//首先生产工厂方法
import java.util.List;
/**
 * @author lipengpeng
 * @desc
 * @date 2016-07-21 下午5:20
 */
public class DynamicClientProxyFactory {
    public static Object createIface(String clazzIfaceName,List<String> servers){
        try{
            int idx = clazzIfaceName.lastIndexOf('$');
            String clazzClientName = clazzIfaceName.substring(0,idx) + "$Client";
            Class clientClazz = Class.forName(clazzClientName);
            DynamicClientProxy proxy = new DynamicClientProxy();
            return proxy.createProxy(clientClazz);
        }catch (Exception e){

        }
        return null;
    }
}
{% endcodeblock %}
{% codeblock lang:xml %}
//spring bean配置
<bean id="helloService" class="com.crazyfish.kamal.test.TestThrift.DynamicClientProxyFactory" factory-method="createIface">
    <constructor-arg>
        <value>com.crazyfish.kamal.controller.IHelloWorldService$Iface</value>
    </constructor-arg>
    <constructor-arg>
        <value>127.0.0.1:8172</value>
    </constructor-arg>
</bean>
{% endcodeblock %}

{% codeblock lang:java %}
//使用方法（测试）
ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("/conf/applicationContext.xml");
IHelloWorldService.Iface service = (IHelloWorldService.Iface)applicationContext.getBean("helloService");
System.out.print(service.sayHello("kkk"));

//使用方法
@Resource
private IHelloWorldService.Iface helloService;
try {
        System.out.print(helloService.sayHello("xxx"));//调用代理方法（invoke）
    }catch (Exception e){
        e.printStackTrace();
    }
{% endcodeblock %}

<h2>附thrift引擎生成的代码</h2>
{% codeblock lang:java %}
import org.apache.thrift.scheme.IScheme;
import org.apache.thrift.scheme.SchemeFactory;
import org.apache.thrift.scheme.StandardScheme;

import org.apache.thrift.scheme.TupleScheme;
import org.apache.thrift.protocol.TTupleProtocol;
import org.apache.thrift.protocol.TProtocolException;
import org.apache.thrift.EncodingUtils;
import org.apache.thrift.TException;
import org.apache.thrift.async.AsyncMethodCallback;
import org.apache.thrift.server.AbstractNonblockingServer.*;
import java.util.List;
import java.util.ArrayList;
import java.util.Map;
import java.util.HashMap;
import java.util.EnumMap;
import java.util.Set;
import java.util.HashSet;
import java.util.EnumSet;
import java.util.Collections;
import java.util.BitSet;
import java.nio.ByteBuffer;
import java.util.Arrays;
import javax.annotation.Generated;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SuppressWarnings({"cast", "rawtypes", "serial", "unchecked"})
@Generated(value = "Autogenerated by Thrift Compiler (0.9.2)", date = "2015-8-7")
public class IHelloWorldService {

  public interface Iface {

    public String sayHello(String username) throws org.apache.thrift.TException;

  }

  public interface AsyncIface {

    public void sayHello(String username, org.apache.thrift.async.AsyncMethodCallback resultHandler) throws org.apache.thrift.TException;

  }

  public static class Client extends org.apache.thrift.TServiceClient implements Iface {
    public static class Factory implements org.apache.thrift.TServiceClientFactory<Client> {
      public Factory() {}
      public Client getClient(org.apache.thrift.protocol.TProtocol prot) {
        return new Client(prot);
      }
      public Client getClient(org.apache.thrift.protocol.TProtocol iprot, org.apache.thrift.protocol.TProtocol oprot) {
        return new Client(iprot, oprot);
      }
    }

    public Client(org.apache.thrift.protocol.TProtocol prot)
    {
      super(prot, prot);
    }

    public Client(org.apache.thrift.protocol.TProtocol iprot, org.apache.thrift.protocol.TProtocol oprot) {
      super(iprot, oprot);
    }

    public String sayHello(String username) throws org.apache.thrift.TException
    {
      send_sayHello(username);
      return recv_sayHello();
    }

    public void send_sayHello(String username) throws org.apache.thrift.TException
    {
      sayHello_args args = new sayHello_args();
      args.setUsername(username);
      sendBase("sayHello", args);
    }

    public String recv_sayHello() throws org.apache.thrift.TException
    {
      sayHello_result result = new sayHello_result();
      receiveBase(result, "sayHello");
      if (result.isSetSuccess()) {
        return result.success;
      }
      throw new org.apache.thrift.TApplicationException(org.apache.thrift.TApplicationException.MISSING_RESULT, "sayHello failed: unknown result");
    }

  }
  public static class AsyncClient extends org.apache.thrift.async.TAsyncClient implements AsyncIface {
    public static class Factory implements org.apache.thrift.async.TAsyncClientFactory<AsyncClient> {
      private org.apache.thrift.async.TAsyncClientManager clientManager;
      private org.apache.thrift.protocol.TProtocolFactory protocolFactory;
      public Factory(org.apache.thrift.async.TAsyncClientManager clientManager, org.apache.thrift.protocol.TProtocolFactory protocolFactory) {
        this.clientManager = clientManager;
        this.protocolFactory = protocolFactory;
      }
      public AsyncClient getAsyncClient(org.apache.thrift.transport.TNonblockingTransport transport) {
        return new AsyncClient(protocolFactory, clientManager, transport);
      }
    }

    public AsyncClient(org.apache.thrift.protocol.TProtocolFactory protocolFactory, org.apache.thrift.async.TAsyncClientManager clientManager, org.apache.thrift.transport.TNonblockingTransport transport) {
      super(protocolFactory, clientManager, transport);
    }

    public void sayHello(String username, org.apache.thrift.async.AsyncMethodCallback resultHandler) throws org.apache.thrift.TException {
      checkReady();
      sayHello_call method_call = new sayHello_call(username, resultHandler, this, ___protocolFactory, ___transport);
      this.___currentMethod = method_call;
      ___manager.call(method_call);
    }

    public static class sayHello_call extends org.apache.thrift.async.TAsyncMethodCall {
      private String username;
      public sayHello_call(String username, org.apache.thrift.async.AsyncMethodCallback resultHandler, org.apache.thrift.async.TAsyncClient client, org.apache.thrift.protocol.TProtocolFactory protocolFactory, org.apache.thrift.transport.TNonblockingTransport transport) throws org.apache.thrift.TException {
        super(client, protocolFactory, transport, resultHandler, false);
        this.username = username;
      }

      public void write_args(org.apache.thrift.protocol.TProtocol prot) throws org.apache.thrift.TException {
        prot.writeMessageBegin(new org.apache.thrift.protocol.TMessage("sayHello", org.apache.thrift.protocol.TMessageType.CALL, 0));
        sayHello_args args = new sayHello_args();
        args.setUsername(username);
        args.write(prot);
        prot.writeMessageEnd();
      }

      public String getResult() throws org.apache.thrift.TException {
        if (getState() != org.apache.thrift.async.TAsyncMethodCall.State.RESPONSE_READ) {
          throw new IllegalStateException("Method call not finished!");
        }
        org.apache.thrift.transport.TMemoryInputTransport memoryTransport = new org.apache.thrift.transport.TMemoryInputTransport(getFrameBuffer().array());
        org.apache.thrift.protocol.TProtocol prot = client.getProtocolFactory().getProtocol(memoryTransport);
        return (new Client(prot)).recv_sayHello();
      }
    }

  }

  public static class Processor<I extends Iface> extends org.apache.thrift.TBaseProcessor<I> implements org.apache.thrift.TProcessor {
    private static final Logger LOGGER = LoggerFactory.getLogger(Processor.class.getName());
    public Processor(I iface) {
      super(iface, getProcessMap(new HashMap<String, org.apache.thrift.ProcessFunction<I, ? extends org.apache.thrift.TBase>>()));
    }

    protected Processor(I iface, Map<String,  org.apache.thrift.ProcessFunction<I, ? extends  org.apache.thrift.TBase>> processMap) {
      super(iface, getProcessMap(processMap));
    }

    private static <I extends Iface> Map<String,  org.apache.thrift.ProcessFunction<I, ? extends  org.apache.thrift.TBase>> getProcessMap(Map<String,  org.apache.thrift.ProcessFunction<I, ? extends  org.apache.thrift.TBase>> processMap) {
      processMap.put("sayHello", new sayHello());
      return processMap;
    }

    public static class sayHello<I extends Iface> extends org.apache.thrift.ProcessFunction<I, sayHello_args> {
      public sayHello() {
        super("sayHello");
      }

      public sayHello_args getEmptyArgsInstance() {
        return new sayHello_args();
      }

      protected boolean isOneway() {
        return false;
      }

      public sayHello_result getResult(I iface, sayHello_args args) throws org.apache.thrift.TException {
        sayHello_result result = new sayHello_result();
        result.success = iface.sayHello(args.username);
        return result;
      }
    }

  }

  public static class AsyncProcessor<I extends AsyncIface> extends org.apache.thrift.TBaseAsyncProcessor<I> {
    private static final Logger LOGGER = LoggerFactory.getLogger(AsyncProcessor.class.getName());
    public AsyncProcessor(I iface) {
      super(iface, getProcessMap(new HashMap<String, org.apache.thrift.AsyncProcessFunction<I, ? extends org.apache.thrift.TBase, ?>>()));
    }

    protected AsyncProcessor(I iface, Map<String,  org.apache.thrift.AsyncProcessFunction<I, ? extends  org.apache.thrift.TBase, ?>> processMap) {
      super(iface, getProcessMap(processMap));
    }

    private static <I extends AsyncIface> Map<String,  org.apache.thrift.AsyncProcessFunction<I, ? extends  org.apache.thrift.TBase,?>> getProcessMap(Map<String,  org.apache.thrift.AsyncProcessFunction<I, ? extends  org.apache.thrift.TBase, ?>> processMap) {
      processMap.put("sayHello", new sayHello());
      return processMap;
    }

    public static class sayHello<I extends AsyncIface> extends org.apache.thrift.AsyncProcessFunction<I, sayHello_args, String> {
      public sayHello() {
        super("sayHello");
      }

      public sayHello_args getEmptyArgsInstance() {
        return new sayHello_args();
      }

      public AsyncMethodCallback<String> getResultHandler(final AsyncFrameBuffer fb, final int seqid) {
        final org.apache.thrift.AsyncProcessFunction fcall = this;
        return new AsyncMethodCallback<String>() { 
          public void onComplete(String o) {
            sayHello_result result = new sayHello_result();
            result.success = o;
            try {
              fcall.sendResponse(fb,result, org.apache.thrift.protocol.TMessageType.REPLY,seqid);
              return;
            } catch (Exception e) {
              LOGGER.error("Exception writing to internal frame buffer", e);
            }
            fb.close();
          }
          public void onError(Exception e) {
            byte msgType = org.apache.thrift.protocol.TMessageType.REPLY;
            org.apache.thrift.TBase msg;
            sayHello_result result = new sayHello_result();
            {
              msgType = org.apache.thrift.protocol.TMessageType.EXCEPTION;
              msg = (org.apache.thrift.TBase)new org.apache.thrift.TApplicationException(org.apache.thrift.TApplicationException.INTERNAL_ERROR, e.getMessage());
            }
            try {
              fcall.sendResponse(fb,msg,msgType,seqid);
              return;
            } catch (Exception ex) {
              LOGGER.error("Exception writing to internal frame buffer", ex);
            }
            fb.close();
          }
        };
      }

      protected boolean isOneway() {
        return false;
      }

      public void start(I iface, sayHello_args args, org.apache.thrift.async.AsyncMethodCallback<String> resultHandler) throws TException {
        iface.sayHello(args.username,resultHandler);
      }
    }

  }

  public static class sayHello_args implements org.apache.thrift.TBase<sayHello_args, sayHello_args._Fields>, java.io.Serializable, Cloneable, Comparable<sayHello_args>   {
    private static final org.apache.thrift.protocol.TStruct STRUCT_DESC = new org.apache.thrift.protocol.TStruct("sayHello_args");

    private static final org.apache.thrift.protocol.TField USERNAME_FIELD_DESC = new org.apache.thrift.protocol.TField("username", org.apache.thrift.protocol.TType.STRING, (short)1);

    private static final Map<Class<? extends IScheme>, SchemeFactory> schemes = new HashMap<Class<? extends IScheme>, SchemeFactory>();
    static {
      schemes.put(StandardScheme.class, new sayHello_argsStandardSchemeFactory());
      schemes.put(TupleScheme.class, new sayHello_argsTupleSchemeFactory());
    }

    public String username; // required

    /** The set of fields this struct contains, along with convenience methods for finding and manipulating them. */
    public enum _Fields implements org.apache.thrift.TFieldIdEnum {
      USERNAME((short)1, "username");

      private static final Map<String, _Fields> byName = new HashMap<String, _Fields>();

      static {
        for (_Fields field : EnumSet.allOf(_Fields.class)) {
          byName.put(field.getFieldName(), field);
        }
      }

      /**
       * Find the _Fields constant that matches fieldId, or null if its not found.
       */
      public static _Fields findByThriftId(int fieldId) {
        switch(fieldId) {
          case 1: // USERNAME
            return USERNAME;
          default:
            return null;
        }
      }

      /**
       * Find the _Fields constant that matches fieldId, throwing an exception
       * if it is not found.
       */
      public static _Fields findByThriftIdOrThrow(int fieldId) {
        _Fields fields = findByThriftId(fieldId);
        if (fields == null) throw new IllegalArgumentException("Field " + fieldId + " doesn't exist!");
        return fields;
      }

      /**
       * Find the _Fields constant that matches name, or null if its not found.
       */
      public static _Fields findByName(String name) {
        return byName.get(name);
      }

      private final short _thriftId;
      private final String _fieldName;

      _Fields(short thriftId, String fieldName) {
        _thriftId = thriftId;
        _fieldName = fieldName;
      }

      public short getThriftFieldId() {
        return _thriftId;
      }

      public String getFieldName() {
        return _fieldName;
      }
    }

    // isset id assignments
    public static final Map<_Fields, org.apache.thrift.meta_data.FieldMetaData> metaDataMap;
    static {
      Map<_Fields, org.apache.thrift.meta_data.FieldMetaData> tmpMap = new EnumMap<_Fields, org.apache.thrift.meta_data.FieldMetaData>(_Fields.class);
      tmpMap.put(_Fields.USERNAME, new org.apache.thrift.meta_data.FieldMetaData("username", org.apache.thrift.TFieldRequirementType.DEFAULT, 
          new org.apache.thrift.meta_data.FieldValueMetaData(org.apache.thrift.protocol.TType.STRING)));
      metaDataMap = Collections.unmodifiableMap(tmpMap);
      org.apache.thrift.meta_data.FieldMetaData.addStructMetaDataMap(sayHello_args.class, metaDataMap);
    }

    public sayHello_args() {
    }

    public sayHello_args(
      String username)
    {
      this();
      this.username = username;
    }

    /**
     * Performs a deep copy on <i>other</i>.
     */
    public sayHello_args(sayHello_args other) {
      if (other.isSetUsername()) {
        this.username = other.username;
      }
    }

    public sayHello_args deepCopy() {
      return new sayHello_args(this);
    }

    @Override
    public void clear() {
      this.username = null;
    }

    public String getUsername() {
      return this.username;
    }

    public sayHello_args setUsername(String username) {
      this.username = username;
      return this;
    }

    public void unsetUsername() {
      this.username = null;
    }

    /** Returns true if field username is set (has been assigned a value) and false otherwise */
    public boolean isSetUsername() {
      return this.username != null;
    }

    public void setUsernameIsSet(boolean value) {
      if (!value) {
        this.username = null;
      }
    }

    public void setFieldValue(_Fields field, Object value) {
      switch (field) {
      case USERNAME:
        if (value == null) {
          unsetUsername();
        } else {
          setUsername((String)value);
        }
        break;

      }
    }

    public Object getFieldValue(_Fields field) {
      switch (field) {
      case USERNAME:
        return getUsername();

      }
      throw new IllegalStateException();
    }

    /** Returns true if field corresponding to fieldID is set (has been assigned a value) and false otherwise */
    public boolean isSet(_Fields field) {
      if (field == null) {
        throw new IllegalArgumentException();
      }

      switch (field) {
      case USERNAME:
        return isSetUsername();
      }
      throw new IllegalStateException();
    }

    @Override
    public boolean equals(Object that) {
      if (that == null)
        return false;
      if (that instanceof sayHello_args)
        return this.equals((sayHello_args)that);
      return false;
    }

    public boolean equals(sayHello_args that) {
      if (that == null)
        return false;

      boolean this_present_username = true && this.isSetUsername();
      boolean that_present_username = true && that.isSetUsername();
      if (this_present_username || that_present_username) {
        if (!(this_present_username && that_present_username))
          return false;
        if (!this.username.equals(that.username))
          return false;
      }

      return true;
    }

    @Override
    public int hashCode() {
      List<Object> list = new ArrayList<Object>();

      boolean present_username = true && (isSetUsername());
      list.add(present_username);
      if (present_username)
        list.add(username);

      return list.hashCode();
    }

    @Override
    public int compareTo(sayHello_args other) {
      if (!getClass().equals(other.getClass())) {
        return getClass().getName().compareTo(other.getClass().getName());
      }

      int lastComparison = 0;

      lastComparison = Boolean.valueOf(isSetUsername()).compareTo(other.isSetUsername());
      if (lastComparison != 0) {
        return lastComparison;
      }
      if (isSetUsername()) {
        lastComparison = org.apache.thrift.TBaseHelper.compareTo(this.username, other.username);
        if (lastComparison != 0) {
          return lastComparison;
        }
      }
      return 0;
    }

    public _Fields fieldForId(int fieldId) {
      return _Fields.findByThriftId(fieldId);
    }

    public void read(org.apache.thrift.protocol.TProtocol iprot) throws org.apache.thrift.TException {
      schemes.get(iprot.getScheme()).getScheme().read(iprot, this);
    }

    public void write(org.apache.thrift.protocol.TProtocol oprot) throws org.apache.thrift.TException {
      schemes.get(oprot.getScheme()).getScheme().write(oprot, this);
    }

    @Override
    public String toString() {
      StringBuilder sb = new StringBuilder("sayHello_args(");
      boolean first = true;

      sb.append("username:");
      if (this.username == null) {
        sb.append("null");
      } else {
        sb.append(this.username);
      }
      first = false;
      sb.append(")");
      return sb.toString();
    }

    public void validate() throws org.apache.thrift.TException {
      // check for required fields
      // check for sub-struct validity
    }

    private void writeObject(java.io.ObjectOutputStream out) throws java.io.IOException {
      try {
        write(new org.apache.thrift.protocol.TCompactProtocol(new org.apache.thrift.transport.TIOStreamTransport(out)));
      } catch (org.apache.thrift.TException te) {
        throw new java.io.IOException(te);
      }
    }

    private void readObject(java.io.ObjectInputStream in) throws java.io.IOException, ClassNotFoundException {
      try {
        read(new org.apache.thrift.protocol.TCompactProtocol(new org.apache.thrift.transport.TIOStreamTransport(in)));
      } catch (org.apache.thrift.TException te) {
        throw new java.io.IOException(te);
      }
    }

    private static class sayHello_argsStandardSchemeFactory implements SchemeFactory {
      public sayHello_argsStandardScheme getScheme() {
        return new sayHello_argsStandardScheme();
      }
    }

    private static class sayHello_argsStandardScheme extends StandardScheme<sayHello_args> {

      public void read(org.apache.thrift.protocol.TProtocol iprot, sayHello_args struct) throws org.apache.thrift.TException {
        org.apache.thrift.protocol.TField schemeField;
        iprot.readStructBegin();
        while (true)
        {
          schemeField = iprot.readFieldBegin();
          if (schemeField.type == org.apache.thrift.protocol.TType.STOP) { 
            break;
          }
          switch (schemeField.id) {
            case 1: // USERNAME
              if (schemeField.type == org.apache.thrift.protocol.TType.STRING) {
                struct.username = iprot.readString();
                struct.setUsernameIsSet(true);
              } else { 
                org.apache.thrift.protocol.TProtocolUtil.skip(iprot, schemeField.type);
              }
              break;
            default:
              org.apache.thrift.protocol.TProtocolUtil.skip(iprot, schemeField.type);
          }
          iprot.readFieldEnd();
        }
        iprot.readStructEnd();

        // check for required fields of primitive type, which can't be checked in the validate method
        struct.validate();
      }

      public void write(org.apache.thrift.protocol.TProtocol oprot, sayHello_args struct) throws org.apache.thrift.TException {
        struct.validate();

        oprot.writeStructBegin(STRUCT_DESC);
        if (struct.username != null) {
          oprot.writeFieldBegin(USERNAME_FIELD_DESC);
          oprot.writeString(struct.username);
          oprot.writeFieldEnd();
        }
        oprot.writeFieldStop();
        oprot.writeStructEnd();
      }

    }

    private static class sayHello_argsTupleSchemeFactory implements SchemeFactory {
      public sayHello_argsTupleScheme getScheme() {
        return new sayHello_argsTupleScheme();
      }
    }

    private static class sayHello_argsTupleScheme extends TupleScheme<sayHello_args> {

      @Override
      public void write(org.apache.thrift.protocol.TProtocol prot, sayHello_args struct) throws org.apache.thrift.TException {
        TTupleProtocol oprot = (TTupleProtocol) prot;
        BitSet optionals = new BitSet();
        if (struct.isSetUsername()) {
          optionals.set(0);
        }
        oprot.writeBitSet(optionals, 1);
        if (struct.isSetUsername()) {
          oprot.writeString(struct.username);
        }
      }

      @Override
      public void read(org.apache.thrift.protocol.TProtocol prot, sayHello_args struct) throws org.apache.thrift.TException {
        TTupleProtocol iprot = (TTupleProtocol) prot;
        BitSet incoming = iprot.readBitSet(1);
        if (incoming.get(0)) {
          struct.username = iprot.readString();
          struct.setUsernameIsSet(true);
        }
      }
    }

  }

  public static class sayHello_result implements org.apache.thrift.TBase<sayHello_result, sayHello_result._Fields>, java.io.Serializable, Cloneable, Comparable<sayHello_result>   {
    private static final org.apache.thrift.protocol.TStruct STRUCT_DESC = new org.apache.thrift.protocol.TStruct("sayHello_result");

    private static final org.apache.thrift.protocol.TField SUCCESS_FIELD_DESC = new org.apache.thrift.protocol.TField("success", org.apache.thrift.protocol.TType.STRING, (short)0);

    private static final Map<Class<? extends IScheme>, SchemeFactory> schemes = new HashMap<Class<? extends IScheme>, SchemeFactory>();
    static {
      schemes.put(StandardScheme.class, new sayHello_resultStandardSchemeFactory());
      schemes.put(TupleScheme.class, new sayHello_resultTupleSchemeFactory());
    }

    public String success; // required

    /** The set of fields this struct contains, along with convenience methods for finding and manipulating them. */
    public enum _Fields implements org.apache.thrift.TFieldIdEnum {
      SUCCESS((short)0, "success");

      private static final Map<String, _Fields> byName = new HashMap<String, _Fields>();

      static {
        for (_Fields field : EnumSet.allOf(_Fields.class)) {
          byName.put(field.getFieldName(), field);
        }
      }

      /**
       * Find the _Fields constant that matches fieldId, or null if its not found.
       */
      public static _Fields findByThriftId(int fieldId) {
        switch(fieldId) {
          case 0: // SUCCESS
            return SUCCESS;
          default:
            return null;
        }
      }

      /**
       * Find the _Fields constant that matches fieldId, throwing an exception
       * if it is not found.
       */
      public static _Fields findByThriftIdOrThrow(int fieldId) {
        _Fields fields = findByThriftId(fieldId);
        if (fields == null) throw new IllegalArgumentException("Field " + fieldId + " doesn't exist!");
        return fields;
      }

      /**
       * Find the _Fields constant that matches name, or null if its not found.
       */
      public static _Fields findByName(String name) {
        return byName.get(name);
      }

      private final short _thriftId;
      private final String _fieldName;

      _Fields(short thriftId, String fieldName) {
        _thriftId = thriftId;
        _fieldName = fieldName;
      }

      public short getThriftFieldId() {
        return _thriftId;
      }

      public String getFieldName() {
        return _fieldName;
      }
    }

    // isset id assignments
    public static final Map<_Fields, org.apache.thrift.meta_data.FieldMetaData> metaDataMap;
    static {
      Map<_Fields, org.apache.thrift.meta_data.FieldMetaData> tmpMap = new EnumMap<_Fields, org.apache.thrift.meta_data.FieldMetaData>(_Fields.class);
      tmpMap.put(_Fields.SUCCESS, new org.apache.thrift.meta_data.FieldMetaData("success", org.apache.thrift.TFieldRequirementType.DEFAULT, 
          new org.apache.thrift.meta_data.FieldValueMetaData(org.apache.thrift.protocol.TType.STRING)));
      metaDataMap = Collections.unmodifiableMap(tmpMap);
      org.apache.thrift.meta_data.FieldMetaData.addStructMetaDataMap(sayHello_result.class, metaDataMap);
    }

    public sayHello_result() {
    }

    public sayHello_result(
      String success)
    {
      this();
      this.success = success;
    }

    /**
     * Performs a deep copy on <i>other</i>.
     */
    public sayHello_result(sayHello_result other) {
      if (other.isSetSuccess()) {
        this.success = other.success;
      }
    }

    public sayHello_result deepCopy() {
      return new sayHello_result(this);
    }

    @Override
    public void clear() {
      this.success = null;
    }

    public String getSuccess() {
      return this.success;
    }

    public sayHello_result setSuccess(String success) {
      this.success = success;
      return this;
    }

    public void unsetSuccess() {
      this.success = null;
    }

    /** Returns true if field success is set (has been assigned a value) and false otherwise */
    public boolean isSetSuccess() {
      return this.success != null;
    }

    public void setSuccessIsSet(boolean value) {
      if (!value) {
        this.success = null;
      }
    }

    public void setFieldValue(_Fields field, Object value) {
      switch (field) {
      case SUCCESS:
        if (value == null) {
          unsetSuccess();
        } else {
          setSuccess((String)value);
        }
        break;

      }
    }

    public Object getFieldValue(_Fields field) {
      switch (field) {
      case SUCCESS:
        return getSuccess();

      }
      throw new IllegalStateException();
    }

    /** Returns true if field corresponding to fieldID is set (has been assigned a value) and false otherwise */
    public boolean isSet(_Fields field) {
      if (field == null) {
        throw new IllegalArgumentException();
      }

      switch (field) {
      case SUCCESS:
        return isSetSuccess();
      }
      throw new IllegalStateException();
    }

    @Override
    public boolean equals(Object that) {
      if (that == null)
        return false;
      if (that instanceof sayHello_result)
        return this.equals((sayHello_result)that);
      return false;
    }

    public boolean equals(sayHello_result that) {
      if (that == null)
        return false;

      boolean this_present_success = true && this.isSetSuccess();
      boolean that_present_success = true && that.isSetSuccess();
      if (this_present_success || that_present_success) {
        if (!(this_present_success && that_present_success))
          return false;
        if (!this.success.equals(that.success))
          return false;
      }

      return true;
    }

    @Override
    public int hashCode() {
      List<Object> list = new ArrayList<Object>();

      boolean present_success = true && (isSetSuccess());
      list.add(present_success);
      if (present_success)
        list.add(success);

      return list.hashCode();
    }

    @Override
    public int compareTo(sayHello_result other) {
      if (!getClass().equals(other.getClass())) {
        return getClass().getName().compareTo(other.getClass().getName());
      }

      int lastComparison = 0;

      lastComparison = Boolean.valueOf(isSetSuccess()).compareTo(other.isSetSuccess());
      if (lastComparison != 0) {
        return lastComparison;
      }
      if (isSetSuccess()) {
        lastComparison = org.apache.thrift.TBaseHelper.compareTo(this.success, other.success);
        if (lastComparison != 0) {
          return lastComparison;
        }
      }
      return 0;
    }

    public _Fields fieldForId(int fieldId) {
      return _Fields.findByThriftId(fieldId);
    }

    public void read(org.apache.thrift.protocol.TProtocol iprot) throws org.apache.thrift.TException {
      schemes.get(iprot.getScheme()).getScheme().read(iprot, this);
    }

    public void write(org.apache.thrift.protocol.TProtocol oprot) throws org.apache.thrift.TException {
      schemes.get(oprot.getScheme()).getScheme().write(oprot, this);
      }

    @Override
    public String toString() {
      StringBuilder sb = new StringBuilder("sayHello_result(");
      boolean first = true;

      sb.append("success:");
      if (this.success == null) {
        sb.append("null");
      } else {
        sb.append(this.success);
      }
      first = false;
      sb.append(")");
      return sb.toString();
    }

    public void validate() throws org.apache.thrift.TException {
      // check for required fields
      // check for sub-struct validity
    }

    private void writeObject(java.io.ObjectOutputStream out) throws java.io.IOException {
      try {
        write(new org.apache.thrift.protocol.TCompactProtocol(new org.apache.thrift.transport.TIOStreamTransport(out)));
      } catch (org.apache.thrift.TException te) {
        throw new java.io.IOException(te);
      }
    }

    private void readObject(java.io.ObjectInputStream in) throws java.io.IOException, ClassNotFoundException {
      try {
        read(new org.apache.thrift.protocol.TCompactProtocol(new org.apache.thrift.transport.TIOStreamTransport(in)));
      } catch (org.apache.thrift.TException te) {
        throw new java.io.IOException(te);
      }
    }

    private static class sayHello_resultStandardSchemeFactory implements SchemeFactory {
      public sayHello_resultStandardScheme getScheme() {
        return new sayHello_resultStandardScheme();
      }
    }

    private static class sayHello_resultStandardScheme extends StandardScheme<sayHello_result> {

      public void read(org.apache.thrift.protocol.TProtocol iprot, sayHello_result struct) throws org.apache.thrift.TException {
        org.apache.thrift.protocol.TField schemeField;
        iprot.readStructBegin();
        while (true)
        {
          schemeField = iprot.readFieldBegin();
          if (schemeField.type == org.apache.thrift.protocol.TType.STOP) { 
            break;
          }
          switch (schemeField.id) {
            case 0: // SUCCESS
              if (schemeField.type == org.apache.thrift.protocol.TType.STRING) {
                struct.success = iprot.readString();
                struct.setSuccessIsSet(true);
              } else { 
                org.apache.thrift.protocol.TProtocolUtil.skip(iprot, schemeField.type);
              }
              break;
            default:
              org.apache.thrift.protocol.TProtocolUtil.skip(iprot, schemeField.type);
          }
          iprot.readFieldEnd();
        }
        iprot.readStructEnd();

        // check for required fields of primitive type, which can't be checked in the validate method
        struct.validate();
      }

      public void write(org.apache.thrift.protocol.TProtocol oprot, sayHello_result struct) throws org.apache.thrift.TException {
        struct.validate();

        oprot.writeStructBegin(STRUCT_DESC);
        if (struct.success != null) {
          oprot.writeFieldBegin(SUCCESS_FIELD_DESC);
          oprot.writeString(struct.success);
          oprot.writeFieldEnd();
        }
        oprot.writeFieldStop();
        oprot.writeStructEnd();
      }

    }

    private static class sayHello_resultTupleSchemeFactory implements SchemeFactory {
      public sayHello_resultTupleScheme getScheme() {
        return new sayHello_resultTupleScheme();
      }
    }

    private static class sayHello_resultTupleScheme extends TupleScheme<sayHello_result> {

      @Override
      public void write(org.apache.thrift.protocol.TProtocol prot, sayHello_result struct) throws org.apache.thrift.TException {
        TTupleProtocol oprot = (TTupleProtocol) prot;
        BitSet optionals = new BitSet();
        if (struct.isSetSuccess()) {
          optionals.set(0);
        }
        oprot.writeBitSet(optionals, 1);
        if (struct.isSetSuccess()) {
          oprot.writeString(struct.success);
        }
      }

      @Override
      public void read(org.apache.thrift.protocol.TProtocol prot, sayHello_result struct) throws org.apache.thrift.TException {
        TTupleProtocol iprot = (TTupleProtocol) prot;
        BitSet incoming = iprot.readBitSet(1);
        if (incoming.get(0)) {
          struct.success = iprot.readString();
          struct.setSuccessIsSet(true);
        }
      }
    }

  }

}
{% endcodeblock %}