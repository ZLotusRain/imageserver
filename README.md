	5.10更改：
		1.picvserver、serv.h注释完善
		2.检索任务拆分成3级，改善上版线程池线程分配不合理的情况
		3.服务器实例、特征库实例做成单例
		4.做并发连接测试，ikendle，并发数1000，连接请求间隔5ms，发送字节数5.184KB，服务端netstat显示均是ESTABLISH。
		并发提到5000，netstat部分连接显示SERV_SEND。原因为服务端listen的数设置了1024，epoll最大监听检测也设成了1024。
	===================================================================
	名称：	图像检索服务器
	环境：	ubuntu14.04.4\opencv2.4.9\make\gdb\qt等
	服务器端主要任务【图片检索】：
		1.接收来自客户端发来的待检索图片
		2.提取待检索图片的特征，将此特征与特征库（图片库每张图片的特征）进行相似度计算(此处使用opencv的SIFT)。
		3.将相似度排行前10的图片发回给客户端
	服务端附带任务【更新+日志】：
		1.更新图片库+特征库
		2.定时写回硬盘的日志文件，方便服务器崩溃查错用
	服务器设计思路：
		【检索任务设计】			
			1.整个检索任务被拆分成3级：读任务（IO，接收客户端图片）+计算任务（cpu，计算相似度）+写任务（IO，发送top10相似度图片给客户端）
			2.主线程epollET检测是否有新连接、或新数据到来（新连接=》epoll加监听套接字；新数据=》将接收图片任务添加到读任务队列）
			3.除主线程外，另有3个任务调度队列，专门分别检测读、计算、写任务队列是否有任务，并从线程池中取工作线程，分派工作线程加以处理
			4.线程池中取出的工作线程处理具体的读、计算、写任务
		【其他设计】
			1.每个任务均需与特征库特征逐一计算比对。所以在服务端开启之初就将特征库全部调入内存。
			2.由于多进程共享不如多线程。选多线程，弃多进程
			3.由于线程过多会导致cpu频繁切换。选任务队列，弃一个线程处理一个任务的方式
			4.为避免线程的频繁创建销毁，增线程池
			5.由于select论询效率比不上epoll触发，选epollET，弃select
			6.为避免信号使线程崩溃，线程崩溃导致服务器崩溃，增信号处理
			7.为避免互斥量等忘记解锁、重复解锁，使用构造函数上锁，析构函数解锁
			8.为方便排错，增定时写回硬盘的错误日志

![image](https://github.com/tangsancai/imageserver/blob/master/result/result.jpg)
![image](https://github.com/tangsancai/imageserver/blob/master/result/result2.jpg)





