# python代码覆盖率测试

## 1.代码覆盖率测试的目的
  - 把测试覆盖作为质量目标没有任何意义，而我们应该把它作为一种发现未被测试覆盖的代码的手段。

## 2.代码覆盖率的意义
 - 1.分析未覆盖部分的代码，从而反推在前期测试设计是否充分，没有覆盖到的代码是否是测试设计的盲点，为什么没有考虑到？需求/设计不够清晰，测试设计的理解有误，工程方法应用后的造成的策略性放弃等等，之后进行补充测试用例设计。
 - 2.检测出程序中的废代码，可以逆向反推在代码设计中思维混乱点，提醒设计/开发人员理清代码逻辑关系，提升代码质量。
 - 3.代码覆盖率高不能说明代码质量高，但是反过来看，代码覆盖率低，代码质量不会高到哪里去，可以作为测试自我审视的重要工具之一。
 
## 3.代码覆盖率检测工具：coverage

## 3.1 安装coverage模块
    pip install coverage
    
## 3.2 编写准备测试的代码main.py和测试代码testmain.py
    
    # content of main.py 
    def load(x,y):
        if x > 6 and y >7:
            if x > 10 and y > 9:
                return 3
        else:
            if x > 0 and y > 0:
                return 2
            else:
                return 1
    
    
    # content of testmain.py
    import unittest
    import main

    class MyTest(unittest.TestCase):  
        def tearDown(self):  
            print('111')
        def setUp(self):
            print('222')
        @classmethod
        def tearDownClass(self):
            print('Done')
        @classmethod
        def setUpClass(self):
            print('Start')
        def test_a_run(self):
            self.assertEqual(main.load(3,2),2)
        def test_b_run(self):
            self.assertEqual(main.load(20,30),3)
    if __name__ == '__main__':
        unittest.main()
  
  ## 3.3 代码覆盖率统计：
    coverage run testmain.py
    
  ## 3.4 代码覆盖率报告，在刚才的环境中执行如下命令：
    coverage report
  
  ## 3.5 代码覆盖率高级报告，在刚才的环境中执行如下命令：
    coverage html
  
   - 执行命令后可以在同级目录下看到名为”htmlcov”的文件夹，打开它。
   - 可以看到Coverage生成了一个漂亮、直观的网页来展示各部分代码的覆盖率
   - 它甚至直接列出了哪些代码执行了，哪些代码没有执行
  
  ## 3.6 总结
  Coverage很直观的展示了代码的运行情况，还生成了html页面，提供了高度可视化的细节分析
