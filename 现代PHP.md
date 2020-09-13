# 命名空间

## 定义命名空间

任意合法的`PHP`代码都可以包含在命名空间中，但只有以下类型的代码受命名空间的影响，它们是：类（包括抽象类和traits）、接口、函数和常量。

命名空间通过关键字`namespace` 来声明。如果一个文件中包含命名空间，它必须在其它所有代码之前声明命名空间，除了一个以外：[declare](https://www.php.net/manual/zh/control-structures.declare.php)关键字。

**声明单个命名空间**

```php
<?php
namespace MyProject;

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }

?>
```

在声明命名空间之前唯一合法的代码是用于定义源文件编码方式的 `declare` 语句。另外，所有非` PHP` 代码包括空白符都不能出现在命名空间的声明之前：

```php
<html>
<?php
namespace MyProject; // 致命错误 -　命名空间必须是程序脚本的第一条语句
?>
```

> 注意:同一个命名空间可以定义在多个文件中，即允许将同一个命名空间的内容分割存放在不同的文件中。

## 定义子命名空间

与目录和文件的关系很象，`PHP` 命名空间也允许指定层次化的命名空间的名称。因此，命名空间的名字可以使用分层次的方式定义

**声明分层次的单个命名空间**

```php
<?php
namespace MyProject\Sub\Level;

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }

?>
```

上面的例子创建了常量`MyProject\Sub\Level\CONNECT_OK`，类 `MyProject\Sub\Level\Connection`和函数 `MyProject\Sub\Level\connect`

## 在同一个文件中定义多个命名空间(不建议这么做)

> 注意:违背了“一个文件一个类”的良好实践，因此不建议这么做

可以在同一个文件中定义多个命名空间。在同一个文件中定义多个命名空间有两种语法形式。

**定义多个命名空间，简单组合语法**

```php
<?php
namespace MyProject;

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }

namespace AnotherProject;

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }
?>
```

> 不建议使用这种语法在单个文件中定义多个命名空间。建议使用下面的大括号形式的语法

**定义多个命名空间，大括号语法**

```php
<?php
namespace MyProject {

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }
}

namespace AnotherProject {

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }
}
?>
```

> 在实际的编程实践中，非常不提倡在同一个文件中定义多个命名空间。这种方式的主要用于将多个 `PHP` 脚本合并在同一个文件中。

将全局的非命名空间中的代码与命名空间中的代码组合在一起，只能使用大括号形式的语法。全局代码必须用一个不带名称的 `namespace` 语句加上大括号括起来

**定义多个命名空间和不包含在命名空间中的代码**

```php
<?php
namespace MyProject {

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }
}

namespace { // global code
session_start();
$a = MyProject\connect();
echo MyProject\Connection::start();
}
?>
```

除了开始的declare语句外，命名空间的括号外不得有任何`PHP`代码

**定义多个命名空间和不包含在命名空间中的代码**

```php
<?php
declare(encoding='UTF-8');
namespace MyProject {

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }
}

namespace { // 全局代码
session_start();
$a = MyProject\connect();
echo MyProject\Connection::start();
}
?>
```

## 使用命名空间：基础

`PHP` 命名空间中的元素使用同样的原理。例如，类名可以通过三种方式引用：

1. 非限定名称，或不包含前缀的类名称，例如 `$a=new foo();` 或 `foo::staticmethod();`。如果当前命名空间是 `currentnamespace`，`foo` 将被解析为 `currentnamespace\foo`。如果使用 `foo` 的代码是全局的，不包含在任何命名空间中的代码，则` foo` 会被解析为`foo`。 警告：如果命名空间中的函数或常量未定义，则该非限定的函数名称或常量名称会被解析为全局函数名称或常量名称。详情参见 [使用命名空间：后备全局函数名称/常量名称](https://www.php.net/manual/zh/language.namespaces.fallback.php)。
2. 限定名称,或包含前缀的名称，例如 `$a = new subnamespace\foo();` 或 `subnamespace\foo::staticmethod();`。如果当前的命名空间是 `currentnamespace`，则 `foo` 会被解析为 `currentnamespace\subnamespace\foo`。如果使用 `foo` 的代码是全局的，不包含在任何命名空间中的代码，`foo` 会被解析为`subnamespace\foo`。
3. 完全限定名称，或包含了全局前缀操作符的名称，例如， `$a = new \currentnamespace\foo();` 或 `\currentnamespace\foo::staticmethod();`。在这种情况下，`foo` 总是被解析为代码中的文字名(literal name)`currentnamespace\foo`

file1.php

```php
<?php
namespace Foo\Bar\subnamespace;

const FOO = 1;
function foo() {}
class foo
{
    static function staticmethod() {}
}
?>
```

file2.php

```php
<?php
namespace Foo\Bar;
include 'file1.php';

const FOO = 2;
function foo() {}
class foo
{
    static function staticmethod() {}
}

/* 非限定名称 */
foo(); // 解析为 Foo\Bar\foo resolves to function Foo\Bar\foo
foo::staticmethod(); // 解析为类 Foo\Bar\foo的静态方法staticmethod。resolves to class Foo\Bar\foo, method staticmethod
echo FOO; // resolves to constant Foo\Bar\FOO

/* 限定名称 */
subnamespace\foo(); // 解析为函数 Foo\Bar\subnamespace\foo
subnamespace\foo::staticmethod(); // 解析为类 Foo\Bar\subnamespace\foo,
                                  // 以及类的方法 staticmethod
echo subnamespace\FOO; // 解析为常量 Foo\Bar\subnamespace\FOO
                                  
/* 完全限定名称 */
\Foo\Bar\foo(); // 解析为函数 Foo\Bar\foo
\Foo\Bar\foo::staticmethod(); // 解析为类 Foo\Bar\foo, 以及类的方法 staticmethod
echo \Foo\Bar\FOO; // 解析为常量 Foo\Bar\FOO
?>
```

> 注意访问任意全局类、函数或常量，都可以使用完全限定名称，例如 **`\strlen()`** 或 **`\Exception`** 或 `\INI_ALL`。

**在命名空间内部访问全局类、函数和常量**

```php
<?php
namespace Foo;

function strlen() {}
const INI_ALL = 3;
class Exception {}

$a = \strlen('hi'); // 调用全局函数strlen
$b = \INI_ALL; // 访问全局常量 INI_ALL
$c = new \Exception('error'); // 实例化全局类 Exception
?>
```

## 命名空间和动态语言特征

`PHP` 命名空间的实现受到其语言自身的动态特征的影响。因此，如果要将下面的代码转换到命名空间中

**态访问元素**

example1.php:

```php
<?php
class classname
{
    function __construct()
    {
        echo __METHOD__,"\n";
    }
}
function funcname()
{
    echo __FUNCTION__,"\n";
}
const constname = "global";

$a = 'classname';
$obj = new $a; // prints classname::__construct
$b = 'funcname';
$b(); // prints funcname
echo constant('constname'), "\n"; // prints global
?>
```

必须使用完全限定名称（包括命名空间前缀的类名称）。注意因为在动态的类名称、函数名称或常量名称中，限定名称和完全限定名称没有区别，因此其前导的反斜杠是不必要的。

**动态访问命名空间的元素**

```php
<?php
namespace namespacename;
class classname
{
    function __construct()
    {
        echo __METHOD__,"\n";
    }
}
function funcname()
{
    echo __FUNCTION__,"\n";
}
const constname = "namespaced";

include 'example1.php';

$a = 'classname';
$obj = new $a; // prints classname::__construct
$b = 'funcname';
$b(); // prints funcname
echo constant('constname'), "\n"; // prints global

/* note that if using double quotes, "\\namespacename\\classname" must be used */
$a = '\namespacename\classname';
$obj = new $a; // prints namespacename\classname::__construct
$a = 'namespacename\classname';
$obj = new $a; // also prints namespacename\classname::__construct
$b = 'namespacename\funcname';
$b(); // prints namespacename\funcname
$b = '\namespacename\funcname';
$b(); // also prints namespacename\funcname
echo constant('\namespacename\constname'), "\n"; // prints namespaced
echo constant('namespacename\constname'), "\n"; // also prints namespaced
?>
```

## `namespace`关键字和`__NAMESPACE__`常量

`PHP`支持两种抽象的访问当前命名空间内部元素的方法，**`__NAMESPACE__`** 魔术常量和`namespace`关键字。

常量**`__NAMESPACE__`**的值是包含当前命名空间名称的字符串。在全局的，不包括在任何命名空间中的代码，它包含一个空的字符串。

 **`__NAMESPACE__ `示例, 在命名空间中的代码**

```php
<?php
namespace MyProject;

echo '"', __NAMESPACE__, '"'; // 输出 "MyProject"
?>
```

**`__NAMESPACE__` 示例，全局代码**

```php
<?php

echo '"', __NAMESPACE__, '"'; // 输出 ""
?>
```

常量 **`__NAMESPACE__`** 在动态创建名称时很有用，例如：

**使用`__NAMESPACE__`动态创建名称**

```php
<?php
namespace MyProject;

function get($classname)
{
    $a = __NAMESPACE__ . '\\' . $classname;
    return new $a;
}
?>
```

关键字 `namespace` 可用来显式访问当前命名空间或子命名空间中的元素。它等价于类中的 `self` 操作符。

 **`namespace`操作符，命名空间中的代码**

```php
<?php
namespace MyProject;

use blah\blah as mine; // see "Using namespaces: importing/aliasing"

blah\mine(); // calls function MyProject\blah\mine()
namespace\blah\mine(); // calls function MyProject\blah\mine()

namespace\func(); // calls function MyProject\func()
namespace\sub\func(); // calls function MyProject\sub\func()
namespace\cname::method(); // calls static method "method" of class MyProject\cname
$a = new namespace\sub\cname(); // instantiates object of class MyProject\sub\cname
$b = namespace\CONSTANT; // assigns value of constant MyProject\CONSTANT to $b
?>
```

**`namespace`操作符, 全局代码**

```php
<?php

namespace\func(); // calls function func()
namespace\sub\func(); // calls function sub\func()
namespace\cname::method(); // calls static method "method" of class cname
$a = new namespace\sub\cname(); // instantiates object of class sub\cname
$b = namespace\CONSTANT; // assigns value of constant CONSTANT to $b
?>
```

## 使用命名空间：别名/导入

所有支持命名空间的`PHP`版本支持三种别名或导入方式：为类名称使用别名、为接口使用别名或为命名空间名称使用别名。`PHP 5.6`开始允许导入函数或常量或者为它们设置别名。

在`PHP`中，别名是通过操作符 `use` 来实现的. 下面是一个使用所有可能的五种导入方式的例子

**使用use操作符导入/使用别名**

```php
<?php
namespace foo;
use My\Full\Classname as Another;

// 下面的例子与 use My\Full\NSname as NSname 相同
use My\Full\NSname;

// 导入一个全局类
use ArrayObject;

// importing a function (PHP 5.6+)
use function My\Full\functionName;

// aliasing a function (PHP 5.6+)
use function My\Full\functionName as func;

// importing a constant (PHP 5.6+)
use const My\Full\CONSTANT;

$obj = new namespace\Another; // 实例化 foo\Another 对象
$obj = new Another; // 实例化 My\Full\Classname　对象
NSname\subns\func(); // 调用函数 My\Full\NSname\subns\func
$a = new ArrayObject(array(1)); // 实例化 ArrayObject 对象
// 如果不使用 "use \ArrayObject" ，则实例化一个 foo\ArrayObject 对象
func(); // calls function My\Full\functionName
echo CONSTANT; // echoes the value of My\Full\CONSTANT
?>
```

注意对命名空间中的名称（包含命名空间分隔符的完全限定名称如 `Foo\Bar`以及相对的不包含命名空间分隔符的全局名称如 `FooBar`）来说，前导的反斜杠是不必要的也不推荐的，因为导入的名称必须是完全限定的，不会根据当前的命名空间作相对解析。

**通过use操作符导入/使用别名，一行中包含多个use语句**

```php
<?php
use My\Full\Classname as Another, My\Full\NSname;

$obj = new Another; // 实例化 My\Full\Classname 对象
NSname\subns\func(); // 调用函数 My\Full\NSname\subns\func
?>
```

> 但是为了可读性，建议不要这么写，还是一行写一个use语句比较好：

```php
<?php
use Illuminate\Http\Request;
use Illuminate\Http\Response;
?>
```

导入操作是在编译执行的，但动态的类名称、函数名称或常量名称则不是。

**导入和动态名称**

```php
<?php
use My\Full\Classname as Another, My\Full\NSname;

$obj = new Another; // 实例化一个 My\Full\Classname 对象
$a = 'Another';
$obj = new $a;      // 实际化一个 Another 对象
?>
```

另外，导入操作只影响非限定名称和限定名称。完全限定名称由于是确定的，故不受导入的影响。

**导入和完全限定名称**

```php
<?php
use My\Full\Classname as Another, My\Full\NSname;

$obj = new Another; // instantiates object of class My\Full\Classname
$obj = new \Another; // instantiates object of class Another
$obj = new Another\thing; // instantiates object of class My\Full\Classname\thing
$obj = new \Another\thing; // instantiates object of class Another\thing
?>
```

## 全局空间

如果没有定义任何命名空间，所有的类与函数的定义都是在全局空间，与` PHP` 引入命名空间概念前一样。在名称前加上前缀 `\` 表示该名称是全局空间中的名称，即使该名称位于其它的命名空间中时也是如此。

**使用全局空间说明**

```php
<?php
namespace A\B\C;

/* 这个函数是 A\B\C\fopen */
function fopen() { 
     /* ... */
     $f = \fopen(...); // 调用全局的fopen函数
     return $f;
} 
?>
```

## 使用命名空间：后备全局函数/常量

在一个命名空间中，当 `PHP` 遇到一个非限定的类、函数或常量名称时，它使用不同的优先策略来解析该名称。类名称总是解析到当前命名空间中的名称。因此在访问系统内部或不包含在命名空间中的类名称时，必须使用完全限定名称，例如：

**在命名空间中访问全局类**

```php
<?php
namespace A\B\C;
class Exception extends \Exception {}

$a = new Exception('hi'); // $a 是类 A\B\C\Exception 的一个对象
$b = new \Exception('hi'); // $b 是类 Exception 的一个对象

$c = new ArrayObject; // 致命错误, 找不到 A\B\C\ArrayObject 类
?>
```

对于函数和常量来说，如果当前命名空间中不存在该函数或常量，`PHP` 会退而使用全局空间中的函数或常量

```php
<?php
namespace A\B\C;

const E_ERROR = 45;
function strlen($str)
{
    return \strlen($str) - 1;
}

echo E_ERROR, "\n"; // 输出 "45"
echo INI_ALL, "\n"; // 输出 "7" - 使用全局常量 INI_ALL

echo strlen('hi'), "\n"; // 输出 "1"
if (is_array('hi')) { // 输出 "is not array"
    echo "is array\n";
} else {
    echo "is not array\n";
}
?>
```

## 名称解析规则

在说明名称解析规则之前，我们先看一些重要的定义：

- 非限定名称Unqualified name

  名称中不包含命名空间分隔符的标识符，例如 `Foo`

- 限定名称Qualified name

  名称中含有命名空间分隔符的标识符，例如 `Foo\Bar`

- 完全限定名称Fully qualified name

  名称中包含命名空间分隔符，并以命名空间分隔符开始的标识符，例如 `\Foo\Bar`。 `namespace\Foo` 也是一个完全限定名称。

名称解析遵循下列规则：

1. 对完全限定名称的函数，类和常量的调用在编译时解析。例如 `new \A\B` 解析为类 `A\B`。

2. 所有的非限定名称和限定名称（非完全限定名称）根据当前的导入规则在编译时进行转换。例如，如果命名空间 `A\B\C` 被导入为 `C`，那么对 `C\D\e()` 的调用就会被转换为 `A\B\C\D\e()`。

3. 在命名空间内部，所有的没有根据导入规则转换的限定名称均会在其前面加上当前的命名空间名称。例如，在命名空间 `A\B` 内部调用 `C\D\e()`，则 `C\D\e()` 会被转换为 `A\B\C\D\e()` 。

4. 非限定类名根据当前的导入规则在编译时转换（用全名代替短的导入名称）。例如，如果命名空间 `A\B\C` 导入为C，则 `new C()` 被转换为 `new A\B\C() `。

5. 在命名空间内部（例如A\B），对非限定名称的函数调用是在运行时解析的。例如对函数 `foo()` 的调用是这样解析的：

   1. 在当前命名空间中查找名为 `A\B\foo()` 的函数
   2. 尝试查找并调用 *全局(global)* 空间中的函数 `foo()`。

6. 在命名空间（例如`A\B`）内部对非限定名称或限定名称类（非完全限定名称）的调用是在运行时解析的。下面是调用 `new C()` 及 `new D\E()` 的解析过程： `new C()`的解析:

   1. 在当前命名空间中查找`A\B\C`类。
   2. 尝试自动装载类`A\B\C`。

   `new D\E()`的解析:

   1. 在类名称前面加上当前命名空间名称变成：`A\B\D\E`，然后查找该类。
   2. 尝试自动装载类 `A\B\D\E`。

   为了引用全局命名空间中的全局类，必须使用完全限定名称 `new \C()`。

**名称解析示例**

```php
<?php
namespace A;
use B\D, C\E as F;

// 函数调用

foo();      // 首先尝试调用定义在命名空间"A"中的函数foo()
            // 再尝试调用全局函数 "foo"

\foo();     // 调用全局空间函数 "foo" 

my\foo();   // 调用定义在命名空间"A\my"中函数 "foo" 

F();        // 首先尝试调用定义在命名空间"A"中的函数 "F" 
            // 再尝试调用全局函数 "F"

// 类引用

new B();    // 创建命名空间 "A" 中定义的类 "B" 的一个对象
            // 如果未找到，则尝试自动装载类 "A\B"

new D();    // 使用导入规则，创建命名空间 "B" 中定义的类 "D" 的一个对象
            // 如果未找到，则尝试自动装载类 "B\D"

new F();    // 使用导入规则，创建命名空间 "C" 中定义的类 "E" 的一个对象
            // 如果未找到，则尝试自动装载类 "C\E"

new \B();   // 创建定义在全局空间中的类 "B" 的一个对象
            // 如果未发现，则尝试自动装载类 "B"

new \D();   // 创建定义在全局空间中的类 "D" 的一个对象
            // 如果未发现，则尝试自动装载类 "D"

new \F();   // 创建定义在全局空间中的类 "F" 的一个对象
            // 如果未发现，则尝试自动装载类 "F"

// 调用另一个命名空间中的静态方法或命名空间函数

B\foo();    // 调用命名空间 "A\B" 中函数 "foo"

B::foo();   // 调用命名空间 "A" 中定义的类 "B" 的 "foo" 方法
            // 如果未找到类 "A\B" ，则尝试自动装载类 "A\B"

D::foo();   // 使用导入规则，调用命名空间 "B" 中定义的类 "D" 的 "foo" 方法
            // 如果类 "B\D" 未找到，则尝试自动装载类 "B\D"

\B\foo();   // 调用命名空间 "B" 中的函数 "foo" 

\B::foo();  // 调用全局空间中的类 "B" 的 "foo" 方法
            // 如果类 "B" 未找到，则尝试自动装载类 "B"

// 当前命名空间中的静态方法或函数

A\B::foo();   // 调用命名空间 "A\A" 中定义的类 "B" 的 "foo" 方法
              // 如果类 "A\A\B" 未找到，则尝试自动装载类 "A\A\B"

\A\B::foo();  // 调用命名空间 "A\B" 中定义的类 "B" 的 "foo" 方法
              // 如果类 "A\B" 未找到，则尝试自动装载类 "A\B"
?>
```

# Trait

自 PHP 5.4.0 起，PHP 实现了一种代码复用的方法，称为 trait。

Trait 是为类似 PHP 的单继承语言而准备的一种代码复用机制。Trait 为了减少单继承语言的限制，使开发人员能够自由地在不同层次结构内独立的类中复用 method。Trait 和 Class 组合的语义定义了一种减少复杂性的方式，避免传统多继承和 Mixin 类相关典型问题。

Trait 和 Class 相似，但仅仅旨在用细粒度和一致的方式来组合功能。 无法通过 trait 自身来实例化。它为传统继承增加了水平特性的组合；也就是说，应用的几个 Class 之间不需要继承。

```php
<?php
trait ezcReflectionReturnInfo {
    function getReturnType() { /*1*/ }
    function getReturnDescription() { /*2*/ }
}

class ezcReflectionMethod extends ReflectionMethod {
    use ezcReflectionReturnInfo;
    /* ... */
}

class ezcReflectionFunction extends ReflectionFunction {
    use ezcReflectionReturnInfo;
    /* ... */
}
?>
```

## 优先级

从基类继承的成员被插入的 `SayWorld Trait` 中的` MyHelloWorld `方法所覆盖。其行为 `MyHelloWorld `类中定义的方法一致。优先顺序是当前类中的方法会覆盖 trait 方法，而 trait 方法又覆盖了基类中的方法。(**调用类>Trait>父类（如果有的话**)

```php
<?php
class Base {
    public function sayHello() {
        echo 'Hello ';
    }
}

trait SayWorld {
    public function sayHello() {
        parent::sayHello();
        echo 'World!';
    }
}

class MyHelloWorld extends Base {
    use SayWorld;
}

$o = new MyHelloWorld();
$o->sayHello();
?>
```

以上例程会输出

```
Hello World!
```

**另一个优先级顺序的例子**

```php
trait HelloWorld {
    public function sayHello() {
        echo 'Hello World!';
    }
}

class TheWorldIsNotEnough {
    use HelloWorld;
    public function sayHello() {
        echo 'Hello Universe!';
    }
}

$o = new TheWorldIsNotEnough();
$o->sayHello();
?>
```

以上例程会输出：

```
Hello Universe!
```

## 多个 trait

通过逗号分隔，在 use 声明列出多个 trait，可以都插入到一个类中

```php
<?php
trait Hello {
    public function sayHello() {
        echo 'Hello ';
    }
}

trait World {
    public function sayWorld() {
        echo 'World';
    }
}

class MyHelloWorld {
    use Hello, World;
    public function sayExclamationMark() {
        echo '!';
    }
}

$o = new MyHelloWorld();
$o->sayHello();
$o->sayWorld();
$o->sayExclamationMark();
?>
```

以上例程会输出：

```
Hello World!
```

## 冲突的解决

如果两个 `trait` 都插入了一个同名的方法，如果没有明确解决冲突将会产生一个致命错误。

为了解决多个 trait 在同一个类中的命名冲突，需要使用 `insteadof` 操作符来明确指定使用冲突方法中的哪一个。

以上方式仅允许排除掉其它方法，`as` 操作符可以 为某个方法引入别名。 注意，`as` 操作符不会对方法进行重命名，也不会影响其方法。

**Example #5 冲突的解决**

在本例中 Talker 使用了 trait A 和 B。由于 A 和 B 有冲突的方法，其定义了使用 trait B 中的 `smallTalk` 以及 trait A 中的 `bigTalk`。

Aliased_Talker 使用了 `as` 操作符来定义了 `talk` 来作为 B 的 `bigTalk` 的别名。

```php
<?php
trait A {
    public function smallTalk() {
        echo 'a';
    }
    public function bigTalk() {
        echo 'A';
    }
}

trait B {
    public function smallTalk() {
        echo 'b';
    }
    public function bigTalk() {
        echo 'B';
    }
}

class Talker {
    use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
    }
}

class Aliased_Talker {
    use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
        B::bigTalk as talk;
    }
}
?>
```

> 在 PHP 7.0 之前，在类里定义和 trait 同名的属性，哪怕是完全兼容的也会抛出 **`E_STRICT`**（完全兼容的意思：具有相同的访问可见性、初始默认值）

## 修改方法的访问控制

使用 `as` 语法还可以用来调整方法的访问控制

**修改方法的访问控制**

```php
<?php
trait HelloWorld {
    public function sayHello() {
        echo 'Hello World!';
    }
}

// 修改 sayHello 的访问控制
class MyClass1 {
    use HelloWorld { sayHello as protected; }
}

// 给方法一个改变了访问控制的别名
// 原版 sayHello 的访问控制则没有发生变化
class MyClass2 {
    use HelloWorld { sayHello as private myPrivateHello; }
}
?>
```

## 从 trait 来组成 trait

正如 class 能够使用 trait 一样，其它 trait 也能够使用 trait。在 trait 定义时通过使用一个或多个 trait，能够组合其它 trait 中的部分或全部成员

**从 trait 来组成 trait**

```php
<?php
trait Hello {
    public function sayHello() {
        echo 'Hello ';
    }
}

trait World {
    public function sayWorld() {
        echo 'World!';
    }
}

trait HelloWorld {
    use Hello, World;
}

class MyHelloWorld {
    use HelloWorld;
}

$o = new MyHelloWorld();
$o->sayHello();
$o->sayWorld();
?>
```

以上例程会输出：

```
Hello World!
```

## Trait 的抽象成员

为了对使用的类施加强制要求，trait 支持抽象方法的使用

> 一个可继承实体类，可以通过定义同名非抽象方法来满足要求；方法的签名可以不同

**表示通过抽象方法来进行强制要求**

```php
<?php
trait Hello {
    public function sayHelloWorld() {
        echo 'Hello'.$this->getWorld();
    }
    abstract public function getWorld();
}

class MyHelloWorld {
    private $world;
    use Hello;
    public function getWorld() {
        return $this->world;
    }
    public function setWorld($val) {
        $this->world = $val;
    }
}
?>
```

## Trait 的静态成员

Traits 可以被静态成员静态方法定义

**静态变量**

```php
<?php
trait Counter {
    public function inc() {
        static $c = 0;
        $c = $c + 1;
        echo "$c\n";
    }
}

class C1 {
    use Counter;
}

class C2 {
    use Counter;
}

$o = new C1(); $o->inc(); // echo 1
$p = new C2(); $p->inc(); // echo 1
?>
```

**静态方法**

```php
<?php
trait StaticExample {
    public static function doSomething() {
        return 'Doing something';
    }
}

class Example {
    use StaticExample;
}

Example::doSomething();
?>
```

## 属性

Trait 同样可以定义属性

**定义属性**

```php
<?php
trait PropertiesTrait {
    public $x = 1;
}

class PropertiesExample {
    use PropertiesTrait;
}

$example = new PropertiesExample;
$example->x;
?>
```

Trait 定义了一个属性后，类就不能定义同样名称的属性，否则会产生 fatal error。 有种情况例外：属性是兼容的（同样的访问可见度、初始默认值）。 在 PHP 7.0 之前，属性是兼容的，则会有 E_STRICT 的提醒。

**解决冲突**

```php
<?php
trait PropertiesTrait {
    public $same = true;
    public $different = false;
}

class PropertiesExample {
    use PropertiesTrait;
    public $same = true; // PHP 7.0.0 后没问题，之前版本是 E_STRICT 提醒
    public $different = true; // 致命错误
}
?>
```

