---
title:  Spring 框架的线程池ThreadPoolTaskExecutor
description: 理解Spring 线程次中三个重要的参数 corePoolSize, maxPoolSize和queueCapacity
...


Spring 默认的任务执行器是ThreadPoolTaskExecutor， 其类关系参考下图

![ThreadPoolTaskExecutor.png](http://tech.icoding.tech/Spring-Boot-And-Spring-Cloud/ThreadPoolTaskExecutor.png)


这种模式引入队列缓冲来保证线程数量的可控。 

![ThreadPoolTaskExecutor-queue.png](http://tech.icoding.tech/Spring-Boot-And-Spring-Cloud/ThreadPoolTaskExecutor-queue.png)

### 理解三个重要的参数 corePoolSize, maxPoolSize和queueCapacity


先定义一个名词 来帮助理解： * requiredThreads - 某一时刻需要的线程数量

**在我们提交任务到执行器的时候，将经历以下逻辑： **

1. corePool无法满足需求的时候，任务将缓存至队列。 `（requiredThreads  > corePoolSize）&& requiredThreads < (corePoolSize + queueCapacity) `
2. 任务队列满了的时候执行器会创建新的线程到线程池。 `requiredThreads > (corePoolSize + queueCapacity)  && requiredThreads < queueCapacity + maxPoolSize`  
3. 队列满了， 池子到达上限了， 提交的任务会被拒绝，并抛出异常： `TaskRejectedException`

> 这个模式为什么要引入队列？ 为了通过缓冲的手段，有效控制线程数量。 因为在 Java 中，线程被映射到系统级线程，即操作系统的资源。如果我们不可控地创建线程，我们可能会很快耗尽这些资源 


### 通过Junit 测试来验证上面的逻辑
```
/**
 * 出自： https://www.baeldung.com/java-threadpooltaskexecutor-core-vs-max-poolsize
 * @date : 2022/4/1
 */

import org.junit.Assert;
import org.junit.jupiter.api.Test;
import org.springframework.core.task.TaskRejectedException;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ThreadLocalRandom;

public class ThreadPoolTaskExecutorUnitTest {

    void startThreads(ThreadPoolTaskExecutor taskExecutor, CountDownLatch countDownLatch, int numThreads) {
        for (int i = 0; i < numThreads; i++) {
            int index = i;
            taskExecutor.execute(() -> {
                try {
                    System.out.println("Started Threads: "+index);
                    Thread.sleep(100L * ThreadLocalRandom.current().nextLong(1, 10));
                    System.out.println("This is Threads: "+index);
                    countDownLatch.countDown();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
    }

    /**
     * 默认的时候corePoolSize=1， maxPoolSize和queueCapacity无限制, 始终只有一个线程
     */
    @Test
    public void whenUsingDefaults_thenSingleThread() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.afterPropertiesSet();
        log(taskExecutor);

        int numThreads = 10;
        CountDownLatch countDownLatch = new CountDownLatch(numThreads);
        this.startThreads(taskExecutor, countDownLatch, numThreads);

        while (countDownLatch.getCount() > 0) {
            Assert.assertEquals(1, taskExecutor.getPoolSize());
        }
    }

    /**
     * maxPoolSize + queueCapacity 满足不了线程数的需求的时候，任务会被拒绝，并抛出异常TaskRejectedException
     */
    @Test
    public void whenMaxPoolSizePlusQueueCapacity_lessThan_required_thenGotException() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(5);
        taskExecutor.setMaxPoolSize(10);
        taskExecutor.setQueueCapacity(5);
        taskExecutor.afterPropertiesSet();

        int numThreads = 16;
        CountDownLatch countDownLatch = new CountDownLatch(numThreads);
        try{
            this.startThreads(taskExecutor, countDownLatch, numThreads);
        }catch (Exception e){
            Assert.assertTrue(e instanceof TaskRejectedException);
        }
    }

    /**
     * 当queueCapacity + corePoolSize 满足需求线程数的时候，是不会去创建corePoolSize数量外的线程的。
      */
    @Test
    public void whenCorePoolSizeFiveAndMaxPoolSizeTen_thenFiveThreads() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(5);
        taskExecutor.setMaxPoolSize(10);
        taskExecutor.afterPropertiesSet();

        CountDownLatch countDownLatch = new CountDownLatch(10);
        this.startThreads(taskExecutor, countDownLatch, 10);

        while (countDownLatch.getCount() > 0) {
            Assert.assertEquals(5, taskExecutor.getPoolSize());
        }
    }

    /**
     * 放弃队列缓冲的时候(queueCapacity=0)，会根据需要创建 <=maxPoolSize数量的线程
     */
    @Test
    public void whenCorePoolSizeFiveAndMaxPoolSizeTenAndQueueCapacityZero_thenTenThreads() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(5);
        taskExecutor.setMaxPoolSize(10);
        taskExecutor.setQueueCapacity(0);
        taskExecutor.afterPropertiesSet();

        CountDownLatch countDownLatch = new CountDownLatch(10);
        this.startThreads(taskExecutor, countDownLatch, 10);

        while (countDownLatch.getCount() > 0) {
            Assert.assertEquals(10, taskExecutor.getPoolSize());
        }
    }

    /**
     * corePoolSize +  queueCapacity < 需要线程数量的时候, 需要创建
     */
    @Test
    public void whenCorePoolSizeFiveAndMaxPoolSizeTenAndQueueCapacityTen_thenTenThreads() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(5);
        taskExecutor.setMaxPoolSize(10);
        taskExecutor.setQueueCapacity(10);
        taskExecutor.afterPropertiesSet();

        CountDownLatch countDownLatch = new CountDownLatch(20);
        this.startThreads(taskExecutor, countDownLatch, 20);
        while (countDownLatch.getCount() > 0) {
            Assert.assertEquals(10, taskExecutor.getPoolSize());
        }
    }

    private void log(ThreadPoolTaskExecutor taskExecutor){
        System.out.println("Core Pool Size:" + taskExecutor.getCorePoolSize());
        System.out.println("Max Pool Size:" + taskExecutor.getMaxPoolSize());
    }
}
```

