---
title: 从零学自动化测试之一文学会junit4(二)
categories:
- 从零学自动化测试
tags:
- junit

---
### 说在前面 ###

使用java语言做测试开发的同学，必须要掌握一项测试框架，可能是junit、testng、spock,这里我们选择junit4上手，虽然2016年junit5已经问世，但是还不是主流，testng适合复杂场景测试，spock不时候入门学习。
这里的内容主要源于[官网](https://junit.org/junit4/)
<!-- more -->

### 准备环境 ###
这里我们使用intellij idea 新建一个gradle的java工程，新工程会默认使用junit4作为测试框架。

### 创建第一个用例 ###

我们使用intellij idea 创建被测试类，这个类主要是用来计算+法语句：

```
public class Calculator {
  public int evaluate(String expression) {
    int sum = 0;
    for (String summand: expression.split("\\+"))
      sum += Integer.valueOf(summand);
    return sum;
  }
}
```
创建测试类:
```
public class CalculatorTest {
  @Test
  public void evaluatesExpression() {
    Calculator calculator = new Calculator();
    int sum = calculator.evaluate("1+2+3");
    assertEquals(6, sum);
  }
}
```
junit4创建测试类只需要使用@Test注解标注即可，不像junit3还得继承TestCase，方法名还有限制等等。
右键执行输出：

```
JUnit version 4.12
.
Time: 0,006

OK (1 test)
```

### 常用注解 ###

|注解|用途|
|--|--|
|@Test|定义一个测试方法|
|@BeforeClass|所有的测试方法前被执行，public static修饰，且只执行一次|
|@AfterClass|所有的测试方法后被执行，public static修饰，且只执行一次|
|@Before|每一个测试方法前被执行一次|
|@After|每一个测试方法后被执行一次|
|@Ignore|忽略|
|@RunWith|修改运行器|
|@Rule|使用规则|

```
public class TestFixturesExample {
  static class ExpensiveManagedResource implements Closeable {
    @Override
    public void close() throws IOException {}
  }

  static class ManagedResource implements Closeable {
    @Override
    public void close() throws IOException {}
  }

  @BeforeClass
  public static void setUpClass() {
    System.out.println("@BeforeClass setUpClass");
    myExpensiveManagedResource = new ExpensiveManagedResource();
  }

  @AfterClass
  public static void tearDownClass() throws IOException {
    System.out.println("@AfterClass tearDownClass");
    myExpensiveManagedResource.close();
    myExpensiveManagedResource = null;
  }

  private ManagedResource myManagedResource;
  private static ExpensiveManagedResource myExpensiveManagedResource;

  private void println(String string) {
    System.out.println(string);
  }

  @Before
  public void setUp() {
    this.println("@Before setUp");
    this.myManagedResource = new ManagedResource();
  }

  @After
  public void tearDown() throws IOException {
    this.println("@After tearDown");
    this.myManagedResource.close();
    this.myManagedResource = null;
  }

  @Test
  public void test1() {
    this.println("@Test test1()");
  }

  @Test
  public void test2() {
    this.println("@Test test2()");
  }
}
```
输出：

> @BeforeClass setUpClass
@Before setUp
@Test test2()
@After tearDown
@Before setUp
@Test test1()
@After tearDown
@AfterClass tearDownClass


### Junit4断言 ###
junit4底层使用了org.hamcrest这个断言框架，除此之外自己也封装了一些断言方式，在org.junit.Asert中，
这里就简单列几个：

|方法|描述|
| -- | -- |
|assertTrue(String message, boolean condition)|检查条件是否为真|
|assertFalse(String message, boolean condition)|检查条件是否为假|
|assertEquals(String message, Object expected,Object actual)|检查是否相等|
|assertNotNull(String message, Object object)|检查对象是否不为空|
|assertSame(String message, Object expected, Object actual)|检查两个对象引用是否引用同一对象（即对象是否相等）|

使用的使用通过静态导入非常方便：

```
import static org.junit.Assert.*;
public class AssertionsTest {  
    @Test  
    public void testAssertNull() {  
        String str = null;  
        assertNull(str);  
    }  
}
```
除此之外，junit4包装了hamcrest的assertThat，方便各种断言,例如我们想断言接口返回字符串中是否包含"color"或者"colour"

```
assertTrue(responseString.contains("color") || responseString.contains("colour"));
// ==> failure message: 
// java.lang.AssertionError:


assertThat(responseString, anyOf(containsString("color"), containsString("colour")));
// ==> failure message:
// java.lang.AssertionError: 
// Expected: (a string containing "color" or a string containing "colour")
//      got: "Please choose a font"
```

### junit4运行器 ###
junit4中的所有测试方法都是通过测试运行器执行的，junit4默认使用BlockJUnit4ClassRunner，但是我们可以通过@RunWith注解指定运行器来达到不同的运行器，例如使用系统默认的Suite这个套件运行器可以运行多个类中的测试方法：

```
@RunWith(Suite.class)
 @SuiteClasses({ATest.class, BTest.class, CTest.class})
 public class ABCSuite {
 }
```
suite运行器会反射将SuiteClasses值中的类封装成runners供系统调用，这里我们使用的是intellij，则由intellij的runner调用。这样就达到了套件测试。

其实也可以自定义规则，只需要继承ParentRunner或BlockJUnit4ClassRunner即可
例如Spring Framework中的SpringJUnit4ClassRunner ，Mockito中的MockitoJUnitRunner，Cucumber框架中的Cucumber Runner等等
这里我们看看cucumber自定义的runner.

```
public class Cucumber extends ParentRunner<FeatureRunner> {
}
```
使用时类似suite.

```
@RunWith(Cucumber.class)
@CucumberOptions(
        features= {"classpath:features"},
        glue= {"com.htest.core.steps","com.htest.hjservice.steps"},
        tags= {"@service"},
        plugin = {"json:build/result.json"}
)
public class StartTestCase {
}
```

这里的CucumberOptions注解是Cucumber运行器所需要的参数配置。

### 执行测试顺序 ###
在一个测试类中如果哟多个测试方法例如有testA(),testB(),testC()，那么junit4是以什么顺序执行呢？如果在testng中我们可以通过dependsOnMethods来管理测试方法的顺序。
而junit4默认是通过方法名的字符串比较升序排序的。

```
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class TestMethodOrder {
    @Test
    public void testA() {
        System.out.println("first");
    }
    @Test
    public void testB() {
        System.out.println("second");
    }
    @Test
    public void testC() {
        System.out.println("third");
    }
}
```
### 异常测试 ###
当我们想验证一段代码是否抛出期望的异常时，就会用到异常测试，异常测试的使用时通过expected指定异常类：

```
@Test(expected = IndexOutOfBoundsException.class) 
public void empty() { 
     new ArrayList<Object>().get(0); 
}
```
除此之外我们可以通过@Rule，设定异常规则，这里我们只需要知道@Rule会提供一个切入点，在测试方法前后或中执行被@Rule标记的类逻辑。后面会再次介绍。

```
 public static class ThrowExceptionWithExpectedType {
        @Rule
        public ExpectedException thrown = none();

        @Test
        public void throwsNullPointerException1() {
            thrown.expect(NullPointerException.class);
            throw new NullPointerException();
        }

        @Test
        public void throwsNullPointerException2() {
            thrown.expect(NullPointerException.class);
        }
    }
```
### 忽略测试 ###
 
忽略测试被用来禁止执行junit测试类的某些或者全部测试方法。使用时只需要将@Ignore添加在类或者方法上即可

```
@Ignore("For a good reason")
public class IgnoreMe {
        @Test
        public void iFail() {
            fail();
        }

        @Test
        public void iFailToo() {
            fail();
        }

        @Ignore
        @Test
        public void test() throws Exception {
            fail("test() should not run");
        }
    }
```

### 超时测试 ###
 junit4使用@Timeout注解来测试任意特定方法的执行时间。如果测试方法的执行时间大于指定的超时参数，测试方法将抛出异常，测试结果为失败，指定的超时参数单位为毫秒

```
@Test(timeout=1000)
public void testWithTimeout() {
  ...
}
```
timeout超时测试的实现是通过创建一个守护线程来执行该测试方法的。同样我们也可以使用@Rule来创建超时规则:

```
public class HasGlobalTimeout {
    public static String log;
    @Rule
    public Timeout globalTimeout = Timeout.seconds(10); // 10 seconds max per method tested

    @Test
    public void testSleepForTooLong() throws Exception {
        log += "ran1";
        TimeUnit.SECONDS.sleep(100); // sleep for 100 seconds
    }
}
```
Timeout.seconds(10)设定每个方法超时规则。

### 参数化测试 ###
junit支持参数化测试，但是使用起来比testng要复杂点，需要以下五步：
* 对测试类添加注解 @RunWith(Parameterized.class)
* 将需要使用参数定义为私有变量
* 使用上一步骤声明的私有变量作为入参，创建构造函数
* 创建一个使用@Parameters注解的公共静态方法，它将需要测试的各种变量值通过数组或集合的形式返回
* 使用定义的私有变量定义测试方法

例如我们使用一个函数来返回斐波那契数列中指定坐标的数：

```
public class Fibonacci {
    public static int compute(int n) {
    	int result = 0;
    	
        if (n <= 1) { 
        	result = n; 
        } else { 
        	result = compute(n - 1) + compute(n - 2); 
        }
        
        return result;
    }
}
```

斐波那契数列顺序应该是这样的：0,1,1,2,3,5,8。我们使用参数化的方式验证是否正确：

```
@RunWith(Parameterized.class)
public class FibonacciTest {
    @Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][] {     
                 { 0, 0 }, { 1, 1 }, { 2, 1 }, { 3, 2 }, { 4, 3 }, { 5, 5 }, { 6, 8 }  
           });
    }

    private int fInput;

    private int fExpected;

    public FibonacciTest(int input, int expected) {
        fInput= input;
        fExpected= expected;
    }

    @Test
    public void test() {
        assertEquals(fExpected, Fibonacci.compute(fInput));
    }
}
```
这里参数传递是通过构造函数的方式注入成成员变量，junit4还提供了@Parameter的方式来注入成员变量

```
@RunWith(Parameterized.class)
public class FibonacciTest {
    @Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][] {
                 { 0, 0 }, { 1, 1 }, { 2, 1 }, { 3, 2 }, { 4, 3 }, { 5, 5 }, { 6, 8 }  
           });
    }

    @Parameter // first data value (0) is default
    public /* NOT private */ int fInput;

    @Parameter(1)
    public /* NOT private */ int fExpected;

    @Test
    public void test() {
        assertEquals(fExpected, Fibonacci.compute(fInput));
    }
}
```
@Parameters有个name属性，它是通过二维数组参数来为一维数组生成一个名称，{index}代表当前参数的二维数组的index，{0}代表在二维数组中的第一个参数，{1}代表在二维数组中的第二个参数。例如下面这组参数：

```
 	@RunWith(Parameterized.class)
    public static class AdditionTestWithArray {
  		@Parameters(name = "{index}: {0} + {1} = {2}")
        public static Object[][] data() {
            return new Object[][] { { 0, 0, 0 }, { 1, 1, 2 }, { 3, 2, 5 },
                    { 4, 3, 7 } };
        }

	...
}
```
生成的测试case名称为：
> [0:0 + 0 = 0]
> [1:1 + 1 = 2]
> [2:3 + 2 = 5]
> [3:4 + 3 = 7]


同样，我们可以使用@Parameter(0)来选择注入二维数组中第一个参数

```
	@Parameter // first data value (0) is default
    public /* NOT private */ int fInput;
```

为了方便参数化，如果需要的参数只有一个，那么第四步可以不返回一个二维数组，可以是Iterable或一维数组

```
@Parameters
public static Iterable<? extends Object> data() {
    return Arrays.asList("first test", "second test");
}
或
@Parameters
public static Object[] data() {
    return new Object[] { "first test", "second test" };
}
```

### 规则 ###
junit4的@Rule，有点类似AOP的思想，以被测试类为目标（target） ，以被测方法为切入点（Pointcut），我们自定义的规则，来定义通知、增强处理（Advice）就是你想要的功能。这里说的比较晦涩，我们举个例子：在测试方法执行过程中想创建一些临时文件，当方法结束了就自动删除临时文件，那么我们就可以使用系统提供的TemporaryFolder规则。TemporaryFolder会在每个方法执行前创建一个临时文件夹，在执行方法过程中创建的文件都会由临时文件夹管理，当方法结束后会自动删除。

```
public static class HasTempFolder {
  @Rule
  public final TemporaryFolder folder = new TemporaryFolder();

  @Test
  public void testUsingTempFolder() throws IOException {
    File createdFile = folder.newFile("myfile.txt");
    File createdFolder = folder.newFolder("subfolder");
    // ...
  }
} 
```
除了TemporaryFolder规则外，junit4还提供了很多好用的规则：

|规则|用途|
|--|--|
|TemporaryFolder|管理临时文件|
|ExternalResource|管理额外的资源|
|ErrorCollector |错误收集|
|WatchmanTest|测试观察者，处理测试结果状态|
|TestName|获取测试方法名|
|Timeout|超时测试|
|ExpectedException |异常测试|

我们也可以自定义规则，只需要实现TestRule接口或MethodRule即可，如果同时实现两个接口则之后执行TestRule的apply，例如：

```
   public static class BothKindsOfRule implements TestRule, org.junit.rules.MethodRule {
        public int applications = 0;

        public Statement apply(Statement base, FrameworkMethod method,
                Object target) {
            applications++;
            return base;
        }

        public Statement apply(Statement base, Description description) {
            applications++;
            return base;
        }
    }

    public static class OneFieldTwoKindsOfRule {
        @Rule
        public BothKindsOfRule both = new BothKindsOfRule();

        @Test
        public void onlyOnce() {
            assertEquals(1, both.applications);
        }
    }
```

### 分类测试 ###
在junit4中没有像testng中分组的概念，只用分类测试，使用时分为以下步骤：
* 定义分类
* 将方法标记分类
* 指定@RunWith(Categories.class)运行器
* 使用IncludeCategory或ExcludeCategory选择要执行的分类

```
//定义两个分类
public interface FastTests { /* category marker */ }
public interface SlowTests { /* category marker */ }
//将方法分类
public class A {
  @Test
  public void a() {
    fail();
  }

  @Category(SlowTests.class)
  @Test
  public void b() {
  }
}
//将类中的方法都分成相同的多个类
@Category({SlowTests.class, FastTests.class})
public class B {
  @Test
  public void c() {

  }
}

//执行SlowTests分类的测试方法
@RunWith(Categories.class)
@IncludeCategory(SlowTests.class)
@ExcludeCategory(FastTests.class)
@SuiteClasses( { A.class, B.class }) // Note that Categories is a kind of Suite
public class SlowTestSuite {
  // Will run A.b, but not A.a or B.c
}
```

我这里是使用gradle构建的工程，gradle提供了对junit4的支持，在test任务中，我们可以指定执行那些分类.
```
test {
    useJUnit {
        includeCategories 'org.gradle.junit.CategoryA'
        excludeCategories 'org.gradle.junit.CategoryB'
    }
}
```