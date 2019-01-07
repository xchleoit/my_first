1.使用synchronized修饰符修饰的方法
    1. synchronized可以修饰静态方法，如果修饰静态方法，则会锁住整个类
    
        public synchronized void print（）{
        Code
        }

 2. 使用synchronized修饰符的代码块(this is a change,so we should do more page to complete this page to change the line)

        Synchronized{
            Code
        }

    1. 同步是一个高开销的操作，需要注意，如果没有必要同步整个方法，可以选择同步必要的代码块
 3. 使用volatile修饰符修饰的变量（修饰符具有原子性）
    1. 线程的不同步主要出现在堆域的读写上，如果让域本身解决这个问题。就不用修改操作该域的方法了。用final修饰的域，有锁保护的域和volatile修饰的域都可解决该问题。
    Public volatile int  k =0;
 4. 使用Java重入锁RecentLock
    1. 属于Java.util.concurrent包内的类，实现了Lock接口。可重入(连续获取两次锁)互斥。
    2. 具有的方法：ReentrantLock（）[实现一个ReentrantLock类]，lock（），unlock(）
    3. ReentrantLock获取的是公平锁（按照先来后到的顺序获取锁，synchronized按照抢占的形似获取锁）
    
            Condition condition = new ReentrantLock().newCondition();//类似于等待通知机制
            condition.await();//等待
            condition.signal();//唤醒
    
            public class test {
                    private int account=100;
                    private Lock lock = new ReentrantLock();
                    public int getAccount() {
                        return account;
                    }
                    public void setAccount(int account) {
                        this.account = account;
                    }
                    public void save(int money) {
                        lock.lock();//锁住
                        try {
                            account += money;
                        }finally {
                            lock.unlock();//解锁
                        }
                    }
                }

 5.使用局部变量实现编程同步ThreadLocal
    1. 如果使用ThreadLocal管理变量，则每一个使用该变量的线程都会生成一个改变量的副本。
    2. 副本之间相互独立，每个线程都可随意修改自己的副本，而不会影响到其他的线程。当ThreadLocal结束后回收所有副本。
    3. ThreadLocal类常用的方法：
        1. ThreadLocal（）：创建一个线程本地向量
        2. get（）返回此线程局部变量中当前副本的值
        3. initValue（）：返回此线程局部变量的当前线程的“初始值”
        4. set（T value）：将当前线程的局部变量的当前线程副本中的值设为value
        
                /*
                 * 有多少个线程就有多少个account的副本
                 * new ThreadLocal();
                 * set(T value);
                 * get()
                 * */
                public class test {
                 	private static ThreadLocal<Integer> account = new ThreadLocal<Integer>() {
                 		protected Integer initiaValue() {
                 			return 100;
                 		}
                 	};
                 	public void save(int money) {
                 		account.set(account.get()+money);
                 	}
                 	public int getAccount() {
                 		return account.get();
                 	}
                 	
                }
                /*
                *ThreadLocal和同步机制
                *a。ThreadLocal与同步机制都是为了解决多线程中访问冲突的问题
                *前者采用“空间换时间”的方法，后者采用“时间换空间”的方法
                */

 6. 使用阻塞队列实现线程同步（LinkedBlockingQueue<E>）
    1. LinkedBlockingQueue<E>是基于已连接节点的，范围任意的阻塞队列（FIFO）
    2. LinkedBlockingQueue类常用方法
        1. LinkedBlockingQueue()：创建一个容量为Integer.MAX_VALUE的LinkedBlockingQueue队列
        2. put（E e）：在队尾添加一个元素，如果队列满则阻塞
        3. size（）：返回队列中元素的个数
        4. take（）：移除并返回队列头元素

                /*
                 * 采用阻塞队列实现买卖商品的线程同步
                 * 
                 * */
                public class test {
                 	private LinkedBlockingQueue<Integer> queue =new LinkedBlockingQueue<Integer>();
                 	//定义生产商品数量
                 	private static final int size =10;
                 	//定义启动线程的标志，为0时，启动生产商品的线程，为1时，启动消费商品的线程
                 	private int flag = 0;
                 	private class LinkedBlockThread implements Runnable{
                 		@Override
                 		public void run() {
                 			// TODO Auto-generated method stub
                 			int new_flag = flag++;
                 			System.out.println("启动线程"+new_flag);
                 			if(new_flag == 0) {
                 				for(int i =0;i<size ; i++) {
                 					int b = new Random().nextInt();
                 					System.out.println("生产商品"+b+"号");
                 					try {
                 						queue.put(b);
                 					}catch(InterruptedException e) {
                 						e.printStackTrace();
                 					}
                 					System.out.println("仓库中还有商品："+queue.size());
                 					try {
                 						Thread.sleep(100);
                 					}catch(InterruptedException e) {
                 						e.printStackTrace();
                 					}
                 				}	
                 			}else {
                 				for(int i =0;i<size/2 ; i++) {
                 					try {
                 						int n = queue.take();
                 						System.out.println("消费了"+n+"商品");
                 					}catch(InterruptedException e) {
                 						e.printStackTrace();
                 					}
                 					System.out.println("仓库中还有商品："+queue.size());
                 					try {
                 						Thread.sleep(100);
                 					}catch(InterruptedException e) {
                 						e.printStackTrace();
                 					}
                 				}
                 			}
                 		}	
                 	}
                 	public static void main(String[] args) {
                 		test t = new test();
                 		LinkedBlockThread lbt = t.new LinkedBlockThread();
                 		Thread thread1 = new Thread(lbt);
                 		Thread thread2 = new Thread(lbt);
                 		thread1.start();
                 		thread2.start();
                 	}
                }
                /*
                *BlockingQueue<E>定义了阻塞的常用方法，尤其是三种添加元素的方法，我们需要注意，当队列满时：
                *add（）方法会抛出异常
                *offer（）方法会返回false
                *put（）方法会阻塞
                */

 7. 使用原子变量时线程同步（util.concurrent.atomic）
    1. 需要使用线程同步的原因是普通变量的操作不是原子的
    2. java.util.concurrent.atomic包中提供了创建原子类型变量的工具类
    3. 使用该类可以简化线程同步
    *其中AtomicInteger表示可以用原子方式更新int的值，可用在应用程序中，但不能用于替换Integer。可扩展Number，允许哪些处理基于数字类的工具和实用工具进行统一访问。
    4. AtomicInteger类常用的方法：
        1. AtomicInteger（int initialValue）：创建具有给定初始值的新的AtomicInteger
        2. addAddGet(int dalta):    以原子方式将给定值与当前值相加
        3. get（）：获取当前值

                /*
                 * 开始10000张票
                 * 卖掉1000张（1000个线程，一个线程一张）
                 * AtomicInteger保证原子性
                 * */
                public class AtomicTongbu {
                 	class Ticket{
                 		public final AtomicInteger x = new AtomicInteger(10000);
                 	}
                 	class buyTicket implements Runnable{
                 		private Ticket t ;
                 		public buyTicket(Ticket t ) {
                 			this.t = t;
                 		}
                 		@Override
                 		public void run() {
                 			// TODO Auto-generated method stub
                 			t.x.getAndAdd(-1);
                 		}
                 		
                 	}
                 	public static void main(String[] args) throws InterruptedException{
                 		// TODO Auto-generated method stub
                 		Ticket t = new AtomicTongbu().new Ticket();
                 		System.out.println("还剩"+t.x.get()+"票");
                 		Runnable a =new AtomicTongbu().new buyTicket(t);
                 		Thread[] thread = new Thread[1000];
                 		for(int i =0;i<thread.length;i++) {
                 			thread[i]= new Thread(a);
                 		}
                 		for(int i =0;i<thread.length;i++) {
                 			thread[i].start();
                 		}
                 		Thread.sleep(500);
                 		System.out.println(t.x.get());
                 	}
                }
                /*
                *原子操作主要有：
                *对于引用变量和大多数原始变量（long和double除外）的读写操作
                *对于所有使用volatile修饰的变量（包括long和double）的读写操作
                */
