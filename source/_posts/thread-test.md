---
title: java多线程测试
categories: java
date: 2018-3-24 10:32:02
tags: [java,多线程]
comments: true
---

**直接上例子**

<!-- more -->

```java
public class ApiTest {
     
    ApplicationContextUtil applicationContext = ApplicationContextUtil
            .getInstance();
 
    RecommendInfoService recommendInfoService = (RecommendInfoService) applicationContext
            .getBean("recommendInfoService");
     
    @Test
    public void multiThreads() throws InterruptedException, ExecutionException {
        int number = 3; //线程数  
        ExecutorService executorService = Executors.newFixedThreadPool(number);  
           
        List<Future<String>> results = new ArrayList<Future<String>>();  
        for (int i=0; i < number; i++) { //非阻塞地启动number个线程，由Future接收结果  
            Future<String> future = executorService.submit(new MyThread());  
            //Thread.sleep(300);  
            results.add(future);  
        }  
           
        for(Future<String> f : results){ //从Future中获取结果，打印出来  
            String ss = f.get();  
            System.out.println(ss);  
        }  
    }
     
    class MyThread implements Callable<String>{  
        @Override 
        public String call() throws Exception {  
           // return recommendInfoService.rtest("1100070023874046224810008", "1100121023874101446610014");  
            return "1";  
        }  
    }
}
```