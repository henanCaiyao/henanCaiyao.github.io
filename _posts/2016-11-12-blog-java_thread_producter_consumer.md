---
layout: post
title: "java 多线程笔记"
description: "学习"
category: 技术学习
tags: [技术学习]
---
{% include JB/setup %}
# java多线程实现生产者消费者  
JAVA实现生产者消费者有两种方式： 1. wait()/notifyAll()。 2. 使用阻塞队列，JAVA并发包里提供了很多BlockedQueue的实现，LinkedBlockedQueue可以实现无界队列和有界队列；ArrayBlockedQueue只能实现有界队列。以下代码实现制作土司过程的一个仿真程序，制作土司有三个步骤： 1. 制作土司； 2. 在原始土司上涂黄油； 3. 在涂过黄油的土司上涂果酱。  

	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	import java.util.concurrent.LinkedBlockingQueue;
	import java.util.concurrent.TimeUnit;

	/**
	 * 制作土司仿真程序
	 * 
	 * 三个过程： 
	 * 1. 制作土司
	 * 2. 给土司抹黄油
	 * 3. 在抹过黄油的土司上涂果酱
	 * 
	 * @author caiyao_home
	 *
	 */
	public class ToasteMaker {
	/**
	 * 
	 * 三个任务分别完成 制作土司、给土司抹黄油、涂果酱
	 * 
	 * 
	 * 三个无界队列分别存放 原始土司、抹过黄油的土司、涂过果酱的土司
	 * 
	 * 
	 */
	public LinkedBlockingQueue<Toast> 
	dryQueue = new LinkedBlockingQueue<>(), 
	butterQueue = new LinkedBlockingQueue<>(), 
	jamQueue = new LinkedBlockingQueue<>();
	
	class DryTask implements Runnable{
		@Override
		public void run() {
			for(int i = 0 ; i < 10 ; i ++){
				Toast toast = new Toast();
				try {
					dryQueue.put(toast);
				} catch (InterruptedException e) {
					System.out.println("dryTask interrupted !");
				}
			}
		}
	}
	class ButterTask implements Runnable{
		@Override
		public void run() {
			while(true){
				try {
					Toast tempToast = dryQueue.take();
					tempToast.currentStatus = Toast.Status.BUTTERED;
					butterQueue.put(tempToast);
				} catch (InterruptedException e) {
					System.out.println("ButterTask interrupted !");
				}
			}
		}
	}
	class JamTask implements Runnable {
		@Override
		public void run() {
			while(true){
				try {
					Toast tempToast = butterQueue.take();
					tempToast.currentStatus = Toast.Status.JAM;
					jamQueue.put(tempToast);
				} catch (InterruptedException e) {
					System.out.println("JamTask interrupted !");
				}
			}
		}
	}
	class Consumer implements Runnable{
		@Override
		public void run() {
			while(true){
				try {
					Toast tempToast = jamQueue.take();
					System.out.println("consuemr eat one toast !" + tempToast.toString());
				} catch (InterruptedException e) {
					System.out.println("consumer interrupted !");
				}
			}
		}
	}
	public static void main(String[] args) {
		
		ToasteMaker test = new ToasteMaker();
		
		ExecutorService pool = Executors.newCachedThreadPool();
		
		DryTask dryTask = test.new DryTask();
		
		ButterTask butterTask = test.new ButterTask();
		
		JamTask jamTask = test.new JamTask();
		
		Consumer consumer = test.new Consumer() ;
		
		
		pool.execute(dryTask);
		
		pool.execute(butterTask);
		
		pool.execute(jamTask);
		
		pool.execute(consumer);
		
		try {
			TimeUnit.SECONDS.sleep(5);
			pool.shutdownNow();
			System.out.println("finish !");
		} catch (InterruptedException e) {
			System.out.println("main thread had been interrupted !");
		}
		
	}
	}


	class Toast {
	
	public enum Status{DRY,BUTTERED,JAM}
	
	public Status currentStatus = Status.DRY;
	
	public void butter(){this.currentStatus = Status.BUTTERED;}
	
	public void jam(){this.currentStatus = Status.JAM;}
	
	}



    
**坚持就会胜利，努力就能成功！**  
　　　　　　　　　　　　　　　　　　　　　　　update : 2016年11月5日
　　　　　　　　　　　　　　　　　　　　　　