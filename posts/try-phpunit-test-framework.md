# 尝试 PHPUnit 测试框架

## Reference

- [Unit testing with PHPUnit](https://www.youtube.com/playlist?list=PLfdtiltiRHWGXSggf05W-pJbD47-_d8bJ)

------------------------------------------------------------------------------------------------------------------------

## 前情

在了解一些设计原则时，常常看到“测试驱动”这个名词，加之早些时候也知道在 PHP 开发中，常用的单元测试框架是`PHPUnit`，但因为没有合适的文字教程，而官网上的示例又不甚明白（惭愧），就一直迟迟没有花时间去真正了解。

在网上冲浪的过程中，发现YouTube上有位大佬做了[PHPUnit的教程视频](https://www.youtube.com/playlist?list=PLfdtiltiRHWGXSggf05W-pJbD47-_d8bJ)，故此做下这份笔记，以免遗忘。

## 开始

### 在项目中引入 PHPUnit

自然是使用 composer 来引入该库。

```bash
composer require --dev phpunit/phpunit
```

现在项目中的`composer.json`大概是这个样子：

```json
{
    "require": {
        "php": ">=7.1"
    },
    "require-dev": {
        "phpunit/phpunit": "^8.4"
    }
}
```

### 配置 PHPUnit

在引入 PHPUnit 后，可以发现`vendor`目录下多出许多文件夹。其中有一个`bin`目录，存放了 PHPUnit 的命令行启动文件。`phpunit`适用于Unix系统命令行执行，而`phpunit.bat`显然是在Windows下执行的。

这里我用的是 VS Code 的终端，配置了`Git Bash`，可以直接键入 Linux 命令执行。

进入项目根目录，尝试执行：

```bash
vendor/bin/phpunit
```

将输出一系列 PHPUnit 的选项与命令参考。

而 PHPUnit 的配置是根据**项目根目录**下的`phpunit.xml`文件来设置的。这里是跟着视频配置的一份简略的 XML 文件，不做太多的了解：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="vendor/autoload.php"
         colors="true"
         verbose="true"
         stopOnFailure="false">
    <testsuites>
        <testsuite name="Test suite">
            <directory>tests</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

对比了一下`vendor/phpunit/phpunit/phpunit.xml`文件，可以发现一点点不同，例如`bootstrap`属性。

大致可以从 XML 文件中看出，bootstrap 属性应该是一个入口文件（**有待商榷**），而`<testsuite name="Test suite">`（测试套件）下的`<directory>`标签则明显是说测试套件所在的目录。

其余不做了解，以后用到再说。

然后再次在终端中执行`vendor/bin/phpunit`，输出如下：

```
PHPUnit 8.4.3 by Sebastian Bergmann and contributors.

Runtime:       PHP 7.3.6
Configuration: E:\rep\project\phpunit.xml

Time: 70 ms, Memory: 4.00 MB

No tests executed!
```

从其大意可以看出，该命令是根据配置，来运行整个测试套件，并输出测试结果的。

### 开始一个简单的单元测试

现在我们进入`tests`目录，并在其下建立一个名为`unit`的文件夹：

```bash
# 进入 tests
cd tests
# 创建 unit 文件夹：存放单元测试文件
mkdir unit
```

并且在`unit`目录下创建文件`SampleTest.php`：

```bash
# 进入 unit
cd unit
# 创建空白文件
touch SampleTest.php
```

然后在该文件中键入以下代码：

```php
<?php

use PHPUnit\Framework\TestCase;

class SampleTest extends TestCase
{
    
}
```

代码中我们设计了一个继承`TestCase`的空类。然后运行测试命令：

```bash
# 返回根目录
cd ..
cd ..
# 执行 PHPUnit 测试
vendor/bin/phpunit
```

输出如下：

```
PHPUnit 8.4.3 by Sebastian Bergmann and contributors.

Runtime:       PHP 7.3.6
Configuration: E:\rep\project\phpunit.xml

W
 1 / 1 (100%)

Time: 849 ms, Memory: 4.00 MB

There was 1 warning:

1) Warning
No tests found in class "SampleTest".

WARNINGS!
Tests: 1, Assertions: 0, Warnings: 1.
```

可以看出，该命令成功测试了一个文件，并发出一个警告：在类`SampleTest`中没有发现任何测试（当然，现在`SampleTest`就是一个空类）。

然后我们尝试复制`SampleTest.php`文件，命名为`SampleTest2.php`，再次运行上面的命令：

输出与上面的输出一致，也许是因为类名一致或者是文件名不符合 PHPUnit 测试的规范（类名+ Test）被忽略了。

于是我又建了一个名为`Sample2Test.php`在同级目录下，并复制了`SampleTest.php`的代码。再次运行测试命令，输出：

```
Fatal error: Cannot declare class SampleTest, because the name is already in use in E:\rep\project\tests\unit\SampleTest.php on line 8
```

提示类名已存在，出现重复定义的致命错误。

于是修改`Sample2Test.php`中的类名为`SampleTest2`，执行测试命令，发现成功！

由此说明：

- PHPUnit 测试框架只测试规定文件夹内（在`phpunit.xml`中定义的目录）名称符合`类名+Test`规则的文件
- 文件名符合规范而类名不符合规范、但只要不定义已存在的类名时，是合法的


现在删除那些用于测试文件名的文件：

```bash
# rm + 路径 即删除文件（路径前加斜线表示绝对路径，不加表示相对路径
rm tests/unit/SampleTest2.php
rm tests/unit/Sample2Test.php
```

然后我们给`SampleTest.php`类文件添加测试：

```php
<?php

use PHPUnit\Framework\TestCase;

class SampleTest extends TestCase
{
    public function testTrueAssertsToTrue()
    {
        $this->assertTrue(true);
    }
}
```

我们给`SampleTest`类添加了一个方法：`testTrueAssertsToTrue()`，意为“条件为 true 时断言为 true”（这句话不是我翻译过来的）。我的英语实在是太烂了，没能看懂视频里说的是什么，于是去找了找`assertTrue()`方法的实现（继承自`TestCase`于是去该类找，没有找到，但发现该类继承自`Assert`，顺着继续找）。最后在`vendor/phpunit/phpunit/src/Framework/Assert/Functions.php`中找到具体实现：

```php
/**
 * Asserts that a condition is true.
 *
 * @throws ExpectationFailedException
 * @throws \SebastianBergmann\RecursionContext\InvalidArgumentException
 *
 * @psalm-assert true $condition
 *
 * @see Assert::assertTrue
 */
function assertTrue($condition, string $message = ''): void
{
    Assert::assertTrue(...\func_get_args());
}
```

在`vendor/phpunit/phpunit/src/Framework/Assert.php`中：

```php
    /**
     * Asserts that a condition is true.
     *
     * @throws ExpectationFailedException
     * @throws \SebastianBergmann\RecursionContext\InvalidArgumentException
     *
     * @psalm-assert true $condition
     */
    public static function assertTrue($condition, string $message = ''): void
    {
        static::assertThat($condition, static::isTrue(), $message);
    }
```

大致就是这样。

我们来测试更新后的代码：`vendor/bin/phpunit`，输出：

```
PHPUnit 8.4.3 by Sebastian Bergmann and contributors.

Runtime:       PHP 7.3.6
Configuration: E:\rep\project\phpunit.xml

.
 1 / 1 (100%)

Time: 130 ms, Memory: 4.00 MB

OK (1 test, 1 assertion)
```

说明成功一项测试断言成功！

好奇的我将“测试真假”这个方法都试了一遍，`SampleTest.php`中代码如下（测试结果在注释中）：

```php
<?php

use PHPUnit\Framework\TestCase;

class SampleTest extends TestCase
{
    // 判定 真 时：断言为 真
    // 结果：成功
    public function testTrueAssertsToTrue()
    {
        $this->assertTrue(true);
    }
    // 判定 真 时：断言为 假
    // 结果：失败
    public function testTrueAssertsToFalse()
    {
        $this->assertTrue(false);
    }
    // 判定 假 时：断言为 假
    // 结果：成功
    public function testFalseAssertsToFalse()
    {
        $this->assertFalse(false);
    }
    // 判定 假 时：断言为 真
    // 结果：失败
    public function testFalseAssertsToTrue()
    {
        $this->assertFalse(true);
    }
}
```

测试的运行结果：

```
PHPUnit 8.4.3 by Sebastian Bergmann and contributors.

Runtime:       PHP 7.3.6
Configuration: E:\rep\project\phpunit.xml

.F.F
 4 / 4 (100%)

Time: 101 ms, Memory: 4.00 MB

There were 2 failures:

1) SampleTest::testTrueAssertsToFalse
Failed asserting that false is true.

E:\rep\project\tests\unit\SampleTest.php:14

2) SampleTest::testFalseAssertsToTrue
Failed asserting that true is false.

E:\rep\project\tests\unit\SampleTest.php:24

FAILURES!
Tests: 4, Assertions: 4, Failures: 2.
```

这类应该就是**断言**测试：提供一个条件，判定其真假。

### 测试 User model

我们首先建立了一个名为`UserModelTest.php`的文件在`tests`文件夹，并测试`setName()`方法。代码如下：

```php
<?php

use PHPUnit\Framework\TestCase;

class UserModelTest extends TestCase
{
    public function testSetMyName()
    {
        $user = new \Linnzh\Models\User();

        $user->setName('一个被设置的名称');

        $this->assertEquals($user->getName(), '一个被设置的名称');// 比较两个参数是否一致
    }
}
```

此时我们的项目文件中并没有`User`模型类，运行`vendor/bin/phpunit`，看看会输出什么：

```bash
# ...
There was 1 error:

1) UserModelTest::testSetMyName
Error: Class 'Linnzh\Models\User' not found
# ...
```

很聪明，它提醒我们类`Linnzh\Models\User`并不存在。

既然如此，我们新建一个`Uesr`类，在`src/models`目录中，代码如下：

```php
<?php

namespace Linnzh\Models;

class User
{
    
}
```

很明显，我们在这个类里什么也没设计，也就是说我们测试的`setName()`方法并不存在。再次运行`vendor/bin/phpunit`看看：

```bash
# ...
There was 1 error:

1) UserModelTest::testSetMyName
Error: Call to undefined method Linnzh\Models\User::setName()
# ...
```

它告诉我们类中并没有定义`setName()`方法。真棒呀！提示清晰。

那我们就定义一个`setName()`方法：

```php
<?php

namespace Linnzh\Models;

class User
{
    public $name;
    
    public function setName($name)
    {
        $this->name = $name;
    }
}
```

再次运行测试命令：

```bash
# ...
There was 1 error:

1) UserModelTest::testSetMyName
Error: Call to undefined method Linnzh\Models\User::getName()
# ...
```

提示未定义`getName()`方法。既如此，那就定义一个：

```php
<?php

namespace Linnzh\Models;

class User
{
    public $name;

    public function setName($name)
    {
        $this->name = $name;
    }

    public function getName()
    {
        return $this->name;
    }
}
```

再次运行测试命令，可以发现测试通过。

经过上面的几次尝试，可以得知，PHPUnit 测试框架会严格检查测试方法中涉及的每一个函数或方法，并按照既定的规则进行比照，这比手工测试要精准的多。可以试一试将`setName()`方法定义为静态方法看看？

另外，PHPUnit 测试框架并不是智能的，它并不能测试你在方法中给出的数据以外的数据。因此，可以在一个测试方法中，多测试几个边界值。

**同时，也可以事先写好测试方法，在测试方法中写明期望结果。然后根据测试结果来检查代码问题。**

并且，类方法根据条件或使用场景的不同，可能有不同的使用方式，这就可以多写几个相关的测试方法（如果方法名不能明确表示该测试方法的用途，则需要写注释），尽可能地让测试覆盖所有的代码。

在测试`User`模型类时我们只用到一种比照方法`assertEquals()`，更多的笔者也没有接触。以后如果遇到了，再更新。

关于`User`模型测试就到这里。

### 使用注释标识测试方法

在之前的代码中我们可以看到，编写一个测试方法，必须在方法前添加`test`字样，这样可能看起来有点麻烦？

PHPUnit 还支持使用注释标识测试方法，例如：

```php
    /** @test 测试一段代码 */
    public function get_a_code()
    {
        $this->assertEquals('测试一段代码', '测试一段代码');
    }
```

这里的注释必须是块状注释（`/**  */`），并且中间的注释文本必须包含`@test`。

这仅仅是一种途径，使用哪种方式标识测试方法全凭喜好。

### 使用启动方法`setUp()`

在 PHPUnit 测试框架中，可以识别名为`setUp()`的方法。它将在调用每个测试方法前调用这个方法（而不同于`__construct()`构造方法只调用一次）：

```php
    public function setUp(): void
    {
        echo '我永远是一个宝宝!';
    }
```

> 如果提示声明必须兼容，可以查看`vendor/phpunit/phpunit/src/Fraamework/TestCase.php`中`setUp()`方法的定义。

因此，在测试某个类时，我们可以在`setUp()`方法中定义获取这个类对象，这样剩下的测试方法将不用每次都`new`同一个对象导致资源浪费。当然，这也需要一个类成员变量来保存它。就像这样：

```php
    protected $user;

    public function setUp(): void
    {
        $this->user = new User();
    }

    public function testSetMyName()
    {
        $this->user->setName('一个被设置的名称');

        $this->assertEquals($this->user->getName(), '一个被设置的名称');// 比较两个参数是否一致
    }
```

这节省了很多代码不是吗？

### 测试一个集合（`Collection`）类

视频中的第七节是讲测试一个集合（`Collection`）类。虽然英语渣的我现在也不知道为什么要有这么一个类。但，跟着做呗。

首先我们写一个测试集合类的文件`CollectionTest.php`，这也是**测试驱动开发**的一种形式吧：

```php
<?php

use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{
    /** @test */
    public function empty_instantiated_collection_returns_no_items()
    {
        $collection = new Collection();

        $this->assertEmpty($collection->get());
    }
}
```

这里我们写了一个测试一个被初始化后的`Collection`类的`items`是否为空。当然，肯定会提示我们找不到`Collection`类。那就新建；提示我们未定义`get()`方法，那就定义。于是`Collection.php`代码如下：

```php
<?php

namespace Linnzh\Support;

class Collection
{
    protected $items = [];

    public function get()
    {
        return $this->items;
    }
}
```

再进行测试，可以发现测试通过。

#### 检查 items 中的元素数量是否正确

既然`$items`是一个数组，那么我们就检查它的元素数量是否正确：

```php
    /** @test */
    public function count_is_correct_for_items_passed_in()
    {
        $collection = new Collection([
            'one', 'two', 'three'
        ]);

        $this->assertEquals(3, $collection->count());
    }
```

这里我们给`Collection`类传入了一个包含 3 个元素的数组，然后在`Collection`类中定义`count()`方法：

```php
    public function count()
    {
        return count($this->items);
    }
```

运行测试`vendor/bin/phpunit`，发现得到结果为 0 ，而不是 3。也就是说我们没能把这个数组传进去。嗯，我们没写构造函数😄补上补上：

```php
<?php

namespace Linnzh\Support;

class Collection
{
    protected $items = [];

    public function __construct(array $items = [])
    {
        $this->items = $items;
    }

    public function get()
    {
        return $this->items;
    }

    public function count()
    {
        return count($this->items);
    }
}
```

再次运行测试，可以发现所有的测试已通过。

#### 测试集合（`Collection`）类是否为可迭代对象（IteratorAggregate）

PHPUnit 测试框架有一个名为`assertInstanceof()`的方法，用来测试对象是否为指定类的实例。如下：

```php
    /** @test */
    public function collection_is_instance_of_iterator_aggregate()
    {
        $collection = new Collection();

        $this->assertInstanceOf(IteratorAggregate::class, $collection);
    }
```

运行结果：

```bash
There was 1 failure:

1) CollectionTest::collection_is_instance_of_iterator_aggregate
Failed asserting that Linnzh\Support\Collection Object (...) is an instance of interface "IteratorAggregate".
```

提示对象不是接口实例。现在我们让集合类继承这个接口：

```php
use IteratorAggregate;

class Collection implements IteratorAggregate
```

再次运行，提示说这个接口有个必须实现的抽象方法`IteratorAggregate::getIterator`。行吧，随便定义一下呗：

```php
    public function getIterator()
    {
        return [];
    }
```

再次运行，全部测试已通过。

但这样仅仅是测试了类型。该如何知道这个对象确实为可迭代呢？那就是循环它！

```php
    /** @test */
    public function collection_can_be_iterated()
    {
        $collection = new Collection([
            'one', 'two', 'three'
        ]);

        $items = [];

        foreach ($collection as  $item) {
            $items[] = $item;
        }

        $this->assertCount(3, $items);
    }
```

如果对象`$collection`可循环，则说明这个对象是可迭代的。运行`vendor/bin/phpunit`，不出意外地，测试失败了。

于是修改代码：

```php

use ArrayIterator;

// ...

    public function getIterator()
    {
        return new ArrayIterator($this->items);
    }
```

再次运行测试，全部通过！Congratulations！

#### 测试两个集合类是否可以合并

就像`array_merge()`函数一样，我们来检验两个不同的集合类对象是否可以合并。

```php
    /** @test */
    public function collection_can_be_merged_with_another_collection()
    {
        $collection1 = new Collection(['one', 'two']);
        $collection2 = new Collection(['three', 'four', 'five']);

        $newcollection = $collection1->merge($collection2);

        $this->assertCount(5, $newcollection->get());
        $this->assertEquals(5, $newcollection->count());
        $this->assertInstanceOf(Collection::class, $newcollection);
    }
```

```php
    public function merge(Collection $collection)
    {
        return new Collection(array_merge($this->get(), $collection->get()));
    }
```

这里我们借助了`array_merge()`函数，因为在前面我们已经实现集合类的可迭代化思路，也就是说我们可以把集合类当作最熟悉的数组处理，当然不能直接`merge`对象，而是`merge`可当作数组的`get()`方法返回的结果。并且由于处于一个类中，我们也不需要像`array_merge()`函数一样传入两个参数，传入被合并的对象即可。

现在我们运行`vendor/bin/phpunit`，可以看到所有测试都已通过。

有什么可以改进的呢？可以的。我们`merge()`方法返回的是一个新的集合类对象，也就是产生了一个副本，而不是直接修改原值。那我们想直接修改原值该怎么做呢？

```php
    public function merge(Collection $collection)
    {
        return $this->add($collection->get());
    }

    public function add(array $items)
    {
        $this->items = array_merge($this->items, $items);
    }
```

```php
    /** @test */
    public function collection_can_be_merged_with_another_collection()
    {
        $collection1 = new Collection(['one', 'two']);
        $collection2 = new Collection(['three', 'four', 'five']);

        $collection1->merge($collection2);

        $this->assertCount(5, $collection1->get());
        $this->assertEquals(5, $collection1->count());
        $this->assertInstanceOf(Collection::class, $collection1);
    }
```

直接使用`add()`方法修改对象成员`$items`即可！

运行测试，全部通过！

上面的测试用例是合并两个集合类对象，并添加了`add()`方法。现在我们测试`add()`方法：向已存在的集合类对象中添加元素

```php
    /** @test */
    public function can_add_to_existing_collection()
    {
        $collection = new Collection(['one', 'two']);

        $collection->add(['two']);

        $this->assertCount(3, $collection->get());
        $this->assertEquals(3, $collection->count());
    }
```

测试也全部通过。

另外我们还可以测试集合类对象是否可以转化为 JSON 字符串，或者直接将集合类转化为可被 JSON 化的实例。这里不再详细讲述。

### 测试一个简单的计算器（Calculator）

视频的第八到十节是讲解一个简单的计算器类（Calculator）的测试。

首先，我们来算加法（Addtion）。

写一个测试加法的文件`AddtionTest.php`：

```php
<?php

use Linnzh\Calculator\Addtion;
use PHPUnit\Framework\TestCase;

class AddtionTest extends TestCase
{
    /** @test */
    public function adds_up_given_operands()
    {
        $addtion = new Addtion();// 加法类
        $addtion->setOperands([10, 5]);// 设置操作数

        $this->assertEquals(15, $addtion->calculate());// 计算值
    }
}
```

同时开始编写加法类（Addtion）：

```php
<?php

namespace Linnzh\Calculator;

class Addtion
{

    protected $operands;

    public function setOperands(array $operands)
    {
        $this->operands = $operands;
    }


    public function calculate()
    {
        return array_sum($this->operands);
    }
}
```

运行测试，全部通过。

这个加法类的特点呢，在于把将操作数存入数组，然后进行一次计算（也就是相加）的操作。避免了多个操作数单次相加时重复操作。当然，这里也没有检验操作数是否全为数字，这里也不受影响，会自动将非数字的值转化为 0 。

#### 添加一个计算接口

因为计算器中的每个运算都会“计算”（calculate）一下，所以我们可以提取这部分作为接口。

```php
<?php

namespace Linnzh\Calculator;

interface OperationInterface
{
    public function calculate();
}
```

```php
class Addtion implements OperationInterface
```

再次运行测试，全部通过。

#### 检查并处理异常

加法操作并没有什么避讳，但在代码中，我们设定了操作数。那么如果没有操作数呢？如下：

```php
    /** @test */
    public function no_operands_given_throws_exception_when_calculating()
    {
        // $this->expectException(NoOperandsException::class);

        $addtion = new Addtion();
        $addtion->calculate();
    }
```

测试结果：

```bash
There was 1 error:

1) AddtionTest::no_operands_given_throws_exception_when_calculating
array_sum() expects parameter 1 to be array, null given
```

可以看到，这将抛出异常。

于是我们定义这么一个异常：

```php
<?php

namespace Linnzh\Calculator\Exception;

use Exception;

class NoOperandsException extends Exception
{
    
}
```

再次运行测试命令，输出：

```bash
There was 1 error:

1) AddtionTest::no_operands_given_throws_exception_when_calculating
array_sum() expects parameter 1 to be array, null given
```

还是抛出异常。这是因为我们还没有处理它。既然要处理异常，那么我们需要在适当的情况抛出这个异常：

```php
// code

protected $operands = [];

// code

    public function calculate()
    {
        if(count($this->operands) === 0) {
            throw new NoOperandsException;
        }
        
        return array_sum($this->operands);
    }
```

现在再次运行测试，可以发现测试都已通过。

#### 添加一个除法（Division）类

我们刚才做好了加法（Addtion）的运算，乘法（Multiplication）运算的过程和加法一致，代码仓库中已有相关代码，这里不再说明。

现在开始做一个除法（Division）。

像做加法一样，我们做测试驱动开发，先写测试文件`DivisionTest.php`：

```php
<?php

use Linnzh\Calculator\Division;
use PHPUnit\Framework\TestCase;

class DivisionTest extends TestCase
{
    /** @test */
    public function divides_given_operands()
    {
        $division = new Division();
        $division->setOperands([100, 2]);// 50

        $this->assertEquals(50, $division->calculate());
    }
}
```

运行测试命令`vendor/bin/phpunit`，可以看到我们还没有类`Division`。

然后我们创建它，就像这样：

```php
<?php

namespace Linnzh\Calculator;

class Division
{

}
```

再次测试，我们没有定义`setOperands()`和`calculte()`方法。

然后我们定义它，就像这样：

```php
    protected $operands = [];

    public function setOperands(array $operands)
    {
        $this->operands = $operands;
    }
```

`calculate()`方法我们是实现接口`OperationInterface`的：

```php
class Division implements OperationInterface

    public function calculate()
    {
        if(count($this->operands) === 0) {
            throw new NoOperandsException;
        }

        $result = 0;

        foreach ($this->operands as $key => $operand) {
            if($key === 0) {
                $result = $operand;
                continue;
            }
            $result = $result / $operand;
        }

        return $result;
    }
```

除法（和减法）运算要注意的是它不像加法和乘法是一个一个数字累计性的，可以将 0 或 1 当作初始值。它必须将第一个数作为初始值，也就有上面代码中的`if`段。

除了使用`foreach`循环迭代每个操作数，也可以使用`array_reduce()`函数来逐一迭代操作数，最后返回一个单一的值。代码如下：

```php
    public function calculate()
    {
        return array_reduce($this->operands, function($a, $b) {
            if($a !== null && $b !== null) {
                return $a / $b;
            }
            return $b;
        });
    }
```

当然，这是不够。我们还没考虑被除数为零的情况。现在我们加一个被除数为零的测试：

```php
    /** @test */
    public function divided_by_zero()
    {
        $division = new Division();
        $division->setOperands([100, 2, 0]);// Exception
        $this->assertEquals(50, $division->calculate());
    }
```

运行测试，输出：

```bash
There was 1 error:

1) DivisionTest::divided_by_zero
Division by zero
```

一种做法是提前移除操作数中的`0`，也就是过滤为`0`的操作数：

```php
        return array_reduce(array_filter($this->operands), function($a, $b) {
            if($a !== null && $b !== null) {
                return $a / $b;
            }
            return $b;
        });
```

这里使用的是`array_filter()`函数，它将过滤掉数组中等同于`false`的元素。

现在运行测试，可以发现测试都已通过。


#### 整合`setOperands()`方法

我们在四个运算中都使用了`setOperands()`方法，并且有一个成员变量为`operands`。为了减少代码量，也为了统一，我们可以将之集成到一个抽象类`OperationAbstract`中，然后在四则运算中继承它：

```php
<?php

namespace Linnzh\Calculator;

abstract class OperationAbstract
{
    protected $operands = [];

    public function setOperands(array $operands)
    {
        $this->operands = array_filter($operands, 'is_numeric');// 过滤非数字
    }
}
```

```php
class Addtion extends OperationAbstract implements OperationInterface
```

```php
class Subtraction extends OperationAbstract implements OperationInterface
```

```php
class Multiplication extends OperationAbstract implements OperationInterface
```

```php
class Division extends OperationAbstract implements OperationInterface
```

### 测试整个计算器系统

四则运算我们已经设置好了，但是一个完整的计算器不可能只使用一种运算方式。因此，我们需要一个完整的计算器类（Calculator）来整合这些运算。

同样是**测试驱动开发**，我们先写`CalculatorTest`类：

```php
<?php

use Linnzh\Calculator\Addtion;
use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    /** @test */
    public function can_set_single_operation()
    {
        $addtion = new Addtion();
        $addtion->setOperands([10, 5, 1]);

        $calculator = new Calculator();
        $calculator->setOperation($addtion);

        $this->assertCount(1, $calculator->getOperations());
    }
}
```

运行测试，很明显，我们缺少`Calculator`类。

```php
<?php

namespace Linnzh\Calculator;

class Calculator
{

}
```

根据测试文件，我们添加`setOperation()`方法。很明显，我们还需要一个成员变量`$operations`，并且传入的参数是实现了`OperationInterface`接口的实例，所以最后的代码：

```php
    protected $operations = [];

    public function setOperation(OperationInterface $operation)
    {
        $this->operations[] = $operation;
    }

    public function getOperations()
    {
        return $this->operations;
    }
```

再次运行测试代码，可以发现测试全部通过。

单一的运算测试我们通过了，现在测试多个运算：

```php
    /** @test */
    public function can_set_multiple_operations()
    {
        $addtion = new Addtion();
        $addtion->setOperands([10, 5, 1]);

        $sub = new Subtraction();
        $sub->setOperands([100, 200, -30]);

        $multi = new Multiplication();
        $multi->setOperands([10, 5]);

        $division = new Division();
        $division->setOperands([100, 2]);

        $calculator = new Calculator();
        $calculator->setOperations([$addtion, $sub, $multi, $division, 233]);

        $this->assertCount(5, $calculator->getOperations());
    }
```

这里我们使用了与`setOperation()`不同的方法，传入一个数组而不是单个参数，定义如下：

```php
    public function setOperations(array $operations)
    {
        $this->operations = array_merge(array_filter(
            $operations,
            function ($operation) {
                if (!$operation instanceof OperationInterface) {
                    return false;
                }
                return true;
            }
        ));
    }
```

上面的方法也可以简化为：

```php
    public function setOperations(array $operations)
    {
        $this->operations = array_merge(array_filter(
            $operations,
            function ($operation) {
                return $operation instanceof OperationInterface;
            }
        ));
    }
```

并且在这个方法，我们过滤了未实现`OperationInterface`接口的元素。现在运行测试，可以看到测试全部通过！当然，为了测试的单一原则，我们把这个要点分成两个测试：

```php
    /** @test */
    public function can_set_multiple_operations()
    {
        $addtion = new Addtion();
        $addtion->setOperands([10, 5, 1]);
        $sub = new Subtraction();
        $sub->setOperands([100, 200, -30]);
        $multi = new Multiplication();
        $multi->setOperands([10, 5]);

        $calculator = new Calculator();
        $calculator->setOperations([$addtion, $sub, $multi]);

        $this->assertCount(3, $calculator->getOperations());
    }

    /** @test */
    public function operations_are_ignored_if_not_instance_of_operation_interface()
    {
        $addtion = new Addtion();
        $addtion->setOperands([10, 5, 1]);
        $sub = new Subtraction();
        $sub->setOperands([100, 200, -30]);

        $calculator = new Calculator();
        $calculator->setOperations([$addtion, $sub, 2333]);

        $this->assertCount(2, $calculator->getOperations());
    }
```

测试也是可以通过的。

准备工作已完成，现在是重头戏：这个计算器到底能不能计算出正确的结果？为此我们写了一个测试：

```php
    /** @test */
    public function can_calculte_result()
    {
        $addtion = new Addtion();
        $addtion->setOperands([10, 5, 1]);// 16

        $calculator = new Calculator();
        $calculator->setOperation($addtion);

        $this->assertEquals(16, $calculator->calculate());
    }
```

那么应该怎样计算呢？我们有运算数组，每个运算数组分别有各自的操作数数组，也有自己的计算方法。

对于只有一个运算的计算来说，我们只需要调用它自己的计算方法。代码如下：

```php
    public function calculate()
    {
        return $this->operations[0]->calculate();
    }
```

运行测试可以发现测试通过了。那么对于多个运算来说呢？根据上面的案例，可以得知，我们只需要循环调用每个运算的计算方法即可，并且在计算时可以获取每则运算的结果，这个结果势必需要保存的。
根据期望，我们先写测试：

```php
    /** @test */
    public function calculate_method_returns_multiple_results()
    {
        $addtion = new Addtion();
        $addtion->setOperands([10, 5, 1]);// 16
        $division = new Division();
        $division->setOperands([10, 5]);// 2

        $calculator = new Calculator();
        $calculator->setOperations([$addtion, $division]);

        $this->assertIsArray($calculator->calculate());
        $this->assertEquals(16, $calculator->calculate()[0]);
        $this->assertEquals(2, $calculator->calculate()[1]);
    }
```

所以我们需要修改之前的`calculate()`方法，让它在传入多个运算时返回一个数组，数组元素为每则运算的运算结果：

```php
    public function calculate()
    {
        if (count($this->operations) > 1) {
            $results = [];
            foreach ($this->operations as $operation) {
                $results[] = $operation->calculate();
            }
            return $results;
        }
        return $this->operations[0]->calculate();
    }
```

当然，上面代码中的`foreach`循环也可以使用`array_map()`函数来代替：

```php
    public function calculate()
    {
        if (count($this->operations) > 1) {
            return array_map(function ($operation) {
                return $operation->calculate();
            }, $this->operations);
        }
        return $this->operations[0]->calculate();
    }
```

至此，简易计算器就完成了。关于 PHPUnit 测试框架的说明也就到这里。
