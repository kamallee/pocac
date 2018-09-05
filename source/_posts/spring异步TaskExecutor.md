title: spring异步TaskExecutor 
date: 2016-01-13 17:54:25
tags: spring 异步 TaskExecutor
categories: spring
---
JDK5.0新增了并发工具包java.util.concurrent,其中执行器Executor是其中的一个重要的类。Executor的存在是为了方便并发操作，对Runnable实例的执行进行了抽象，使实现者可以提供更为丰富的实现。Spring为Executor的处理引入了新的抽象层，继而诞生TaskExecutor和TaskScheduler。
<!--more-->
<h2>spring配置</h2>
{% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="
      http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
      http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
      http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
      http://www.springframework.org/schema/task
      http://www.springframework.org/schema/task/spring-task-3.0.xsd">

    <task:annotation-driven executor="myExecutor" scheduler="myScheduler"/>
    <task:executor id="myExecutor" pool-size="5"/>
    <task:scheduler id="myScheduler" pool-size="10"/>
</beans>
{% endcodeblock %}
<h2>具体使用</h2>
{% codeblock lang:java %}
package com.crazyfish.kamal.test.testspringtask;

import org.springframework.beans.factory.InitializingBean;
import org.springframework.core.task.AsyncTaskExecutor;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * Created by kamal on 16/1/13.
 */
@Component
public class TestSpringTask implements InitializingBean{//初始化bean时执行下面方法
    @Resource(name="myExecutor")
    private AsyncTaskExecutor executor;


    public void afterPropertiesSet() throws Exception {
        final AtomicInteger count = new AtomicInteger();
        for(int i = 0;i < 100;i ++) {
            executor.execute(new Runnable() {
                public void run() {
                    System.out.println("xx:" + count.incrementAndGet());
                }
            });
        }
    }
}
{% endcodeblock %}

