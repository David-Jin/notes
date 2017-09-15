# 以下文章摘录于网上资料

### Caffe学习之caffe具体运行流程分析

最近一段时间因实习需要，学习了一下caffe，在此陆陆续续记录一些和caffe相关的笔记。

我们都知道要运行一个caffe model的时候需要在命令行下输入：
```
./build/tools/caffe train -solver xxx.prototxt 
然后模型就跑起来了，但是具体程序的入口——main是哪个，执行的顺序是如何的可能不是特别清晰，为了更加理解caffe，在此对其流程做一个具体的分析。
分析方法很简单，细看运行model之后的，输出的Log，比如在此运行lenet model：
./build/tools/caffe train -solver examples/mnist/lenet_solver.prototxt
```

你会得到如下Log：
```
I0708 09:53:58.417726  3435 caffe.cpp:218] Using GPUs 0
I0708 09:53:58.560006  3435 caffe.cpp:223] GPU 0: TITAN X (Pascal)
I0708 09:54:01.458295  3435 solver.cpp:44] Initializing solver from parameters: 
...
I0708 09:54:01.458588  3435 solver.cpp:87] Creating training net from net file: examples/mnist/lenet_train_test.prototxt
I0708 09:54:01.459116  3435 net.cpp:294] The NetState phase (0) differed from the phase (1) specified by a rule in layer mnist
I0708 09:54:01.459144  3435 net.cpp:294] The NetState phase (0) differed from the phase (1) specified by a rule in layer accuracy
I0708 09:54:01.459230  3435 net.cpp:51] Initializing net from parameters:
...
...
```

观察程序运行的具体哪个.cpp 及哪一行代码，即可理解，因此可得caffe具体运行流程如下：

- step1: 命令行下输入./build/tools/caffe train -solver xxx.prototxt 运行了程序的入口caffe.cpp main()
- step2: caffe.cpp main()根据命令行输入的参数train 调用caffe.cpp train()
- step3: caffe.cpp train()读取xxx.prototxt的参数 调用solver.cpp Solver()的构造函数创建Solver对象
- step4: 创建Solver对象的时候需要调用solver.cpp Init()函数来初始化模型的网络
- step5: solver.cpp Init()函数调用solver.cpp InitTrainNet()和InitTestNets()函数来分别初始化训练和测试网络。
- step6: InitTrainNet() 通过xxx.prototxt 指定的xxxnet.prototxt读取net的参数，调用net.cpp Net()的构造函数，创建训练网络，
- step7: net.cpp Net()调用net.cpp Init()函数，通过for循环来(1)创建网络中每一个Layer对象，(2)设置bottom和top，(3）调用layer.cpp Setup()
- step8: 调用InitTestNets()创建测试网络，与InitTrainNet()类似
- step9: 运行返回到caffe.cpp train()中，利用创建好的solver对象调用solver.cpp Solve()函数
- step10: solver.cpp Solve() 调用 solver.cpp Step()函数，while循环迭代的次数，每次迭代 1）调用net.cpp ForwardBackward()来前向以及后向传播 2)solve.cpp ApplyUpdate()更新参数 3）每一定轮次运行solver.cpp TestAll()
- step11: 运行结束
可能有点绕，总结而言就是：

1.  入口函数caffe.cpp main()(调用caffe.cpp train()) 
2.  caffe.cpp train()中 创建solver对象(调用solver.cpp Init()) 
3.  solver.cpp Init()中 solver.cpp InitTrainNet()和InitTestNets()创建训练网络和测试网络 
4.  运行返回到caffe.cpp train()中，调用solver.cpp Solve()来训练网络，具体训练过程在solver.cpp Step()中实现，迭代直至程序结束
