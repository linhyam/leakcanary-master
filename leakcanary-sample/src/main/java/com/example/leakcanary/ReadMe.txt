使用前加依赖，在application中添加
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    LeakCanary.install(this);
  }
}

debug build 中，如果检测到某个 activity 有内存泄露，LeakCanary 就是自动地显示一个通知。

工作机制

RefWatcher.watch() 创建一个 KeyedWeakReference 到要被监控的对象。

然后在后台线程检查引用是否被清除，如果没有，调用GC。(虚拟机垃圾收集(GC))

如果引用还是未被清除，把 heap 内存 dump 到 APP 对应的文件系统中的一个 .hprof 文件中。

在另外一个进程中的 HeapAnalyzerService 有一个 HeapAnalyzer 使用HAHA 解析这个文件。
(HeapAnalyzerService是在一个分离的进程中开始的，HeapAnalyzer通过使用HAHA（https://github.com/square/haha ）解析heap dump)

得益于唯一的 reference key, HeapAnalyzer 找到 KeyedWeakReference，定位内存泄露。

HeapAnalyzer 计算 到 GC roots 的最短强引用路径，并确定是否是泄露。如果是的话，建立导致泄露的引用链。

引用链传递到 APP 进程中的 DisplayLeakService， 并以通知的形式展示出来。

什么是内存泄露

一些对象有着有限的生命周期。当这些对象所要做的事情完成了，我们希望他们会被回收掉。但是如果有一系列对这个对象的引用，
那么在我们期待这个对象生命周期结束的时候被收回的时候，它是不会被回收的。它还会占用内存，这就造成了内存泄露。持续累加，内存很快被耗尽。

内存泄露(memory leak)，是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，
但是内存泄露堆积后果很严重，无论多少内存，迟早会被占光。memory leak最终会导致out of memory。

如何使用
1、创建examleapplication继承application,在oncreate内使用install来安装leakcanary
2、添加3个关于leakcanary依赖
3、这样当运行程序时，会自动安装leakcanary检测内存泄漏，并以通知形式返回哪里泄漏。


