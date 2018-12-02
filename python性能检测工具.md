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
