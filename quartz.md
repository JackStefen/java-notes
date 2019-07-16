## 1.在项目里引入quartz

```

<dependency>

    <groupId>org.quartz-scheduler</groupId>

    <artifactId>quartz</artifactId>

    <version>2.3.0</version>

</dependency>

```

# 2.quartz的简单实例

```

package com.example.quartz_demo;

import org.quartz.JobDetail;

import org.quartz.Scheduler;

import static org.quartz.JobBuilder.newJob;

import static org.quartz.TriggerBuilder.newTrigger;

import static org.quartz.SimpleScheduleBuilder.*;

import org.quartz.SchedulerException;

import org.quartz.Trigger;

import org.quartz.impl.StdSchedulerFactory;

/**

* Hello world!

*

*/

public class App

{

    public static void main( String[] args )

    {

        try {

Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();

scheduler.start();

   try {

    Thread.sleep(100000);

   } catch (InterruptedException e) {

    e.printStackTrace();

   }

   scheduler.shutdown();

} catch (SchedulerException e) {

e.printStackTrace();

}

    }

}

```
启动程序，默认会启动10个线程，我们可以在配置文件中设置quartz运行的相关属性，在resources目录下新建一个quartz.properties文件，添加
```
org.quartz.scheduler.instanceName = MyScheduler
org.quartz.threadPool.threadCount = 3
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
```
再次启动程序，启动的线程就变成3个了，一个配置项为调度器的名称。第三个为设置Quartz数据的存储方式

再看一个例子：

```
package com.example.quartz_demo;

import org.quartz.JobDetail;
import org.quartz.Scheduler;
import static org.quartz.JobBuilder.newJob;
import static org.quartz.TriggerBuilder.newTrigger;
import static org.quartz.SimpleScheduleBuilder.*;
import org.quartz.SchedulerException;
import org.quartz.Trigger;
import org.quartz.impl.StdSchedulerFactory;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args )
    {
        try {
			Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
			scheduler.start();
			
			
			JobDetail job = newJob(HelloJob.class).
					withIdentity("job1", "group1").
					build();
			
			Trigger trigger = newTrigger().
					withIdentity("trigger1", "group1")
					.startNow()
					.withSchedule(simpleSchedule()
					.withIntervalInSeconds(10)
					.repeatForever())
					.build();
			scheduler.scheduleJob(job, trigger);
//			try {
//				Thread.sleep(100000);
//			} catch (InterruptedException e) {
//				e.printStackTrace();
//			}
//			scheduler.shutdown();
		} catch (SchedulerException e) {
			e.printStackTrace();
		}
    }
}

```
可以详细的设置任务的开始，频率等调度详情

# 3.trigger
withIntervalInSeconds秒级的任务
withRepeatCount指定运行次数后，仅仅运行argumnt+1次Job就停止运行Job
```
Trigger trigger = newTrigger()
				    .withIdentity("trigger3", "group1")
				    .startNow()  // if a start time is not given (if this line were omitted), "now" is implied
				    .withSchedule(simpleSchedule()
				        .withIntervalInSeconds(1)
				        .withRepeatCount(3)) // note that 10 repeats will give a total of 11 firings                  
				    .build();
```
withIntervalInMinutes分钟级的Job
withIntervalInHours小时级的Job
endAt指定任务的结束时间，到期后停止Job的调度
```
Trigger  trigger = newTrigger()
    .withIdentity("trigger7", "group1")
    .withSchedule(simpleSchedule()
        .withIntervalInMinutes(5)
        .repeatForever())
    .endAt(dateOf(22, 0, 0))
    .build();
```

# 4.使用数据库级别的存储任务

- 导入`D:\UserData\Downloads\quartz-2.2.3\docs\dbTables\tables_mysql_innodb.sql`quartz包中的sql脚步到数据库中
![sql文件导入到数据库中的表.png](https://upload-images.jianshu.io/upload_images/3004516-febc48cdd08a84ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 项目中，  `pom.xml`中引入mysql数据库的连接驱动
```
<dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.35</version>
    </dependency>
```
- 配置 quartz.properties文件
```
org.quartz.scheduler.instanceName = MyScheduler
org.quartz.scheduler.instanceId = AUTO
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount = 10
org.quartz.threadPool.threadPriority = 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread = true
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.tablePrefix = QRTZ_
org.quartz.jobStore.useProperties = true
org.quartz.jobStore.isClustered = false
org.quartz.jobStore.clusterCheckinInterval = 20000
org.quartz.jobStore.misfireThreshold = 60000
org.quartz.jobStore.dataSource = qzDs
org.quartz.dataSource.qzDs.driver = com.mysql.jdbc.Driver
org.quartz.dataSource.qzDs.URL = jdbc:mysql://localhost:3306/qurtz_db
org.quartz.dataSource.qzDs.user = root
org.quartz.dataSource.qzDs.password = 123456
org.quartz.dataSource.qzDs.maxConnections = 10
```
- 实例代码：
```
package com.example.quartz_demo;

import org.quartz.JobDetail;
import org.quartz.JobKey;
import org.quartz.Scheduler;
import static org.quartz.JobBuilder.newJob;
import static org.quartz.TriggerBuilder.newTrigger;
import static org.quartz.SimpleScheduleBuilder.*;
import org.quartz.SchedulerException;
import org.quartz.SchedulerFactory;
import org.quartz.Trigger;
import org.quartz.TriggerKey;
import org.quartz.impl.StdSchedulerFactory;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args )
    {
        try {
        	SchedulerFactory factory = new StdSchedulerFactory();
			Scheduler scheduler = factory.getScheduler();
			scheduler.start();
			TriggerKey tKey = new TriggerKey("trigger1", "group1");
			JobKey jKey = new JobKey("job1", "group1");

			if(scheduler.checkExists(tKey)){
				Trigger trigger = scheduler.getTrigger(tKey);
				scheduler.rescheduleJob(tKey, trigger);
			}else{
			
				JobDetail job = newJob(HelloJob.class).
						withIdentity(jKey).
						build();
				
				Trigger trigger = newTrigger().
						withIdentity(tKey)
						.startNow()
						.withSchedule(simpleSchedule()
						.withIntervalInSeconds(10)
						.repeatForever())
						.build();
	//			Trigger trigger = newTrigger()
	//				    .withIdentity("trigger3", "group1")
	//				    .startNow()  // if a start time is not given (if this line were omitted), "now" is implied
	//				    .withSchedule(simpleSchedule()
	//				        .withIntervalInSeconds(1)
	//				        .withRepeatCount(3)) // note that 10 repeats will give a total of 11 firings                  
	//				    .build();
				scheduler.scheduleJob(job, trigger);
			}
//			try {
//				Thread.sleep(100000);
//			} catch (InterruptedException e) {
//				e.printStackTrace();
//			}
//			scheduler.shutdown();
		} catch (SchedulerException e) {
			e.printStackTrace();
		}
    }
}

```
代码中，进行了已存在触发器的检查，如果数据库中已经存在，再创建就会引起异常
```
org.quartz.ObjectAlreadyExistsException: Unable to store Job : 'group1.job1', because one already exists with this identification.
	at org.quartz.impl.jdbcjobstore.JobStoreSupport.storeJob(JobStoreSupport.java:1113)
	at org.quartz.impl.jdbcjobstore.JobStoreSupport$2.executeVoid(JobStoreSupport.java:1067)
	at org.quartz.impl.jdbcjobstore.JobStoreSupport$VoidTransactionCallback.execute(JobStoreSupport.java:3765)
	at org.quartz.impl.jdbcjobstore.JobStoreSupport$VoidTransactionCallback.execute(JobStoreSupport.java:3763)
	at org.quartz.impl.jdbcjobstore.JobStoreSupport.executeInNonManagedTXLock(JobStoreSupport.java:3849)
	at org.quartz.impl.jdbcjobstore.JobStoreTX.executeInLock(JobStoreTX.java:93)
	at org.quartz.impl.jdbcjobstore.JobStoreSupport.storeJobAndTrigger(JobStoreSupport.java:1063)
	at org.quartz.core.QuartzScheduler.scheduleJob(QuartzScheduler.java:855)
	at org.quartz.impl.StdScheduler.scheduleJob(StdScheduler.java:249)
	at com.example.quartz_demo.App.main(App.java:53)
```