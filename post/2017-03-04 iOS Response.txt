Title: iOS The Responder Chain
Date: 2017-03-04 17:21

  iOS App的事件分发和响应机制是基于Responder Chain模式的。今天我详细的讨论一下iOS里面的Responder Chain。

### 事件的传递
  iOS设备有许多用户产生的事件，比如常见的Touch Event，Motion。当这些event产生时UIKit把他们封装成对应的Event对象然后放到当前活跃的App的事件队列里面，这些event对象包含了处理事件所需要的所有信息。UIApplication单例对象在事件队列里面拿到event对象以后会将其沿着特定的路径进行传递分发直到事件得到处理。一般来说event会首先交给key window对象，然后由window转发给一个首先处理事件的对象，首先处理事件的对象取决于事件的类型。
>*  **Touch事件：**对于touch event来说，window尝试将touch event传递给touch所在view。寻找touch view的叫做Hit Test，具体的过程就是从window开始遍历整个view的树，如果touch发生的point被包含view之内怎递归寻找它的subviews，否则停止。

>* **Motion和其它一些事件**则是由window传递给First Responder。

### Responder Chain 是由Responder组成的
  大部分事件都依赖Responder Chain来传递事件。Responder Chain是由一系列的Responder构成，从First Responder开始，以UIApplication结尾。如果First Responder不能处理事件，它将事件传给Next Responder，以此类推，直到事件被处理或者是没有更多的Responder。

  First Responder就是指定首先收到时间的对象。一般来说First Responder是一个UIView。一个对象要成为First Responder有两个条件：
>* 1 实现 canBecomeFirstResponder 返回YES。
>* 2 调用 becomeFirstResponder 方法。

```
需要特别注意一点：调用becomeFirstResponder方法的时候最好确保响应链（Responder Chain）已经创建好。需要在viewdidappear中调用，而不是viewwillappear。
```