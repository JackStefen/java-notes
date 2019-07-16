# 1.示例
```
package demoGradleOne;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class App {

	public static void main(String[] args) {

        ThreadPoolExecutor executorPool = new ThreadPoolExecutor(2, 4, 10, TimeUnit.SECONDS, 
        		new ArrayBlockingQueue<Runnable>(2), new RejectHandler());
        for (int i = 0; i < 10; i++) {
            Runnable worker = new ThreadWorker("" + i);
            executorPool.execute(worker);
          }
        executorPool.shutdown(); // This will make the executor accept no new threads and finish all existing threads in the queue
        while (!executorPool.isTerminated()) { // Wait until all threads are finish,and also you can use "executor.awaitTermination();" to wait
        }
        System.out.println("所有的活都干活了...");
	}

}
```
ThreadWorker线程任务

```
package demoGradleOne;

public class ThreadWorker implements Runnable {
	private String command;
    
    public ThreadWorker(String s){
        this.command=s;
    }
 
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+" 开始干活，编号 = "+ command);
        processCommand(command);
        System.out.println(Thread.currentThread().getName()+" 活干完了。");
    }
 
    private void processCommand(String command) {
        try {
        	System.out.println("正在干, 编号：" + command + " 的活....");
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
    @Override
    public String toString(){
        return this.command;
    }
}

```
RejectHandler拒绝策略
```
package demoGradleOne;

import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;

public class RejectHandler implements RejectedExecutionHandler{

	@Override
	public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
		System.out.println("编号： " + r.toString() + " 任务被拒绝啦，没空搭理你...");
	}

}

```
运行结果：
```
pool-1-thread-2 开始干活，编号 = 1
pool-1-thread-4 开始干活，编号 = 5
编号： 6 任务被拒绝啦，没空搭理你...
pool-1-thread-3 开始干活，编号 = 4
正在干, 编号：4 的活....
pool-1-thread-1 开始干活，编号 = 0
编号： 7 任务被拒绝啦，没空搭理你...
正在干, 编号：5 的活....
正在干, 编号：1 的活....
编号： 8 任务被拒绝啦，没空搭理你...
正在干, 编号：0 的活....
编号： 9 任务被拒绝啦，没空搭理你...
pool-1-thread-3 活干完了。
pool-1-thread-2 活干完了。
pool-1-thread-2 开始干活，编号 = 2
正在干, 编号：2 的活....
pool-1-thread-1 活干完了。
pool-1-thread-4 活干完了。
pool-1-thread-3 开始干活，编号 = 3
正在干, 编号：3 的活....
pool-1-thread-2 活干完了。
pool-1-thread-3 活干完了。
所有的活都干活了...

```
## ThreadPoolExecutor Core and maximum pool sizes
ThreadPoolExecutor 会自动调整池的大小根据corePoolSize和maximumPoolSize的设置。
当一个任务通过execute方法提交后，如果当前的线程数少于corePoolSize，新的线程将会被创建用于处理请求。即便是其他工作线程在闲置状态。如果线程数大于corePoolSize但是少于maximumPoolSize,新的线程只有在queue满的情况下，才会被创建。通过设置corePoolSize和maxmumPoolSize成一样的值，可以创建一个固定大小的线程池。通过设置maximumPoolSize成一个基本上无限的值，比如Integer.MAX_VALUE，创建的线程池允许任意数量的并发任务，但是可以通过setCorePoolSize(int)和setMaximumPoolSize(int)动态的修改。
## ThreadPoolExecutor keepAliveTime
如果池子当前拥有超过corePoolSize个线程，多余的线程会被终止，如果他们已经闲置的时长超过keepAliveTime.这提供了一种在不主动使用池时减少资源消耗的方法。如果池稍后变得更活跃，则将构造新线程。该参数可以通过setKeepAliveTime(long, java.util.concurrent.TimeUnit)方法动态的改变。使用Long.MAX_VALUE TimeUnit.NANOSECONDS可以有效防止闲置线程。默认的，keep-alive策略只有在超过corePoolSize数据的线程时，才会被使用。但是allowCoreThreadTimeOut(boolean)方法可以用于将这种超时控制策略应用到核心线程上。只要keepAliveTime值为非零。

## ThreadPoolExecutor Queuing
任何BlockingQueue都可用于传输和保留提交的任务。 此队列的使用与池大小调整交互:
如果线程数少于corePoolSize个线程在运行，Executor永远喜欢添加一个新的线程，而不是放到队列中去。如果多个corePoolSize个线程在运行，Executor将会把请求任务放到队列中，而不是新添加线程。如果一个请求任务无法添加到队列中，新的线程将会创建在线程数少于maximumPoolSize。否则，任务将会被拒绝。
队列有三种常规策略:
- 直接交换. 默认选择为SynchronousQueue，它将任务交给线程而不另外保存它们。 在这里，如果没有可立即运行的线程，则尝试排队任务将失败，所以，一个新的线程将会被构建。此策略在处理可能具有内部依赖性的请求集时避免了锁定。直接切换通常需要无限制的maximumPoolSizes以避免拒绝新提交的任务。 这也将允许无限制的线程增长，当任务到达快于任务执行时。
- 无界队列. 使用无界队列（例如，没有预定义容量的LinkedBlockingQueue）将导致新任务在所有corePoolSize线程忙时在队列中等待。 因此，只会创建corePoolSize线程。maximumPoolSize设置将不会有任何影响当每个任务完全独立于其他任务时，这可能是适当的，所以，任务不能影响其他的任务执行。例如，在一个web服务器上，虽然这种排队方式可以用于平滑请求的瞬时突发，但是也存在线程无限制增长的可能性，在任务到达比任务执行快的情况下。
- 有界队列. 有界队列（例如，ArrayBlockingQueue）在与有值maximumPoolSizes一起使用时有助于防止资源耗尽，但可能更难以调整和控制，队列大小和最大池大小可以相互交换：使用大型队列和小型池最小化CPU使用率，OS资源和上下文切换开销，但可能导致人为的低吞吐量。 如果任务经常阻塞（例如，如果它们是I / O绑定的），系统可能能够为您提供比您允许的更多线程的时间。 使用小队列通常需要更大的池大小，这会使CPU更加繁忙，但可能会遇到不可接受的调度开销，这也会降低吞吐量。


