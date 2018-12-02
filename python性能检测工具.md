# python性能检测工具

## 1. 分析程序的性能可以归结为回答四个基本问题
 - 正运行的多快
 - 速度瓶颈在哪里
 - 内存使用率是多少
 - 内存泄露在哪里

## 2.程序运行多块

## 2.1 用 time 粗粒度的计算时间
    
    $ time python yourprogram.py
    real    0m1.028s
    user    0m0.001s
    sys     0m0.003s
    
 - real - 表示实际的程序运行时间
 - user - 表示程序在用户态的cpu总时间
 - sys - 表示在内核态的cpu总时间

通过sys和user时间的求和，你可以直观的得到系统上没有其他程序运行时你的程序运行所需要的CPU周期。若sys和user时间之和远远少于real时间，那么你可以猜测你的程序的主要性能问题很可能与IO等待相关。

## 2.2 使用分析器逐行统计时间和执行频率
 - 1. 安装[line_profiler](https://github.com/rkern/line_profiler)
    
          pip3 install line_profiler
    
 - 2. 编写测试代码：为了使用这个工具，首先在你想测量的函数上设置 @profile 修饰符。不用担心，为了这个修饰符，你不需要引入任何东西。kernprof.py脚本会在运行时自动注入你的脚本。

          #primes.py 
          @profile
          def primes(n): 
              if n==2:
                  return [2]
              elif n<2:
                  return []
              s=range(3,n+1,2)
              mroot = n ** 0.5
              half=(n+1)/2-1
              i=0
              m=3
              while m <= mroot:
                  if s[i]:
                      j=(m*m-3)/2
                      s[j]=0
                      while j<half:
                          s[j]=0
                          j+=m
                  i=i+1
                  m=2*i+3
               return [2]+[x for x in s if x]
           primes(100)
   
 - 3.运行测试代码：-l 选项告诉 kernprof 把修饰符 @profile 注入你的脚本，-v 选项告诉 kernprof 一旦你的脚本完成后，展示计时信息
           kernprof -l -v primes.py
  输出结果如下：
            Wrote profile results to primes.py.lprof
            Timer unit: 1e-06 s

            Total time: 9.1e-05 s
            File: primes.py
            Function: primes at line 1

            Line #      Hits         Time  Per Hit   % Time  Line Contents
            ==============================================================
                 1                                           @profile
                 2                                           def primes(n): 
                 3         1          1.0      1.0      1.1      if n==2:
                 4                                                   return [2]
                 5         1          1.0      1.0      1.1      elif n<2:
                 6                                                   return []
                 7         1          4.0      4.0      4.4      s=list(range(3,n+1,2))
                 8         1          9.0      9.0      9.9      mroot = n ** 0.5
                 9         1          2.0      2.0      2.2      half=(n+1)/2-1
                10         1          0.0      0.0      0.0      i=0
                11         1          1.0      1.0      1.1      m=3
                12         5          4.0      0.8      4.4      while m <= mroot:
                13         4          3.0      0.8      3.3          if s[i]:
                14         3          3.0      1.0      3.3              j=(m*m-3)//2
                15         3          3.0      1.0      3.3              s[j]=0
                16        31         18.0      0.6     19.8              while j<half:
                17        28         15.0      0.5     16.5                  s[j]=0
                18        28         15.0      0.5     16.5                  j+=m
                19         4          2.0      0.5      2.2          i=i+1
                20         4          2.0      0.5      2.2          m=2*i+3
                21         1          8.0      8.0      8.8      return [2]+[x for x in s if x]
           

## 2.3 使用计时上下文管理器进行细粒度计时

## 3. 内存使用率是多少

## 3.1 安装[memory_profiler](https://github.com/fabianp/memory_profiler)
   
    pip3 install -U memory_profiler
    pip3 install psutil   # 建议安装psutil包，因为它可以大大改善memory_profiler的性能
    
## 3.2 测试代码编写与line_profiler一样，因此运行primes.py,如下：
     python -m memory_profiler primes.py

## 3.3 上述命令输出结果如下：
     Filename: primes.py

     Line #    Mem usage    Increment   Line Contents
     ================================================
          1   28.754 MiB   28.754 MiB   @profile
          2                             def primes(n): 
          3   28.754 MiB    0.000 MiB       if n==2:
          4                                     return [2]
          5   28.754 MiB    0.000 MiB       elif n<2:
          6                                     return []
          7   28.754 MiB    0.000 MiB       s=list(range(3,n+1,2))
          8   28.754 MiB    0.000 MiB       mroot = n ** 0.5
          9   28.754 MiB    0.000 MiB       half=(n+1)/2-1
         10   28.754 MiB    0.000 MiB       i=0
         11   28.754 MiB    0.000 MiB       m=3
         12   28.754 MiB    0.000 MiB       while m <= mroot:
         13   28.754 MiB    0.000 MiB           if s[i]:
         14   28.754 MiB    0.000 MiB               j=(m*m-3)//2
         15   28.754 MiB    0.000 MiB               s[j]=0
         16   28.754 MiB    0.000 MiB               while j<half:
         17   28.754 MiB    0.000 MiB                   s[j]=0
         18   28.754 MiB    0.000 MiB                   j+=m
         19   28.754 MiB    0.000 MiB           i=i+1
         20   28.754 MiB    0.000 MiB           m=2*i+3
         21   28.754 MiB    0.000 MiB       return [2]+[x for x in s if x]

## 4 内存泄漏
cPython解释器使用引用计数做为记录内存使用的主要方法。这意味着每个对象包含一个计数器，当某处对该对象的引用被存储时计数器增加，当引用被删除时计数器递减。当计数器到达零时，cPython解释器就知道该对象不再被使用，所以删除对象，释放占用的内存。如果程序中不再被使用的对象的引用一直被占有，那么就经常发生内存泄漏。下面为一些常用的内存泄漏分析工具。

## 4.1 安装[objgraph](https://mg.pov.lt/objgraph/)
    pip3 install objgraph
    
## 4.2 objgraph的应用
 - 显示占据python程序内存的头N个对象
 - 显示一段时间以后哪些对象被删除活增加了
 - 在我们的脚本中显示某个给定对象的所有引用

         objgraph.show_refs([y], filename='sample-graph.png')
       import objgraph
       x = []
       y = [x, [x], dict(x=x)]

       # 列出最多实例的类型
       print(objgraph.show_most_common_types(shortnames=False))
       # 显示两次调用之间的对象数量变化
       print(objgraph.show_growth(limit=None))
       # 获取指定类型的对象
       print(objgraph.by_type(y))
       # 查找反向引用
       print(objgraph.find_backref_chain(y, objgraph.is_proper_module))
