title: java异常
date: 2015-08-24 18:32:51
tags: java exception
categories: exception
---
Java 异常处理是使用 Java 语言进行软件开发和测试脚本开发时不容忽视的问题之一，是否进行异常处理直接关系到开发出的软件的稳定性和健壮性。
<!--more-->
<h2>异常类层次结构</h2>
{% img /img/exception.jpg 600 377 异常类层次结构 %} 
<h2>异常分类</h2>
Error（错误）:是程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。例如，Java虚拟机运行错误（Virtual MachineError），当 JVM 缺少继续执行所需的内存资源时，将出现OutOfMemoryError。这些异常发生时，JVM一般选择终止线程。这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时，如Java虚拟机运行错误（Virtual MachineError）、类定义错误（NoClassDefFoundError）等。这些错误是不可查的，因为它们在应用程序的控制和处理能力之外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况。在Java中，错误通过Error的子类描述。

Exception（异常）:是程序本身可以处理的异常。Exception 类有一个重要的子类RuntimeException。RuntimeException类及其子类表示“JVM 常用操作”引发的错误。异常能被程序本身处理，错误则无法处理。 

<h2>cheked&unchecked exception</h2>
Java异常(包括Exception和Error)分为可查异常(checked exceptions)和不可查异常(unchecked exceptions)。

可查异常（编译器要求必须处置的异常）：正确的程序在运行中，很容易出现的、情理可容的异常状况。可查异常虽然是异常状况，但在一定程度上它的发生是可以预计的，而且一旦发生这种异常状况，就必须采取某种方式进行处理。 除了RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。不可查异常(编译器不要求强制处置的异常):包括运行时异常（RuntimeException与其子类）和错误（Error）。 

Exception 这种异常分两大类运行时异常和非运行时异常(编译异常)。程序中应当尽可能去处理这些异常。

运行时异常：都是RuntimeException类及其子类异常，如NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常)等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。

非运行时异常（编译异常）：是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。

{% link 异常处理注意事项 http://www.ibm.com/developerworks/cn/java/j-lo-exception-misdirection/#ibm-pcon true 注意事项 %}

异常代码举例
{% codeblock lang:java %}
public static void main(String[] args) throws Exception{/**一般不要在这里抛出异常，从设计耦合角度仔细考虑一下，
     这里的Exception污染了上层调用代码，因为调用层需要显式的利用try-catch捕捉，或者向更上层次进一步抛出。根据设计隔离原则**/
        List names = new ArrayList();//这里应该使用泛型，泛型能够保证类型的准确性
        names.add("John");
        names.add("Pierre");
        names.add(45); // Autoboxing wraps 45 into an Integer, which is stored
        try {
            for (int i = 0; i < names.size(); i++) {
                String name = (String) names.get(i);
                System.out.println(name);
            }
        }catch(ClassCastException e){//不要使用覆盖式异常处理块，不要用Exception处理所有异常
            System.out.println("class cast ex");//这里应该正确处理，而不是简单输出到控制台
            e.printStackTrace();
            throw new RuntimeException("error");//抛出后后面代码不再执行，包括finally代码块
            //return;
            //System.exit(2);//finally不执行
        }catch(Exception e){//捕获所有异常
            e.printStackTrace();
        }finally {
            System.out.println("if do finally");//在return之前执行
        }
        System.out.println("do it ?");
        //由于发送异常没有正确的处理，导致后续抛出更多异常
        System.out.println("you must not do this" + (String)names.get(2));
    }
{% endcodeblock %}
{% link 自定义异常使用建议 https://northconcepts.com/blog/2013/01/18/6-tips-to-improve-your-exception-handling/ true 异常使用建议 %}
异常处理小框架
{% img /img/exceptionhandle.png %}
