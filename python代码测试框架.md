# python单元测试框架

## 1.单元测试目的
 - 一个测试单元必须关注一个很小的功能函数，证明它是正确的；
 - 每个测试单元必须是完全独立的，必须能单独运行。这样意味着每一个测试方法必须重新加载数据，执行完毕后做一些清理工作。通常通过setUp()和setDown()方法处理；
 - 编写执行快速的测试代码。在某些情况下，测试需要加载复杂的数据结构，而且每次执行的时候都要重新加载，这个时候测试执行会很慢。因此，在这种情况下，可以将这种测试放置一个后台的任务中。
 - 采用测试工具并且学着怎么使用它。
 - 在编写代码前执行完整的测试，而且在编写代码后再重新执行一次。这样能保证你后来编写的代码不会破坏任何事情；
 - 在提交代码前执行完整的测试；
 - 如果在开发期间被打断了工作，写一个打断的单元测试，关于你下一步将要开发的。当你回来工作时，你能知道上一步开发到的指针；
 - 单元测试函数使用长的而且具有描述性的名字。在正式执行代码中，可能使用square()或sqr()取名，但是在测试函数中，你必须取像test_square_of_number_2()、test_square_negativer_number()这些名字，这些名字描述更加清楚；
 - 测试代码必须具有可读性；
 - 单元测试对新进的开发人员来说是工作指南。
 - 
## 2.单元测试框架：[unittest](https://docs.python.org/2/library/unittest.html#)

## 2.1 基本概念：Python中有一个自带的单元测试框架是unittest模块，用它来做单元测试，它里面封装好了一些校验返回的结果方法。
 - TestCase： 测试用例
 - TestSuite： 多个测试用例的集合
 - TestLoader：加载TestCase到TestSuite
 - TestRunner： 执行测试用例的,测试的结果会保存到TestResult实例中，包括运行了多少测试用例，成功了多少，失败了多少等信息

## 2.2 常用校验结果
    
        assertEqual(a, b)        a == b      
        assertNotEqual(a, b)     a != b      
        assertTrue(x)            bool(x) is True      
        assertFalse(x)           bool(x) is False      
        assertIsNone(x)          x is None     
        assertIsNotNone(x)       x is not None   
        assertIn(a, b)           a in b    
        assertNotIn(a, b)        a not in b

## 2.3 最基本的测试用例
       import unittest #导入unittest模块
       def load(x,y):
           if x > 6 and y >7:
               if x > 10 and y > 9:
                   return 3
       else:
           if x > 0 and y > 0:
               return 2
           else:
               return 1
       
       class MyTest(unittest.TestCase):  # 定义一个测试用例的类,继承unittest.TestCase
           def test_a_run(self):  # 测试用例。注意：定义一个测试用例，测试用例必须test开始，不然不会当做是测试用例
                self.assertEqual(load(3,6), 2)  
        
           def test_b_run(self):  # 测试用例
                self.assertEqual(load(20,30), 3)
           
       if __name__ == '__main__':
           unittest.main()     #运行所有的测试用例 

## 2.4 生成测试报告
       import unittest
       import HtmlTestRunner,xmlrunner #导入生成报告的模块
       
       def load(x,y):
           if x > 6 and y >7:
               if x > 10 and y > 9:
                   return 3
       else:
           if x > 0 and y > 0:
               return 2
           else:
               return 1
       
       class MyTest(unittest.TestCase): 
           def test_a_run(self):  
           '''a'''                 # 描述，会同测试用例标题一并显示在测试报告里
                self.assertEqual(load(3,6), 2)  
        
           def test_b_run(self):  
                self.assertEqual(load(20,30), 3)
                
       class MyTest2(unittest.TestCase): 
           def test_c_run(self):
                self.assertEqual(load(3,6), 2)  
        
           def test_d_run(self): 
                self.assertEqual(load(20,30), 3)
                
       if __name__ == '__main__':
           suite=unittest.TestSuite()  #定义一个测试套件
           suite.addTest(MyTest('test_a_run'))     #添加测试用例方法一：往测试套件里新增一条测试用例，测试用例只能用一种方法，同时两种会报错
           suite.addTest(MyTest('test_b_run'))
           # suite.addTest(unittest.makeSuite(MyTest))  #添加测试用例方法二：往测试套件里新增这个类下的所有测试用例
           # suite.addTest(unittest.makeSuite(MyTest2)) 
           
           runner=HtmlTestRunner.HTMLTestRunner(output="testreport")  #生成测试报告方法一：HtmlTestRunner，指定输出目录
           runner.run(suite) 
         
## 2.5 unittest中的setUp和tearDown的应用

    import unittest
    class MyTest(unittest.TestCase):
        @classmethod
        def setUpClass(cls):  #类开始前运行，比如在执行这些用例之前需要备份数据库
            print('1')
        @classmethod
        def tearDownClass(cls):   #类结束后运行，比如在执行这些用例之后需要还原数据库
            print('0')
        def setUp(self):  #测试用例执行前运行
            print('a')
        def tearDown(self):   #测试用例执行后运行
            print('z')
        def testa(self):
            print('测试用例1')
           self.assertEqual(1,1)
        def testb(self):
           print('测试用例2')
           self.assertEqual(1,1)
    if __name__=='__main__':
        unittest.main()

## 2.6 unittest批量读取python文件获取所有测试用例

    if __name__ == '__main__':
        suite = unittest.TestSuite()    #创建测试套件
        all_cases = unittest.defaultTestLoader.discover('.','test_*.py')
        #找到某个目录下所有的以test开头的Python文件里面的测试用例
        for case in all_cases:
            suite.addTests(case)    #把所有的测试用例添加进来
        runner = HtmlTestRunner.HTMLTestRunner(output="testreport")
        runner.run(suite)


## 参考链接：
https://www.cnblogs.com/znyyy/p/8086281.html

## 其他单元测试框架
[pytest](https://docs.pytest.org/en/latest/getting-started.html)

## 应用
[代码覆盖度测试]()

