---
title: 线程池
date: 2023-04-29 22:10:53
tags:
---

## 1. 一个简单的在播放视频中使用线程池的例子

在这个示例中，线程池创建了一个固定大小的线程池，最多同时运行 poolSize 个任务。每当用户播放一个短视频时，将启动下载、解码和渲染视频的任务，并在正确的时机等待任务完成或取消任务。使用的是submit()方法，返回一个future类型的对象。

```
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  import java.util.concurrent.Future;
  import java.util.concurrent.TimeUnit;
  
  public class ShortVideoApp {
      
      private ExecutorService threadPool; // 线程池
      
      public ShortVideoApp(int poolSize) {
          threadPool = Executors.newFixedThreadPool(poolSize); // 创建固定大小的线程池
      }
      
      public void playShortVideo(String videoUrl) throws InterruptedException {
          Future downloadFuture = threadPool.submit(() -> download(videoUrl)); // 提交下载任务并获取 Future 对象
          Future decodeFuture = threadPool.submit(this::decode); // 提交解码任务并获取 Future 对象
          Future renderFuture = threadPool.submit(this::render); // 提交渲染任务并获取 Future 对象
          
          try {
              downloadFuture.get(1, TimeUnit.SECONDS); // 等待下载完成，最多等待 1 秒钟
              decodeFuture.get(); // 等待解码完成
              renderFuture.get(); // 等待渲染完成
          } catch (Exception e) {
              downloadFuture.cancel(true); // 取消下载任务
              decodeFuture.cancel(true); // 取消解码任务
              renderFuture.cancel(true); // 取消渲染任务
              throw new InterruptedException("播放短视频失败：" + e.getMessage());
          }
      }
      
      public void shutdown() {
          threadPool.shutdown(); // 关闭线程池
          try {
              if (!threadPool.awaitTermination(60, TimeUnit.SECONDS)) { // 最多等待 60 秒钟
                  threadPool.shutdownNow(); // 强制关闭线程池
                  if (!threadPool.awaitTermination(60, TimeUnit.SECONDS)) { // 再次等待 60 秒钟
                      System.err.println("线程池强制关闭失败");
                  }
              }
          } catch (InterruptedException e) {
              threadPool.shutdownNow(); // 强制关闭线程池
              Thread.currentThread().interrupt(); // 重置中断标志位
          }
      }
      
      private void download(String videoUrl) {
          System.out.println("开始下载视频：" + videoUrl);
          try {
              Thread.sleep(3000); // 模拟下载耗时
          } catch (InterruptedException e) {
              Thread.currentThread().interrupt(); // 重置中断标志位
          }
          System.out.println("视频下载完成");
      }
      
      private Void decode() {
          System.out.println("开始解码视频");
          ...
      }
      
      private Void render() {
          System.out.println("开始渲染视频");
          ...
      }
  }
```

## 2.一个更简单的在播放视频中使用线程池的例子

使用execute()，没有返回值，这会导致不知道任务是否成功执行。

new Runnable() 是在 Java 中创建一个实现了 Runnable 接口的匿名内部类的实例。Runnable 接口是 Java 标准库中定义的一个函数式接口，它只包含一个无参方法 run()。

```
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  
  public class ShortVideoApp {
      
      private ExecutorService threadPool = Executors.newFixedThreadPool(4); // 创建固定大小的线程池
      
      public void playShortVideo(String videoUrl) {
          threadPool.execute(new Runnable() { // 提交任务给线程池执行
              @Override
              public void run() {
                  // 下载视频
                  download(videoUrl);
                  
                  // 解码视频
                  decode();
                  
                  // 渲染视频
                  render();
              }
          });
      }
      
      private void download(String videoUrl) {
          // 下载视频文件
      }
      
      private void decode() {
          // 解码视频文件
      }
      
      private void render() {
          // 渲染视频画面
      }
  }
```

使用 new Runnable() 创建一个实现了 Runnable 接口的匿名内部类的实例，可以方便地将该实例作为参数传递给其他方法或类的构造函数。除此之外，也可以直接使用Lambda表达式：

```
  threadPool.execute(() -> {
      // 下载视频
      download(videoUrl);      
      // 解码视频
      decode();
      // 渲染视频
      render();
  });
```

## 3. 使用线程池访问数据库

```
  import java.sql.Connection;
  import java.sql.DriverManager;
  import java.sql.ResultSet;
  import java.sql.SQLException;
  import java.sql.Statement;
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  
  public class DBUtil {
      // JDBC driver和数据库URL
      static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
      static final String DB_URL = "jdbc:mysql://localhost:3306/test";
  
      // 数据库用户名和密码
      static final String USER = "root";
      static final String PASS = "123456";
  
      // 线程池大小
      static final int THREAD_POOL_SIZE = 10;
  
      // 创建线程池
      private static ExecutorService executorService = Executors.newFixedThreadPool(THREAD_POOL_SIZE);
  
      public static void main(String[] args) {
          // 执行多个查询任务
          for (int i = 1; i <= 10; i++) {
              executorService.execute(new QueryTask(i));
          }
  
          // 关闭线程池
          executorService.shutdown();
      }
  
      // 查询任务
      private static class QueryTask implements Runnable {
          private int id;
  
          public QueryTask(int id) {
              this.id = id;
          }
  
          @Override
          public void run() {
              Connection conn = null;
              Statement stmt = null;
              try {
                  // 注册JDBC驱动
                  Class.forName(JDBC_DRIVER);
  
                  // 打开连接
                  conn = DriverManager.getConnection(DB_URL, USER, PASS);
  
                  // 执行查询
                  stmt = conn.createStatement();
                  String sql = "SELECT * FROM users WHERE id=" + id;
                  ResultSet rs = stmt.executeQuery(sql);
  
                  // 处理结果集
                  while (rs.next()) {
                      int userId = rs.getInt("id");
                      String userName = rs.getString("name");
                      System.out.println("用户ID：" + userId + "，用户名：" + userName);
                  }
  
                  // 关闭结果集、语句和连接
                  rs.close();
                  stmt.close();
                  conn.close();
              } catch (SQLException se) {
                  se.printStackTrace();
              } catch (Exception e) {
                  e.printStackTrace();
              } finally {
                  try {
                      if (stmt != null) stmt.close();
                  } catch (SQLException se2) {
                  }
                  try {
                      if (conn != null) conn.close();
                  } catch (SQLException se) {
                      se.printStackTrace();
                  }
              }
          }
      }
  }
```

这段代码演示了如何使用线程池来执行多个并发数据库查询任务。在这个例子中，我们使用了Java JDBC来访问MySQL数据库，并使用了内置的`Executors.newFixedThreadPool()`方法来创建大小为10的线程池。

每个查询任务都被封装在一个独立的`QueryTask`类中，这个类实现了`Runnable`接口，可以作为线程执行。在`main()`方法中，我们通过一个循环来创建多个`QueryTask`实例，并将它们提交给线程池执行。

在`run()`方法中，我们首先获取一个数据库连接（这里使用了JDBC的标准方式），然后执行一条查询语句，并将结果集打印出来。完成查询操作后，我们需要关闭结果集、语句和连接，以释放系统资源。

最后，在程序执行完所有查询任务后，我们调用线程池的`shutdown()`方法来关闭线程池。

**实际上，常见的数据库访问框架（如JDBC、Hibernate等）通常也会内置线程池功能，以便开发者能够方便地使用线程池来优化数据库访问，因此不需要手动创建线程池。上述代码只是一个例子。**

## 4.在一个短视频app的后端研发中，哪些地方会使用到线程池？



1. 请求处理：当用户发送请求到服务器时，服务器需要对请求进行处理，并返回相应结果。为了提高并发处理能力，可以使用线程池来处理请求。
2. 数据库访问：在短视频App的后端中，可能需要频繁地对数据库进行读写操作，使用线程池来管理数据库连接池，可以避免频繁创建和销毁数据库连接带来的开销，并且更好地控制系统的资源使用情况。
3. 文件上传和下载：在短视频App中，用户可能会上传和下载大量的视频和图片等文件。使用线程池来处理这些文件的上传和下载任务，可以提高系统的并发处理能力和响应速度。
4. 消息推送：在短视频App中，可能需要向用户推送各种消息，如点赞、评论、关注等。为了提高消息推送的并发能力，可以使用线程池来处理消息推送任务。
5. 异步处理：在短视频App的后端中，一些非核心业务逻辑可能需要异步处理，以避免阻塞主线程。例如，将视频转码、生成缩略图等操作可以放入线程池中异步执行。

总之，在短视频App的后端研发中，使用线程池可以提高系统的并发处理能力和响应速度，并且更好地管理系统的资源使用情况。
