---
title: 【并发】多线程+CyclicBarrier实现计算任务拆解
date: 2023-09-28 23:50:10
tags:
  - 并发
  - 多线程
category: 
  - 后端
---



## 场景

假设有一个大的计算任务，**需要对一个数组中的所有元素进行求和**。我们可以将这个任务拆分成多个子任务，每个子任务负责求和一部分元素，最后将所有子任务的结果累加得到最终结果

## CyclicBarrier

> 位于java.util.concurrent.CyclicBarrier
>
> 作用 : 它允许一组线程互相等待，直到到达某个公共屏障点 (Common Barrier Point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。**因为该 Barrier 在释放等待线程后可以重用，所以称它为循环( Cyclic ) 的 屏障( Barrier )** 。

### 构造方法

**CyclicBarrier(int parties, Runnable barrierAction)**

- 创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，
- 并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。

### 其他方法

**barrier.await()**

- 会阻塞当前相关的线程

## 代码

**有一个长度为 10 的数组，需要计算所有元素的和**。

我们`创建了两个线程参与计算`，`并将数组拆分成两部分，分别交给不同的线程计算`。

每个线程计算完部分和后，通过 `CyclicBarrier等待其他线程完成`，`最后在栅栏点处进行部分和的累加得到总和`。

```java
/**
 * 多线程+CyclicBarrier 实现计算任务拆解
 */
public class MultiThreadSum {

    private static int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};// 模拟大的计算任务
    private static int threadCount = 2; // 假设有两个线程参与计算
    private static int[] partialSums; // 用于存储各个线程计算的部分和

    public static void main(String[] args) {
        partialSums = new int[threadCount]; // 初始化部分和数组

        // 创建一个 CyclicBarrier，指定参与的线程数为 threadCount，
        // 当所有线程都达到栅栏点时，会触发后续的任务
        CyclicBarrier barrier = new CyclicBarrier(threadCount, new Runnable() {
            @Override
            public void run() {
                int totalSum = 0;
                for (int sum : partialSums) {
                    totalSum += sum;
                }
                System.out.println("总和为：" + totalSum);
            }
        });

        // 创建并启动多个线程，每个线程负责计算部分和
        for (int i = 0; i < threadCount; i++) {
            /**
             *  线程0的索引：startIndex: 0 * (10 / 2) = 0 , endIndex: 1 * (10 / 2) = 5
             *  线程1的索引: startIndex: 1 * (10 / 2) = 5 ,  2 * (10 / 2) =  10
             */
            int startIndex = i * (numbers.length / threadCount);
            int endIndex = (i + 1) * (numbers.length / threadCount);
            /**
             * 线程0计算范围 [0,5, barrier]
             * 线程1计算范围 [5,10 barrier]
             */
            new Thread(new PartialSumTask(startIndex, endIndex, barrier)).start();
        }
    }

    static class PartialSumTask implements Runnable {
        private int startIndex;
        private int endIndex;
        private CyclicBarrier barrier;

        public PartialSumTask(int startIndex, int endIndex, CyclicBarrier barrier) {
            this.startIndex = startIndex;
            this.endIndex = endIndex;
            this.barrier = barrier;
        }

        @Override
        public void run() {
            int sum = 0;
            for (int i = startIndex; i < endIndex; i++) {
                sum += numbers[i];
            }
            /**
             * 将线程计算任务结果加到公共变量partialSums中
             * 线程0总和累计到partialSums[0]： partialSums[0/(10/2)] ;
             * 线程1总和累计到partialSums[1]：partialSums[5/(10/2)]  ;
             */
            partialSums[startIndex / (numbers.length / threadCount)] = sum;
            System.out.println(Thread.currentThread().getName() + " 计算的部分和为：" + sum);

            try {
                // 当前线程达到栅栏点，等待其他线程到达
                barrier.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

`barrier.await()` 会阻塞当前线程，也就是在 `PartialSumTask` 中执行的线程。它会等待所有参与的线程都到达栅栏点之后才会继续执行后续的任务。所以在这里，当每个 `PartialSumTask` 线程执行到 `barrier.await()` 时，它会等待其他线程也执行到相同的位置。一旦所有参与的线程都到达栅栏点，`CyclicBarrier` 就会释放所有线程，它们可以继续执行后续的任务。

## 结果

```shell
Thread-0 计算的部分和为：15
Thread-1 计算的部分和为：40
总和为：55
```
