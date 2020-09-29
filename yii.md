Yii

# 应用结构

## 控制器

控制器是 [MVC](http://en.wikipedia.org/wiki/Model–view–controller) 模式中的一部分， 是继承[yii\base\Controller](https://www.yiichina.com/doc/api/2.0/yii-base-controller)类的对象，负责处理请求和生成响应。 具体来说，控制器从[应用主体](https://www.yiichina.com/doc/guide/2.0/structure-applications) 接管控制后会分析请求数据并传送到[模型](https://www.yiichina.com/doc/guide/2.0/structure-models)， 传送模型结果到[视图](https://www.yiichina.com/doc/guide/2.0/structure-views)，最后生成输出响应信息。

### 动作

控制器由 *操作* 组成，它是执行终端用户请求的最基础的单元， 一个控制器可有一个或多个操作。

如下示例显示包含两个动作`view` and `create` 的控制器`post`：

```php
namespace app\controllers;

use Yii;
use app\models\Post;
use yii\web\Controller;
use yii\web\NotFoundHttpException;

class PostController extends Controller
{
    public function actionView($id)
    {
        $model = Post::findOne($id);
        if ($model === null) {
            throw new NotFoundHttpException;
        }

        return $this->render('view', [
            'model' => $model,
        ]);
    }

    public function actionCreate()
    {
        $model = new Post;

        if ($model->load(Yii::$app->request->post()) && $model->save()) {
            return $this->redirect(['view', 'id' => $model->id]);
        } else {
            return $this->render('create', [
                'model' => $model,
            ]);
        }
    }
}
```

在操作 `view` (定义为 `actionView()` 方法)中， 代码首先根据请求模型ID加载 [模型](https://www.yiichina.com/doc/guide/2.0/structure-models)， 如果加载成功，会渲染名称为`view`的[视图](https://www.yiichina.com/doc/guide/2.0/structure-views)并显示，否则会抛出一个异常。

在操作 `create` (定义为 `actionCreate()` 方法)中, 代码相似. 先将请求数据填入[模型](https://www.yiichina.com/doc/guide/2.0/structure-models)， 然后保存模型，如果两者都成功，会跳转到ID为新创建的模型的`view`操作， 否则显示提供用户输入的`create`视图。

### 路由

终端用户通过所谓的*路由*寻找到动作，路由是包含以下部分的字符串：

- 模块ID: 仅存在于控制器属于非应用的[模块](https://www.yiichina.com/doc/guide/2.0/structure-modules);
- 控制器ID: 同应用（或同模块如果为模块下的控制器） 下唯一标识控制器的字符串;
- 操作ID: 同控制器下唯一标识操作的字符串。

路由使用如下格式:

```php
ControllerID/ActionID
```

如果属于模块下的控制器，使用如下格式：

```php
ModuleID/ControllerID/ActionID
```

如果用户的请求地址为 `http://hostname/index.php?r=site/index`, 会执行`site` 控制器的`index` 操作。 更多关于处理路由的详情请参阅 [路由](https://www.yiichina.com/doc/guide/2.0/runtime-routing) 一节。

### 创建控制器

在[Web applications](https://www.yiichina.com/doc/api/2.0/yii-web-application)网页应用中，控制器应继承[yii\web\Controller](https://www.yiichina.com/doc/api/2.0/yii-web-controller) 或它的子类。 同理在[console applications](https://www.yiichina.com/doc/api/2.0/yii-console-application)控制台应用中，控制器继承[yii\console\Controller](https://www.yiichina.com/doc/api/2.0/yii-console-controller) 或它的子类。 如下代码定义一个 `site` 控制器:

```php
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
}
```

#### 控制器ID

通常情况下，控制器用来处理请求有关的资源类型， 因此控制器ID通常为和资源有关的名词。 例如使用`article`作为处理文章的控制器ID。

控制器ID应仅包含英文小写字母、数字、下划线、中横杠和正斜杠， 例如 `article` 和 `post-comment` 是真是的控制器ID， `article?`, `PostComment`, `admin\post`不是控制器ID。

控制器Id可包含子目录前缀，例如 `admin/article` 代表 [controller namespace](https://www.yiichina.com/doc/api/2.0/yii-base-application#$controllerNamespace-detail) 控制器命名空间下 `admin`子目录中 `article` 控制器。 子目录前缀可为英文大小写字母、数字、下划线、正斜杠，其中正斜杠用来区分多级子目录(如 `panels/admin`)。

#### 控制器类命名

控制器ID遵循以下规则衍生控制器类名：

1. 将用正斜杠区分的每个单词第一个字母转为大写。注意如果控制器ID包含正斜杠， 只将最后的正斜杠后的部分第一个字母转为大写；
2. 去掉中横杠，将正斜杠替换为反斜杠;
3. 增加`Controller`后缀;
4. 在前面增加[controller namespace](https://www.yiichina.com/doc/api/2.0/yii-base-application#$controllerNamespace-detail)控制器命名空间.

下面为一些示例，假设[controller namespace](https://www.yiichina.com/doc/api/2.0/yii-base-application#$controllerNamespace-detail) 控制器命名空间为 `app\controllers`:

- `article` 对应 `app\controllers\ArticleController`;
- `post-comment` 对应 `app\controllers\PostCommentController`;
- `admin/post-comment` 对应 `app\controllers\admin\PostCommentController`;
- `adminPanels/post-comment` 对应 `app\controllers\adminPanels\PostCommentController`.

控制器类必须能被 [自动加载](https://www.yiichina.com/doc/guide/2.0/concept-autoloading)，所以在上面的例子中， 控制器`article` 类应在 [别名](https://www.yiichina.com/doc/guide/2.0/concept-aliases) 为`@app/controllers/ArticleController.php`的文件中定义， 控制器`admin/post-comment`应在`@app/controllers/admin/PostCommentController.php`文件中。

> **信息：** 最后一个示例 `admin/post-comment` 表示你可以将控制器放在 [controller namespace](https://www.yiichina.com/doc/api/2.0/yii-base-application#$controllerNamespace-detail)控制器命名空间下的子目录中， 在你不想用 [模块](https://www.yiichina.com/doc/guide/2.0/structure-modules) 的情况下给控制器分类，这种方式很有用。

#### 控制器部署

可通过配置 [controller map](https://www.yiichina.com/doc/api/2.0/yii-base-module#$controllerMap-detail) 来强制上述的控制器ID和类名对应， 通常用在使用第三方不能掌控类名的控制器上。

配置 [应用配置](https://www.yiichina.com/doc/guide/2.0/structure-applications#application-configurations) 中的[application configuration](https://www.yiichina.com/doc/guide/2.0/structure-applications#application-configurations)，如下所示：

```php
[
    'controllerMap' => [
        // 用类名申明 "account" 控制器
        'account' => 'app\controllers\UserController',

        // 用配置数组申明 "article" 控制器
        'article' => [
            'class' => 'app\controllers\PostController',
            'enableCsrfValidation' => false,
        ],
    ],
]
```

#### 默认控制器

每个应用有一个由[yii\base\Application::$defaultRoute](https://www.yiichina.com/doc/api/2.0/yii-base-module#$defaultRoute-detail)属性指定的默认控制器； 当请求没有指定 [路由](https://www.yiichina.com/doc/guide/2.0/structure-controllers#ids-routes)，该属性值作为路由使用。 对于[Web applications](https://www.yiichina.com/doc/api/2.0/yii-web-application)网页应用，它的值为 `'site'`，对于 [console applications](https://www.yiichina.com/doc/api/2.0/yii-console-application) 控制台应用，它的值为 `help`，所以URL为 `http://hostname/index.php` 表示由 `site` 控制器来处理。

可以在 [应用配置](https://www.yiichina.com/doc/guide/2.0/structure-applications#application-configurations) 中修改默认控制器，如下所示：

```php
[
    'defaultRoute' => 'main',
]
```

### 创建动作

创建操作可简单地在控制器类中定义所谓的 *操作方法* 来完成，操作方法必须是以`action`开头的公有方法。 操作方法的返回值会作为响应数据发送给终端用户， 如下代码定义了两个操作 `index` 和 `hello-world`:

```php
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public function actionIndex()
    {
        return $this->render('index');
    }

    public function actionHelloWorld()
    {
        return 'Hello World';
    }
}
```

#### 动作ID

操作通常是用来执行资源的特定操作，因此， 操作ID通常为动词，如`view`, `update`等。

操作ID应仅包含英文小写字母、数字、下划线和中横杠，操作ID中的中横杠用来分隔单词。 例如`view`, `update2`, `comment-post`是真实的操作ID， `view?`, `Update`不是操作ID.

可通过两种方式创建操作ID，内联操作和独立操作. An inline action is 内联操作在控制器类中定义为方法；独立操作是继承[yii\base\Action](https://www.yiichina.com/doc/api/2.0/yii-base-action)或它的子类的类。 内联操作容易创建，在无需重用的情况下优先使用； 独立操作相反，主要用于多个控制器重用， 或重构为[扩展](https://www.yiichina.com/doc/guide/2.0/structure-extensions)。

#### 内联动作

内联动作指的是根据我们刚描述的操作方法。

动作方法的名字是根据操作ID遵循如下规则衍生：

1. 将每个单词的第一个字母转为大写;
2. 去掉中横杠;
3. 增加`action`前缀.

例如`index` 转成 `actionIndex`, `hello-world` 转成 `actionHelloWorld`。

> **注意：** 操作方法的名字*大小写敏感*，如果方法名称为`ActionIndex`不会认为是操作方法， 所以请求`index`操作会返回一个异常， 也要注意操作方法必须是公有的， 私有或者受保护的方法不能定义成内联操作。

因为容易创建，内联操作是最常用的操作， 但是如果你计划在不同地方重用相同的操作， 或者你想重新分配一个操作，需要考虑定义它为*独立操作*。

#### 独立动作

独立操作通过继承[yii\base\Action](https://www.yiichina.com/doc/api/2.0/yii-base-action)或它的子类来定义。 例如Yii发布的[yii\web\ViewAction](https://www.yiichina.com/doc/api/2.0/yii-web-viewaction) 和[yii\web\ErrorAction](https://www.yiichina.com/doc/api/2.0/yii-web-erroraction)都是独立操作。

要使用独立操作，需要通过控制器中覆盖[yii\base\Controller::actions()](https://www.yiichina.com/doc/api/2.0/yii-base-controller#actions()-detail)方法在*action map*中申明， 如下例所示：

```php
public function actions()
{
    return [
        // 用类来申明"error" 动作
        'error' => 'yii\web\ErrorAction',

        // 用配置数组申明 "view" 动作
        'view' => [
            'class' => 'yii\web\ViewAction',
            'viewPrefix' => '',
        ],
    ];
}
```

如上所示， `actions()` 方法返回键为操作ID、值为对应操作类名 或数组[configurations](https://www.yiichina.com/doc/guide/2.0/concept-configurations) 的数组。 和内联操作不同，独立操作ID可包含任意字符，只要在`actions()` 方法中申明.

为创建一个独立操作类，需要继承[yii\base\Action](https://www.yiichina.com/doc/api/2.0/yii-base-action) 或它的子类，并实现公有的名称为`run()`的方法, `run()` 方法的角色和操作方法类似，例如：

```php
<?php
namespace app\components;

use yii\base\Action;

class HelloWorldAction extends Action
{
    public function run()
    {
        return "Hello World";
    }
}
```

事例：在\app\controllers\BookController里定义

```php
public function actions()
{
    return [
        'test' => 'app\controllers\TestAction'
    ]; // TODO: Change the autogenerated stub
}
```

```php
<?php

namespace app\controllers;

class TestAction extends \yii\base\Action
{

    public function run()
    {
        return "hello world!";
    }

}
```

访问:http://www.yii.com/book/test 显示hello world!

#### 动作结果

操作方法或独立操作的`run()`方法的返回值非常重要， 它表示对应操作结果。

返回值可为 [响应](https://www.yiichina.com/doc/guide/2.0/runtime-responses) 对象，作为响应发送给终端用户。

- 对于[Web applications](https://www.yiichina.com/doc/api/2.0/yii-web-application)网页应用，返回值可为任意数据, 它赋值给[yii\web\Response::$data](https://www.yiichina.com/doc/api/2.0/yii-web-response#$data-detail)， 最终转换为字符串来展示响应内容。
- 对于[console applications](https://www.yiichina.com/doc/api/2.0/yii-console-application)控制台应用，返回值可为整数， 表示命令行下执行的 [exit status](https://www.yiichina.com/doc/api/2.0/yii-base-response#$exitStatus-detail) 退出状态。

在上面的例子中，操作结果都为字符串，作为响应数据发送给终端用户， 下例显示一个操作通过 返回响应对象（因为[redirect()](https://www.yiichina.com/doc/api/2.0/yii-web-controller#redirect()-detail)方法返回一个响应对象） 可将用户浏览器跳转到新的URL。

```php
public function actionForward()
{
    // 用户浏览器跳转到 http://example.com
    return $this->redirect('http://example.com');
}
```

#### 动作参数

内联动作的操作方法和独立动作的 `run()` 方法可以带参数，称为*动作参数*。 参数值从请求中获取，对于[Web applications](https://www.yiichina.com/doc/api/2.0/yii-web-application)网页应用， 每个动作参数的值从`$_GET`中获得，参数名作为键； 对于[console applications](https://www.yiichina.com/doc/api/2.0/yii-console-application)控制台应用, 动作参数对应命令行参数。

如下例，动作`view` (内联动作) 申明了两个参数 `$id` 和 `$version`。

```php
namespace app\controllers;

use yii\web\Controller;

class PostController extends Controller
{
    public function actionView($id, $version = null)
    {
        // ...
    }
}
```

动作参数会被不同的参数填入，如下所示：

- `http://hostname/index.php?r=post/view&id=123`: `$id` 会填入`'123'`， `$version` 仍为 null 空因为没有`version`请求参数;
- `http://hostname/index.php?r=post/view&id=123&version=2`: $id` 和 `$version` 分别填入 `'123'` 和 `'2'`；
- `http://hostname/index.php?r=post/view`: 会抛出[yii\web\BadRequestHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-badrequesthttpexception) 异常 因为请求没有提供参数给必须赋值参数`$id`；
- `http://hostname/index.php?r=post/view&id[]=123`: 会抛出[yii\web\BadRequestHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-badrequesthttpexception) 异常 因为 `$id` 参数收到数组值 `['123']` 而不是字符串.

如果你想要一个动作参数来接受数组值，你应该使用 `array` 来提示它，如下所示：

```php
public function actionView(array $id, $version = null)
{
    // ...
}
```

现在如果请求为 `http://hostname/index.php?r=post/view&id[]=123`, 参数 `$id` 会使用数组值 `['123']`， 如果请求为 `http://hostname/index.php?r=post/view&id=123`， 参数 `$id` 会获取相同数组值，因为无类型的 `'123'` 会自动转成数组。

上述例子主要描述网页应用的操作参数，对于控制台应用， 更多详情请参阅[控制台命令](https://www.yiichina.com/doc/guide/2.0/tutorial-console)。

#### 默认动作

每个控制器都有一个由 [yii\base\Controller::$defaultAction](https://www.yiichina.com/doc/api/2.0/yii-base-controller#$defaultAction-detail) 属性指定的默认操作， 当[路由](https://www.yiichina.com/doc/guide/2.0/structure-controllers#ids-routes) 只包含控制器ID， 会使用所请求的控制器的默认操作。

默认操作默认为 `index`，如果想修改默认操作，只需简单地在控制器类中覆盖这个属性， 如下所示：

```php
namespace app\controllers;

use yii\web\Controller;

class SiteController extends Controller
{
    public $defaultAction = 'home';

    public function actionHome()
    {
        return $this->render('home');
    }
}
```

### 控制器生命周期

处理一个请求时，[应用主体](https://www.yiichina.com/doc/guide/2.0/structure-applications) 会根据请求 [路由](https://www.yiichina.com/doc/guide/2.0/structure-controllers#routes)创建一个控制器， 控制器经过以下生命周期来完成请求：

1. 在控制器创建和配置后，[yii\base\Controller::init()](https://www.yiichina.com/doc/api/2.0/yii-base-baseobject#init()-detail) 方法会被调用。
2. 控制器根据请求操作ID创建一个操作对象:
   - 如果操作ID没有指定，会使用[default action ID](https://www.yiichina.com/doc/api/2.0/yii-base-controller#$defaultAction-detail)默认操作ID；
   - 如果在[action map](https://www.yiichina.com/doc/api/2.0/yii-base-controller#actions()-detail)找到操作ID， 会创建一个独立操作；
   - 如果操作ID对应操作方法，会创建一个内联操作；
   - 否则会抛出[yii\base\InvalidRouteException](https://www.yiichina.com/doc/api/2.0/yii-base-invalidrouteexception)异常。
3. 控制器按顺序调用应用主体、模块（如果控制器属于模块）、 控制器的beforeAction()方法；
   - 如果任意一个调用返回false，后面未调用的`beforeAction()`会跳过并且操作执行会被取消； action execution will be cancelled.
   - 默认情况下每个 `beforeAction()` 方法会触发一个 `beforeAction` 事件，在事件中你可以追加事件处理操作；
4. 控制器执行操作:
   - 请求数据解析和填入到操作参数；
5. 控制器按顺序调用控制器、模块（如果控制器属于模块）、应用主体的afterAction()方法；
   - 默认情况下每个 `afterAction()` 方法会触发一个 `afterAction` 事件， 在事件中你可以追加事件处理操作；
6. 应用主体获取操作结果并赋值给[响应](https://www.yiichina.com/doc/guide/2.0/runtime-responses).

## 模型

模型是 [MVC](http://en.wikipedia.org/wiki/Model–view–controller) 模式中的一部分， 是代表业务数据、规则和逻辑的对象。

可通过继承 [yii\base\Model](https://www.yiichina.com/doc/api/2.0/yii-base-model) 或它的子类定义模型类， 基类[yii\base\Model](https://www.yiichina.com/doc/api/2.0/yii-base-model)支持许多实用的特性：

- [属性](https://www.yiichina.com/doc/guide/2.0/structure-models#attributes): 代表可像普通类属性或数组 一样被访问的业务数据;
- [属性标签](https://www.yiichina.com/doc/guide/2.0/structure-models#attribute-labels): 指定属性显示出来的标签;
- [块赋值](https://www.yiichina.com/doc/guide/2.0/structure-models#massive-assignment): 支持一步给许多属性赋值;
- [验证规则](https://www.yiichina.com/doc/guide/2.0/structure-models#validation-rules): 确保输入数据符合所申明的验证规则;
- [数据导出](https://www.yiichina.com/doc/guide/2.0/structure-models#data-exporting): 允许模型数据导出为自定义格式的数组。

`Model` 类也是更多高级模型如[Active Record 活动记录](https://www.yiichina.com/doc/guide/2.0/db-active-record)的基类， 更多关于这些高级模型的详情请参考相关手册。

> **信息：** 模型并不强制一定要继承[yii\base\Model](https://www.yiichina.com/doc/api/2.0/yii-base-model)，但是由于很多组件支持[yii\base\Model](https://www.yiichina.com/doc/api/2.0/yii-base-model)， 最好使用它做为模型基类。

### 属性

模型通过 *属性* 来代表业务数据，每个属性像是模型的公有可访问属性， [yii\base\Model::attributes()](https://www.yiichina.com/doc/api/2.0/yii-base-model#attributes()-detail) 指定模型所拥有的属性。

可像访问一个对象属性一样访问模型的属性:

```php
$model = new \app\models\ContactForm;

// "name" 是ContactForm模型的属性
$model->name = 'example';
echo $model->name;
```

也可像访问数组单元项一样访问属性，这要感谢[yii\base\Model](https://www.yiichina.com/doc/api/2.0/yii-base-model)支持 [ArrayAccess 数组访问](http://php.net/manual/en/class.arrayaccess.php) 和 [ArrayIterator 数组迭代器](http://php.net/manual/en/class.arrayiterator.php):

```php
$model = new \app\models\ContactForm;

// 像访问数组单元项一样访问属性
$model['name'] = 'example';
echo $model['name'];

// 迭代器遍历模型
foreach ($model as $name => $value) {
    echo "$name: $value\n";
}
```



# 路由

## 创建Urls

```php
use yii\helpers\Url;

// 创建一个普通的路由URL：/index.php?r=post%2Findex
echo Url::to(['post/index']);

// 创建一个带路由参数的URL：/index.php?r=post%2Fview&id=100
echo Url::to(['post/view', 'id' => 100]);

// 创建一个带锚定的URL：/index.php?r=post%2Fview&id=100#content
echo Url::to(['post/view', 'id' => 100, '#' => 'content']);

// 创建一个绝对路径URL：http://www.example.com/index.php?r=post%2Findex
echo Url::to(['post/index'], true);

// 创建一个带https协议的绝对路径URL：https://www.example.com/index.php?r=post%2Findex
echo Url::to(['post/index'], 'https');
```



- 如果路由是一个空字符串，则使用当前请求的[路由](https://www.yiichina.com/doc/api/2.0/yii-base-controller#$route-detail)；
- 如果路由中不存在`/`，则被认为是一个当前控制器下的动作ID， 且路由被附加到当前控制器的[唯一ID](https://www.yiichina.com/doc/api/2.0/yii-base-controller#$uniqueId-detail)后面；
- 如果路由不以`/`开头，被认为是当前模块下的路由， 路由将被附加到当前模块的[唯一ID](https://www.yiichina.com/doc/api/2.0/yii-base-module#$uniqueId-detail)后面

```php
use yii\helpers\Url;

// 当前请求路由：/index.php?r=admin%2Fpost%2Findex
echo Url::to(['']);

// 只有动作ID的相对路由：/index.php?r=admin%2Fpost%2Findex
echo Url::to(['index']);

// 相对路由：/index.php?r=admin%2Fpost%2Findex
echo Url::to(['post/index']);

// 绝对路由：/index.php?r=post%2Findex
echo Url::to(['/post/index']);

// 假设有一个"/post/index"的别名"@posts"：/index.php?r=post%2Findex
Yii::setAlias('@posts','post/index');
echo Url::to(['@posts']).'<br>';
```





```php
use yii\helpers\Url;

// 当前请求URL：/index.php?r=admin%2Fpost%2Findex
echo Url::to();

// 设定了别名的URL：http://example.com
Yii::setAlias('@example', 'http://example.com/');
echo Url::to('@example');

// 绝对URL：http://example.com/images/logo.gif
echo Url::to('/images/logo.gif', true);
```





```php
// 主页URL：/index.php?r=site%2Findex
echo Url::home().'<br>';

// 根URL，如果程序部署到一个Web目录下的子目录时非常有用
echo Url::base().'<br>';

// 当前请求的权威规范URL
// 参考 https://en.wikipedia.org/wiki/Canonical_link_element
echo Url::canonical().'<br>';

// 记住当前请求的URL并在以后获取
Url::remember();
echo Url::previous().'<br>';
```

## 使用美化的 URL

```php
// 
[
    'components' => [
        'urlManager' => [
            'enablePrettyUrl' => true,	 //开启美化URL
            'showScriptName' => false,	//是否显示脚本名称
            'enableStrictParsing' => false,	//是否开启严格解析
            'rules' => [		//规则
                // ...
            ],
        ],
    ],
]
```



规则

```php
[                  //规则
    'url3' => 'url/url3',	//http://www.yii.com/url3
    'url4/<id:\d+>' => 'url/url4',	//http://www.yii.com/url4/100
],
```

也可以用数组来表示

```php
//http://www.yii.com/url5.json
[
    'pattern' => 'url5',
    'route' => 'url/url5',
    'suffix' => '.json',
],
```

### 参数化路由

```php
'rules' => [
    '<controller:(url|post)>/create' => '<controller>/create',
    '<controller:(post|url)>/<id:\d+>/<action:(update|delete)>' => '<controller>/<action>',
    '<controller:(post|url)>/<id:\d+>' => '<controller>/view',
    '<controller:(post|url)>s' => '<controller>/index',
],
```

### 默认参数

```php
 //默认参数值
 [
     'pattern' => 'url/<page:\d+>/<tag>',
     'route' => url/default-route-url',
     'defaults' => ['page' => 1, 'tag' => ''],
 ],
```

- `/index.php/url为 1，`tag` 为 ''。
- `/index.php/url/2`：`page` 为 2，`tag` 为 ''。
- `/index.php/url/2/news`：`page` 为 2，`tag`为 `'news'`。
- `/index.php/url/news`：`page` 为 1，`tag` 为 `'news'`。

### 开启后缀

```php
return [
        'enablePrettyUrl' => true,      //开启美化URL
        'showScriptName' => false,      //是否显示脚本名称
        'enableStrictParsing' => false, //是否开启严格解析
        'suffix' => '.html',            //开启后缀
    .....
]
```

- 当你配置 URL 后缀时，如果请求的 URL 没有此后缀，系统将认为此 URL 无法识别。 这是 SEO（搜索引擎优化）的最佳实践。
- 你可以设置URL后缀为 `/` 让所有的 URL 以斜线结束

- URL规则中此属性将覆盖在[URL管理器](https://www.yiichina.com/doc/api/2.0/yii-web-urlmanager)中设置的值

### Http方法

```php
'rules' => [
    'PUT,POST post/<id:\d+>' => 'post/update',
    'DELETE post/<id:\d+>' => 'post/delete',
    'post/<id:\d+>' => 'post/view',
]
```

### 动态创建路由

```php
<?php


namespace app\components;


use yii\web\Request;
use yii\web\UrlManager;

class CarUrlRule extends \yii\base\BaseObject implements \yii\web\UrlRuleInterface
{

    /**
     * @inheritDoc
     */
    public function parseRequest($manager, $request)
    {
        $pathInfo = $request->getPathInfo();
        if (preg_match('%^(\w+)(/(\w+))?$%', $pathInfo, $matches)) {
            // 检查 $matches[1] 和 $matches[3]
            // 确认是否匹配到一个数据库中保存的厂家和型号。
            // 如果匹配，设置参数 $params['manufacturer'] 和 / 或 $params['model']
            $params['manufacturer'] = '111';
            $params['model'] = '222';
            // 返回 ['car/index', $params]
            return ['car/hodas',$params];
        }
        return false; // 本规则不会起作用
    }

    /**
     * 当上面返回的地址不存时，会以默认url"site/index进入以下方法";
     */
    public function createUrl($manager, $route, $params)
    {
        if ($route === 'car/index') {
            if (isset($params['manufacturer'], $params['model'])) {
                return $params['manufacturer'] . '/' . $params['model'];
            } elseif (isset($params['manufacturer'])) {
                return $params['manufacturer'];
            }
        }
        return false; // this rule does not apply
    }
}
```

```php
'rules' => [
    // ...其它规则...
    [
        'class' => 'app\components\CarUrlRule',
        // ...配置其它参数...
    ],
]
```

### URL规范化



从 2.0.10 版开始[Url管理器](https://www.yiichina.com/doc/api/2.0/yii-web-urlmanager)可以配置用[URL规范器](https://www.yiichina.com/doc/api/2.0/yii-web-urlnormalizer)来处理 相同URL的不同格式，例如是否带结束斜线。因为技术上来说 `http://example.com/path` 和 `http://example.com/path/` 是完全不同的 URL，两个地址返回相同的内容会导致SEO排名降低。 默认情况下 URL 规范器会合并连续的斜线，根据配置决定是否添加或删除结尾斜线， 并且会使用[永久重定向](https://en.wikipedia.org/wiki/HTTP_301)将地址重新跳转到规范化后的URL。 URL规范器可以针对URL管理器全局配置，也可以针对规则单独配置 - 默认每个规则都使用URL管理器中的规范器。 你可以针对特定的URL规则设置 [UrlRule::$normalizer](https://www.yiichina.com/doc/api/2.0/yii-web-urlrule#$normalizer-detail) 为 `false` 来关闭规范化。

```php
'urlManager' => [
    'enablePrettyUrl' => true,
    'showScriptName' => false,
    'enableStrictParsing' => true,
    'suffix' => '.html',
    'normalizer' => [
        'class' => 'yii\web\UrlNormalizer',
        // 调试时使用临时跳转代替永久跳转
        'action' => UrlNormalizer::ACTION_REDIRECT_TEMPORARY,
    ],
    'rules' => [
        // ...其它规则...
        [
            'pattern' => 'posts',
            'route' => 'post/index',
            'suffix' => '/',
            'normalizer' => false, // 针对此规则关闭规范器
        ],
        [
            'pattern' => 'tags',
            'route' => 'tag/index',
            'normalizer' => [
                // 针对此规则不合并连续的斜线
                'collapseSlashes' => false,
            ],
        ],
    ],
]
```

**注意：** 默认 [UrlManager::$normalizer](https://www.yiichina.com/doc/api/2.0/yii-web-urlmanager#$normalizer-detail) 规范器是关闭的。你需要明确配置其开启 来启用 URL 规范化。

# 请求

## 请求参数

```php
$request = Yii::$app->request;

$get = $request->get(); 
// 等价于: $get = $_GET;

$id = $request->get('id');   
// 等价于: $id = isset($_GET['id']) ? $_GET['id'] : null;

$id = $request->get('id', 1);   
// 等价于: $id = isset($_GET['id']) ? $_GET['id'] : 1;

$post = $request->post(); 
// 等价于: $post = $_POST;

$name = $request->post('name');   
// 等价于: $name = isset($_POST['name']) ? $_POST['name'] : null;

$name = $request->post('name', '');   
// 等价于: $name = isset($_POST['name']) ? $_POST['name'] : '';
```



```php
$request = Yii::$app->request;

// 返回所有参数
$params = $request->bodyParams;

// 返回参数 "id"
$param = $request->getBodyParam('id');
```

`POST`，`PUT`，`PATCH` 等等这些提交上来的参数是在请求体中被发送的。 当你通过上面介绍的方法访问这些参数的时候，`request` 组件会解析这些参数

## 请求方法

你可以通过 `Yii::$app->request->method` 表达式来获取当前请求使用的HTTP方法。 这里还提供了一整套布尔属性用于检测当前请求是某种类型

```
$request = Yii::$app->request;

if ($request->isAjax) { /* 该请求是一个 AJAX 请求 */ }
if ($request->isGet)  { /* 请求方法是 GET */ }
if ($request->isPost) { /* 请求方法是 POST */ }
if ($request->isPut)  { /* 请求方法是 PUT */ }
```

## 请求URLs

`request` 组件提供了许多方式来检测当前请求的 URL。

假设被请求的 URL 是 `http://example.com/admin/index.php/product?id=100`， 你可以像下面描述的那样获取 URL 的各个部分：

- [url](https://www.yiichina.com/doc/api/2.0/yii-web-request#$url-detail)：返回 `/admin/index.php/product?id=100`, 此 URL 不包括主机信息部分。
- [absoluteUrl](https://www.yiichina.com/doc/api/2.0/yii-web-request#$absoluteUrl-detail)：返回 `http://example.com/admin/index.php/product?id=100`, 包含host infode的整个URL。
- [hostInfo](https://www.yiichina.com/doc/api/2.0/yii-web-request#$hostInfo-detail)：返回 `http://example.com`, 只有主机信息部分。
- [pathInfo](https://www.yiichina.com/doc/api/2.0/yii-web-request#$pathInfo-detail)：返回 `/product`， 这个是入口脚本之后，问号之前（查询字符串）的部分。
- [queryString](https://www.yiichina.com/doc/api/2.0/yii-web-request#$queryString-detail)：返回 `id=100`，问号之后的部分。
- [baseUrl](https://www.yiichina.com/doc/api/2.0/yii-web-request#$baseUrl-detail)：返回 `/admin`，主机信息之后， 入口脚本之前的部分。
- [scriptUrl](https://www.yiichina.com/doc/api/2.0/yii-web-request#$scriptUrl-detail)：返回 `/admin/index.php`，没有路径信息和查询字符串部分。
- [serverName](https://www.yiichina.com/doc/api/2.0/yii-web-request#$serverName-detail)：返回 `example.com`，URL 中的主机名。
- [serverPort](https://www.yiichina.com/doc/api/2.0/yii-web-request#$serverPort-detail)：返回 80，这是 web 服务中使用的端口。

## HTTP头

你可以通过 [yii\web\Request::$headers](https://www.yiichina.com/doc/api/2.0/yii-web-request#$headers-detail) 属性返回的 [header collection](https://www.yiichina.com/doc/api/2.0/yii-web-headercollection) 获取HTTP头信息

```php
// $headers 是一个 yii\web\HeaderCollection 对象
$headers = Yii::$app->request->headers;

// 返回 Accept header 值
$accept = $headers->get('Accept');

if ($headers->has('User-Agent')) { /* 这是一个 User-Agent 头 */ }
```

请求组件也提供了支持快速访问常用头的方法，包括：

- [userAgent](https://www.yiichina.com/doc/api/2.0/yii-web-request#$userAgent-detail)：返回 `User-Agent` 头。
- [contentType](https://www.yiichina.com/doc/api/2.0/yii-web-request#$contentType-detail)：返回 `Content-Type` 头的值， `Content-Type` 是请求体中MIME类型数据。
- [acceptableContentTypes](https://www.yiichina.com/doc/api/2.0/yii-web-request#$acceptableContentTypes-detail)：返回用户可接受的内容MIME类型。 返回的类型是按照他们的质量得分来排序的。得分最高的类型将被最先返回。
- [acceptableLanguages](https://www.yiichina.com/doc/api/2.0/yii-web-request#$acceptableLanguages-detail)：返回用户可接受的语言。 返回的语言是按照他们的偏好层次来排序的。第一个参数代表最优先的语言。

## 客户端信息

你可以通过 [userHost](https://www.yiichina.com/doc/api/2.0/yii-web-request#$userHost-detail) 和 [userIP](https://www.yiichina.com/doc/api/2.0/yii-web-request#$userIP-detail) 分别获取主机名和客户机的 IP 地址

```php
$userHost = Yii::$app->request->userHost;
$userIP = Yii::$app->request->userIP;
```

# 响应

[yii\web\Response::$statusCode](https://www.yiichina.com/doc/api/2.0/yii-web-response#$statusCode-detail) 状态码默认为 200， 如果需要指定请求失败，可抛出对应的 HTTP 异常

```php
throw new \yii\web\NotFoundHttpException;
```

当[错误处理器](https://www.yiichina.com/doc/guide/2.0/runtime-handling-errors) 捕获到一个异常，会从异常中提取状态码并赋值到响应， 对于上述的 [yii\web\NotFoundHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-notfoundhttpexception) 对应 HTTP 404 状态码， 以下为 Yii 预定义的 HTTP 异常：

- [yii\web\BadRequestHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-badrequesthttpexception)：状态码 400。
- [yii\web\ConflictHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-conflicthttpexception)：状态码 409。
- [yii\web\ForbiddenHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-forbiddenhttpexception)：状态码 403。
- [yii\web\GoneHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-gonehttpexception)：状态码 410。
- [yii\web\MethodNotAllowedHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-methodnotallowedhttpexception)：状态码 405。
- [yii\web\NotAcceptableHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-notacceptablehttpexception)：状态码 406。
- [yii\web\NotFoundHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-notfoundhttpexception)：状态码 404。
- [yii\web\ServerErrorHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-servererrorhttpexception)：状态码 500。
- [yii\web\TooManyRequestsHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-toomanyrequestshttpexception)：状态码 429。
- [yii\web\UnauthorizedHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-unauthorizedhttpexception)：状态码 401。
- [yii\web\UnsupportedMediaTypeHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-unsupportedmediatypehttpexception)：状态码 415。

如果想抛出的异常不在如上列表中，可创建一个 [yii\web\HttpException](https://www.yiichina.com/doc/api/2.0/yii-web-httpexception) 异常， 带上状态码抛出，如下：

```php
throw new \yii\web\HttpException(402);
```

## HTTP头部

```php
$headers = Yii::$app->response->headers;

// 增加一个 Pragma 头，已存在的Pragma 头不会被覆盖。
$headers->add('Pragma', 'no-cache');

// 设置一个Pragma 头. 任何已存在的Pragma 头都会被丢弃
$headers->set('Pragma', 'no-cache');

// 删除Pragma 头并返回删除的Pragma 头的值到数组
$values = $headers->remove('Pragma');
```

注意：** 头名称是大小写敏感的，在 [yii\web\Response::send()](https://www.yiichina.com/doc/api/2.0/yii-web-response#send()-detail) 方法 调用前新注册的头信息并不会发送给用户

## 响应主体

如果已有格式化好的主体字符串，可赋值到响应的 [yii\web\Response::$content](https://www.yiichina.com/doc/api/2.0/yii-web-response#$content-detail) 属性

```php
Yii::$app->response->content = 'hello world!';
```

#  响应

当一个应用在处理完一个[请求](https://www.yiichina.com/doc/guide/2.0/runtime-requests)后, 这个应用会生成一个 [response](https://www.yiichina.com/doc/api/2.0/yii-web-response) 响应对象并把这个响应对象发送给终端用户 这个响应对象包含的信息有 HTTP 状态码，HTTP 头和主体内容等, 从本质上说，网页应用开发最终的目标就是根据不同的请求去构建这些响应对象。

在大多数实际应用情况下，你应该主要地去处理 `response` 这个 [应用组件](https://www.yiichina.com/doc/guide/2.0/structure-application-components)， 在默认情况下，它是一个继承自 [yii\web\Response](https://www.yiichina.com/doc/api/2.0/yii-web-response) 的实例 然而，Yii 也允许你创建自己的响应对象并发送给终端用户，这方面在后续会阐述。

在本节，我们将会讲述如何组装和构建响应并把它发送给终端用户。

## 状态码

构建响应要做的第一件事就是声明请求是否被成功处理，我们可通过设置 [yii\web\Response::$statusCode](https://www.yiichina.com/doc/api/2.0/yii-web-response#$statusCode-detail) 这个属性来做到这一点，该属性接受一个有效的 [HTTP 状态码](https://tools.ietf.org/html/rfc2616#section-10)。例如，表明该请求已被成功处理， 可以设置状态码为 200，如下所示：

```php
Yii::$app->response->statusCode = 200;
```

尽管如此，大多数情况下不需要明确设置状态码， 因为 [yii\web\Response::$statusCode](https://www.yiichina.com/doc/api/2.0/yii-web-response#$statusCode-detail) 状态码默认为 200， 如果需要指定请求失败，可抛出对应的 HTTP 异常，如下所示：

```php
throw new \yii\web\NotFoundHttpException;
```

当[错误处理器](https://www.yiichina.com/doc/guide/2.0/runtime-handling-errors) 捕获到一个异常，会从异常中提取状态码并赋值到响应， 对于上述的 [yii\web\NotFoundHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-notfoundhttpexception) 对应 HTTP 404 状态码， 以下为 Yii 预定义的 HTTP 异常：

- [yii\web\BadRequestHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-badrequesthttpexception)：状态码 400。
- [yii\web\ConflictHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-conflicthttpexception)：状态码 409。
- [yii\web\ForbiddenHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-forbiddenhttpexception)：状态码 403。
- [yii\web\GoneHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-gonehttpexception)：状态码 410。
- [yii\web\MethodNotAllowedHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-methodnotallowedhttpexception)：状态码 405。
- [yii\web\NotAcceptableHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-notacceptablehttpexception)：状态码 406。
- [yii\web\NotFoundHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-notfoundhttpexception)：状态码 404。
- [yii\web\ServerErrorHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-servererrorhttpexception)：状态码 500。
- [yii\web\TooManyRequestsHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-toomanyrequestshttpexception)：状态码 429。
- [yii\web\UnauthorizedHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-unauthorizedhttpexception)：状态码 401。
- [yii\web\UnsupportedMediaTypeHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-unsupportedmediatypehttpexception)：状态码 415。

如果想抛出的异常不在如上列表中，可创建一个 [yii\web\HttpException](https://www.yiichina.com/doc/api/2.0/yii-web-httpexception) 异常， 带上状态码抛出，如下：

```php
throw new \yii\web\HttpException(402);
```

## HTTP 头部

可在 `response` 组件中操控 [header collection](https://www.yiichina.com/doc/api/2.0/yii-web-response#$headers-detail) 来发送 HTTP 头部信息， 例如：

```php
$headers = Yii::$app->response->headers;

// 增加一个 Pragma 头，已存在的Pragma 头不会被覆盖。
$headers->add('Pragma', 'no-cache');

// 设置一个Pragma 头. 任何已存在的Pragma 头都会被丢弃
$headers->set('Pragma', 'no-cache');

// 删除Pragma 头并返回删除的Pragma 头的值到数组
$values = $headers->remove('Pragma');
```

> **信息：** 头名称是大小写敏感的，在 [yii\web\Response::send()](https://www.yiichina.com/doc/api/2.0/yii-web-response#send()-detail) 方法 调用前新注册的头信息并不会发送给用户。

## 响应主体

大多是响应应有一个主体存放你想要显示给终端用户的内容。

如果已有格式化好的主体字符串，可赋值到响应的 [yii\web\Response::$content](https://www.yiichina.com/doc/api/2.0/yii-web-response#$content-detail) 属性， 例如：

```php
Yii::$app->response->content = 'hello world!';
```

如果在发送给终端用户之前需要格式化，应设置 [format](https://www.yiichina.com/doc/api/2.0/yii-web-response#$format-detail) 和 [data](https://www.yiichina.com/doc/api/2.0/yii-web-response#$data-detail) 属性，[format](https://www.yiichina.com/doc/api/2.0/yii-web-response#$format-detail) 属性指定 [data](https://www.yiichina.com/doc/api/2.0/yii-web-response#$data-detail)中数据格式化后的样式，

```php
$response = Yii::$app->response;
$response->format = \yii\web\Response::FORMAT_JSON;
$response->data = ['message' => 'hello world'];
```

Yii支持以下可直接使用的格式，每个实现了[formatter](https://www.yiichina.com/doc/api/2.0/yii-web-responseformatterinterface) 类， 可自定义这些格式器或通过配置 [yii\web\Response::$formatters](https://www.yiichina.com/doc/api/2.0/yii-web-response#$formatters-detail) 属性来增加格式器。

- [HTML](https://www.yiichina.com/doc/api/2.0/yii-web-response#FORMAT_HTML-detail)：通过 [yii\web\HtmlResponseFormatter](https://www.yiichina.com/doc/api/2.0/yii-web-htmlresponseformatter) 来实现。
- [XML](https://www.yiichina.com/doc/api/2.0/yii-web-response#FORMAT_XML-detail)：通过 [yii\web\XmlResponseFormatter](https://www.yiichina.com/doc/api/2.0/yii-web-xmlresponseformatter) 来实现。
- [JSON](https://www.yiichina.com/doc/api/2.0/yii-web-response#FORMAT_JSON-detail)：通过 [yii\web\JsonResponseFormatter](https://www.yiichina.com/doc/api/2.0/yii-web-jsonresponseformatter) 来实现。
- [JSONP](https://www.yiichina.com/doc/api/2.0/yii-web-response#FORMAT_JSONP-detail)：通过 [yii\web\JsonResponseFormatter](https://www.yiichina.com/doc/api/2.0/yii-web-jsonresponseformatter) 来实现。
- [RAW](https://www.yiichina.com/doc/api/2.0/yii-web-response#FORMAT_RAW-detail)：如果要直接发送响应而不应用任何格式，请使用此格式。

因为响应格式默认为 [HTML](https://www.yiichina.com/doc/api/2.0/yii-web-response#FORMAT_HTML-detail)，只需要在操作方法中返回一个字符串， 如果想使用其他响应格式，应在返回数据前先设置格式，

```php
public function actionReJson()
{
    Yii::$app->response->format = Response::FORMAT_JSON;
    return [
        'message' => 'hello world!',
        'code'     => 100,
    ];
}
```

除了使用默认的 `response` 应用组件，也可创建自己的响应对象并发送给终端用户， 可在操作方法中返回该响应对象，

```php
public function actionReJson2()
{
    return Yii::createObject([
        'class' => 'yii\web\Response',
        'format' => \yii\web\Response::FORMAT_JSON,
        'data' => [
            'message' => 'hello world',
            'code' => 100,
        ],
    ]);
}
```

 如果创建你自己的响应对象，将不能在应用配置中设置 `response` 组件，尽管如此， 可使用 [依赖注入](https://www.yiichina.com/doc/guide/2.0/concept-di-container) 应用通用配置到你新的响应对象

## 浏览器跳转

可调用[yii\web\Response::redirect()](https://www.yiichina.com/doc/api/2.0/yii-web-response#redirect()-detail) 方法将用户浏览器跳转到一个 URL 地址，该方法设置合适的 带指定 URL 的 `Location` 头并返回它自己为响应对象，在操作的方法中， 可调用缩写版 [yii\web\Controller::redirect()](https://www.yiichina.com/doc/api/2.0/yii-web-controller#redirect()-detail)，

```php
public function actionOld()
{
    return $this->redirect('http://example.com/new', 301);
}
```

除了动作方法外，可直接调用[yii\web\Response::redirect()](https://www.yiichina.com/doc/api/2.0/yii-web-response#redirect()-detail) 再调用 [yii\web\Response::send()](https://www.yiichina.com/doc/api/2.0/yii-web-response#send()-detail) 方法来确保没有其他内容追加到响应中。

```php
\Yii::$app->response->redirect('http://example.com/new', 301)->send();
```

**信息：** [yii\web\Response::redirect()](https://www.yiichina.com/doc/api/2.0/yii-web-response#redirect()-detail) 方法默认会设置响应状态码为 302，该状态码会告诉浏览器请求的资源 *临时* 放在另一个 URI 地址上，可传递一个 301 状态码告知浏览器请求 的资源已经 *永久* 重定向到新的 URId 地址。

## 发送文件

- [yii\web\Response::sendFile()](https://www.yiichina.com/doc/api/2.0/yii-web-response#sendFile()-detail)：发送一个已存在的文件到客户端
- [yii\web\Response::sendContentAsFile()](https://www.yiichina.com/doc/api/2.0/yii-web-response#sendContentAsFile()-detail)：发送一个文本字符串作为文件到客户端
- [yii\web\Response::sendStreamAsFile()](https://www.yiichina.com/doc/api/2.0/yii-web-response#sendStreamAsFile()-detail)：发送一个已存在的文件流作为文件到客户端

这些方法都将响应对象作为返回值，如果要发送的文件非常大，应考虑使用 [yii\web\Response::sendStreamAsFile()](https://www.yiichina.com/doc/api/2.0/yii-web-response#sendStreamAsFile()-detail) 因为它更节约内存， 以下示例显示在控制器操作中如何发送文件：

```php
public function actionDownload()
{
    return \Yii::$app->response->sendFile('path/to/file.txt');
}
```

如果不是在操作方法中调用文件发送方法，在后面还应调用 [yii\web\Response::send()](https://www.yiichina.com/doc/api/2.0/yii-web-response#send()-detail) 没有其他内容追加到响应中。

```php
\Yii::$app->response->sendFile('path/to/file.txt')->send();
```

# Session和Cookies

## Sessions

### 开启和关闭Sessions

```php
$session = Yii::$app->session;

// 检查session是否开启 
if ($session->isActive) ...

// 开启session
$session->open();

// 关闭session
$session->close();

// 销毁session中所有已注册的数据
$session->destroy();
```

多次调用[open()](https://www.yiichina.com/doc/api/2.0/yii-web-session#open()-detail) 和[close()](https://www.yiichina.com/doc/api/2.0/yii-web-session#close()-detail) 方法并不会产生错误， 因为方法内部会先检查session是否已经开启。

```php
$session = Yii::$app->session;

// 获取session中的变量值，以下用法是相同的：
$language = $session->get('language');
$language = $session['language'];
$language = isset($_SESSION['language']) ? $_SESSION['language'] : null;

// 设置一个session变量，以下用法是相同的：
$session->set('language', 'en-US');
$session['language'] = 'en-US';
$_SESSION['language'] = 'en-US';

// 删除一个session变量，以下用法是相同的：
$session->remove('language');
unset($session['language']);
unset($_SESSION['language']);

// 检查session变量是否已存在，以下用法是相同的：
if ($session->has('language')) ...
if (isset($session['language'])) ...
if (isset($_SESSION['language'])) ...

// 遍历所有session变量，以下用法是相同的：
foreach ($session as $name => $value) ...
foreach ($_SESSION as $name => $value) ...
```

**信息：** 当使用 `session` 组件访问 session 数据时候， 如果 session 没有开启会自动开启， 这和通过 `$_SESSION` 不同，`$_SESSION` 要求先执行 `session_start()`。

当 session 数据为数组时，`session` 组件会限制你直接修改数据中的单元项， 例如：

```php
$session = Yii::$app->session;

// 如下代码不会生效
$session['captcha']['number'] = 5;
$session['captcha']['lifetime'] = 3600;

// 如下代码会生效：
$session['captcha'] = [
    'number' => 5,
    'lifetime' => 3600,
];

// 如下代码也会生效：
echo $session['captcha']['lifetime'];
```

可使用以下任意一个变通方法来解决这个问题

```php
$session = Yii::$app->session;

// 直接使用$_SESSION (确保Yii::$app->session->open() 已经调用)
$_SESSION['captcha']['number'] = 5;
$_SESSION['captcha']['lifetime'] = 3600;

// 先获取session数据到一个数组，修改数组的值，然后保存数组到session中
$captcha = $session['captcha'];
$captcha['number'] = 5;
$captcha['lifetime'] = 3600;
$session['captcha'] = $captcha;

// 使用ArrayObject 数组对象代替数组
$session['captcha'] = new \ArrayObject;
...
$session['captcha']['number'] = 5;
$session['captcha']['lifetime'] = 3600;

// 使用带通用前缀的键来存储数组
$session['captcha.number'] = 5;
$session['captcha.lifetime'] = 3600;
```

为更好的性能和可读性，推荐最后一种方案， 也就是不用存储 session 变量为数组， 而是将每个数组项变成有相同键前缀的 session 变量。

### 自定义 Session 存储

#  Sessions 和 Cookies

Sessions 和 cookies 允许数据在多次请求中保持， 在纯 PHP 中，可以分别使用全局变量 `$_SESSION` 和 `$_COOKIE` 来访问，Yii 将 session 和 cookie 封装成对象并增加一些功能， 可通过面向对象方式访问它们。

## Sessions

和 [请求](https://www.yiichina.com/doc/guide/2.0/runtime-requests) 和 [响应](https://www.yiichina.com/doc/guide/2.0/runtime-responses)类似， 默认可通过为 [yii\web\Session](https://www.yiichina.com/doc/api/2.0/yii-web-session) 实例的 `session` [应用组件](https://www.yiichina.com/doc/guide/2.0/structure-application-components) 来访问 sessions。

### 开启和关闭 Sessions

可使用以下代码来开启和关闭 session。

```php
$session = Yii::$app->session;

// 检查session是否开启 
if ($session->isActive) ...

// 开启session
$session->open();

// 关闭session
$session->close();

// 销毁session中所有已注册的数据
$session->destroy();
```

多次调用[open()](https://www.yiichina.com/doc/api/2.0/yii-web-session#open()-detail) 和[close()](https://www.yiichina.com/doc/api/2.0/yii-web-session#close()-detail) 方法并不会产生错误， 因为方法内部会先检查session是否已经开启。

### 访问 Session 数据

可使用如下方式访问session中的数据：

```php
$session = Yii::$app->session;

// 获取session中的变量值，以下用法是相同的：
$language = $session->get('language');
$language = $session['language'];
$language = isset($_SESSION['language']) ? $_SESSION['language'] : null;

// 设置一个session变量，以下用法是相同的：
$session->set('language', 'en-US');
$session['language'] = 'en-US';
$_SESSION['language'] = 'en-US';

// 删除一个session变量，以下用法是相同的：
$session->remove('language');
unset($session['language']);
unset($_SESSION['language']);

// 检查session变量是否已存在，以下用法是相同的：
if ($session->has('language')) ...
if (isset($session['language'])) ...
if (isset($_SESSION['language'])) ...

// 遍历所有session变量，以下用法是相同的：
foreach ($session as $name => $value) ...
foreach ($_SESSION as $name => $value) ...
```

> **信息：** 当使用 `session` 组件访问 session 数据时候， 如果 session 没有开启会自动开启， 这和通过 `$_SESSION` 不同，`$_SESSION` 要求先执行 `session_start()`。

当 session 数据为数组时，`session` 组件会限制你直接修改数据中的单元项， 例如：

```php
$session = Yii::$app->session;

// 如下代码不会生效
$session['captcha']['number'] = 5;
$session['captcha']['lifetime'] = 3600;

// 如下代码会生效：
$session['captcha'] = [
    'number' => 5,
    'lifetime' => 3600,
];

// 如下代码也会生效：
echo $session['captcha']['lifetime'];
```

可使用以下任意一个变通方法来解决这个问题：

```php
$session = Yii::$app->session;

// 直接使用$_SESSION (确保Yii::$app->session->open() 已经调用)
$_SESSION['captcha']['number'] = 5;
$_SESSION['captcha']['lifetime'] = 3600;

// 先获取session数据到一个数组，修改数组的值，然后保存数组到session中
$captcha = $session['captcha'];
$captcha['number'] = 5;
$captcha['lifetime'] = 3600;
$session['captcha'] = $captcha;

// 使用ArrayObject 数组对象代替数组
$session['captcha'] = new \ArrayObject;
...
$session['captcha']['number'] = 5;
$session['captcha']['lifetime'] = 3600;

// 使用带通用前缀的键来存储数组
$session['captcha.number'] = 5;
$session['captcha.lifetime'] = 3600;
```

为更好的性能和可读性，推荐最后一种方案， 也就是不用存储 session 变量为数组， 而是将每个数组项变成有相同键前缀的 session 变量。

### 自定义 Session 存储

[yii\web\Session](https://www.yiichina.com/doc/api/2.0/yii-web-session) 类默认存储 session 数据为文件到服务器上， Yii 提供以下 session 类实现不同的 session 存储方式：

- [yii\web\DbSession](https://www.yiichina.com/doc/api/2.0/yii-web-dbsession)：存储 session 数据在数据表中
- [yii\web\CacheSession](https://www.yiichina.com/doc/api/2.0/yii-web-cachesession)：存储 session 数据到缓存中，缓存和配置中的[缓存组件](https://www.yiichina.com/doc/guide/2.0/caching-data#cache-components)相关
- yii\redis\Session：存储 session 数据到以 [redis](http://redis.io/) 作为存储媒介中
- yii\mongodb\Session：存储 session 数据到 [MongoDB](http://www.mongodb.org/)。

所有这些 session 类支持相同的 API 方法集，因此， 切换到不同的 session 存储介质不需要修改项目使用 session 的代码。

如果通过`$_SESSION`访问使用自定义存储介质的session， 需要确保session已经用[yii\web\Session::open()](https://www.yiichina.com/doc/api/2.0/yii-web-session#open()-detail) 开启， 这是因为在该方法中注册自定义session存储处理器。

如下为一个示例 显示如何在应用配置中配置 [yii\web\DbSession](https://www.yiichina.com/doc/api/2.0/yii-web-dbsession) 将数据表作为 session 存储介质。

```php
return [
    'components' => [
        'session' => [
            'class' => 'yii\web\DbSession',
            // 'db' => 'mydb',  // 数据库连接的应用组件ID，默认为'db'.
            // 'sessionTable' => 'my_session', // session 数据表名，默认为'session'.
        ],
    ],
];
```

### Flash数据

#  Sessions 和 Cookies

Sessions 和 cookies 允许数据在多次请求中保持， 在纯 PHP 中，可以分别使用全局变量 `$_SESSION` 和 `$_COOKIE` 来访问，Yii 将 session 和 cookie 封装成对象并增加一些功能， 可通过面向对象方式访问它们。

## Sessions

和 [请求](https://www.yiichina.com/doc/guide/2.0/runtime-requests) 和 [响应](https://www.yiichina.com/doc/guide/2.0/runtime-responses)类似， 默认可通过为 [yii\web\Session](https://www.yiichina.com/doc/api/2.0/yii-web-session) 实例的 `session` [应用组件](https://www.yiichina.com/doc/guide/2.0/structure-application-components) 来访问 sessions。

### 开启和关闭 Sessions

可使用以下代码来开启和关闭 session。

```php
$session = Yii::$app->session;

// 检查session是否开启 
if ($session->isActive) ...

// 开启session
$session->open();

// 关闭session
$session->close();

// 销毁session中所有已注册的数据
$session->destroy();
```

多次调用[open()](https://www.yiichina.com/doc/api/2.0/yii-web-session#open()-detail) 和[close()](https://www.yiichina.com/doc/api/2.0/yii-web-session#close()-detail) 方法并不会产生错误， 因为方法内部会先检查session是否已经开启。

### 访问 Session 数据

可使用如下方式访问session中的数据：

```php
$session = Yii::$app->session;

// 获取session中的变量值，以下用法是相同的：
$language = $session->get('language');
$language = $session['language'];
$language = isset($_SESSION['language']) ? $_SESSION['language'] : null;

// 设置一个session变量，以下用法是相同的：
$session->set('language', 'en-US');
$session['language'] = 'en-US';
$_SESSION['language'] = 'en-US';

// 删除一个session变量，以下用法是相同的：
$session->remove('language');
unset($session['language']);
unset($_SESSION['language']);

// 检查session变量是否已存在，以下用法是相同的：
if ($session->has('language')) ...
if (isset($session['language'])) ...
if (isset($_SESSION['language'])) ...

// 遍历所有session变量，以下用法是相同的：
foreach ($session as $name => $value) ...
foreach ($_SESSION as $name => $value) ...
```

> **信息：** 当使用 `session` 组件访问 session 数据时候， 如果 session 没有开启会自动开启， 这和通过 `$_SESSION` 不同，`$_SESSION` 要求先执行 `session_start()`。

当 session 数据为数组时，`session` 组件会限制你直接修改数据中的单元项， 例如：

```php
$session = Yii::$app->session;

// 如下代码不会生效
$session['captcha']['number'] = 5;
$session['captcha']['lifetime'] = 3600;

// 如下代码会生效：
$session['captcha'] = [
    'number' => 5,
    'lifetime' => 3600,
];

// 如下代码也会生效：
echo $session['captcha']['lifetime'];
```

可使用以下任意一个变通方法来解决这个问题：

```php
$session = Yii::$app->session;

// 直接使用$_SESSION (确保Yii::$app->session->open() 已经调用)
$_SESSION['captcha']['number'] = 5;
$_SESSION['captcha']['lifetime'] = 3600;

// 先获取session数据到一个数组，修改数组的值，然后保存数组到session中
$captcha = $session['captcha'];
$captcha['number'] = 5;
$captcha['lifetime'] = 3600;
$session['captcha'] = $captcha;

// 使用ArrayObject 数组对象代替数组
$session['captcha'] = new \ArrayObject;
...
$session['captcha']['number'] = 5;
$session['captcha']['lifetime'] = 3600;

// 使用带通用前缀的键来存储数组
$session['captcha.number'] = 5;
$session['captcha.lifetime'] = 3600;
```

为更好的性能和可读性，推荐最后一种方案， 也就是不用存储 session 变量为数组， 而是将每个数组项变成有相同键前缀的 session 变量。

### 自定义 Session 存储

[yii\web\Session](https://www.yiichina.com/doc/api/2.0/yii-web-session) 类默认存储 session 数据为文件到服务器上， Yii 提供以下 session 类实现不同的 session 存储方式：

- [yii\web\DbSession](https://www.yiichina.com/doc/api/2.0/yii-web-dbsession)：存储 session 数据在数据表中
- [yii\web\CacheSession](https://www.yiichina.com/doc/api/2.0/yii-web-cachesession)：存储 session 数据到缓存中，缓存和配置中的[缓存组件](https://www.yiichina.com/doc/guide/2.0/caching-data#cache-components)相关
- yii\redis\Session：存储 session 数据到以 [redis](http://redis.io/) 作为存储媒介中
- yii\mongodb\Session：存储 session 数据到 [MongoDB](http://www.mongodb.org/)。

所有这些 session 类支持相同的 API 方法集，因此， 切换到不同的 session 存储介质不需要修改项目使用 session 的代码。

> **注意：** 如果通过`$_SESSION`访问使用自定义存储介质的session， 需要确保session已经用[yii\web\Session::open()](https://www.yiichina.com/doc/api/2.0/yii-web-session#open()-detail) 开启， 这是因为在该方法中注册自定义session存储处理器。

> **注意：** 如果使用自定义会话存储，则可能需要显式配置会话垃圾收集器。 一些安装的 PHP（例如 Debian）使用垃圾收集器概率为 0，并在计划任务中离线清理会话文件。 此过程不适用于您的自定义存储， 因此您需要配置 yii\web\Session::$GCProbability 以使用非零值。

学习如何配置和使用这些组件类请参考它们的 API 文档，如下为一个示例 显示如何在应用配置中配置 [yii\web\DbSession](https://www.yiichina.com/doc/api/2.0/yii-web-dbsession) 将数据表作为 session 存储介质。

```php
return [
    'components' => [
        'session' => [
            'class' => 'yii\web\DbSession',
            // 'db' => 'mydb',  // 数据库连接的应用组件ID，默认为'db'.
            // 'sessionTable' => 'my_session', // session 数据表名，默认为'session'.
        ],
    ],
];
```

也需要创建如下数据库表来存储 session 数据：

```sql
CREATE TABLE session
(
    id CHAR(40) NOT NULL PRIMARY KEY,
    expire INTEGER,
    data BLOB
)
```

其中 'BLOB' 对应你选择的数据库管理系统的 BLOB-type 类型，以下一些常用数据库管理系统的 BLOB 类型：

- MySQL: LONGBLOB
- PostgreSQL: BYTEA
- MSSQL: BLOB

> **注意：** 根据 php.ini 设置的 `session.hash_function`，你需要调整 `id` 列的长度， 例如，如果 `session.hash_function=sha256`， 应使用长度为 64 而不是 40 的 char 类型。

或者，可以通过以下迁移完成：

```php
<?php

use yii\db\Migration;

class m170529_050554_create_table_session extends Migration
{
    public function up()
    {
        $this->createTable('{{%session}}', [
            'id' => $this->char(64)->notNull(),
            'expire' => $this->integer(),
            'data' => $this->binary()
        ]);
        $this->addPrimaryKey('pk-id', '{{%session}}', 'id');
    }

    public function down()
    {
        $this->dropTable('{{%session}}');
    }
}
```

### Flash 数据

Flash 数据是一种特别的 session 数据，它一旦在某个请求中设置后， 只会在下次请求中有效，然后该数据就会自动被删除。 常用于实现只需显示给终端用户一次的信息， 如用户提交一个表单后显示确认信息。

可通过 `session` 应用组件设置或访问 `session`，例如

```php
$session = Yii::$app->session;

// 请求 #1
// 设置一个名为"postDeleted" flash 信息
$session->setFlash('postDeleted', 'You have successfully deleted your post.');

// 请求 #2
// 显示名为"postDeleted" flash 信息
echo $session->getFlash('postDeleted');

// 请求 #3
// $result 为 false，因为flash信息已被自动删除
$result = $session->hasFlash('postDeleted');
```

#  Sessions 和 Cookies

Sessions 和 cookies 允许数据在多次请求中保持， 在纯 PHP 中，可以分别使用全局变量 `$_SESSION` 和 `$_COOKIE` 来访问，Yii 将 session 和 cookie 封装成对象并增加一些功能， 可通过面向对象方式访问它们。

## Sessions

和 [请求](https://www.yiichina.com/doc/guide/2.0/runtime-requests) 和 [响应](https://www.yiichina.com/doc/guide/2.0/runtime-responses)类似， 默认可通过为 [yii\web\Session](https://www.yiichina.com/doc/api/2.0/yii-web-session) 实例的 `session` [应用组件](https://www.yiichina.com/doc/guide/2.0/structure-application-components) 来访问 sessions。

### 开启和关闭 Sessions

可使用以下代码来开启和关闭 session。

```php
$session = Yii::$app->session;

// 检查session是否开启 
if ($session->isActive) ...

// 开启session
$session->open();

// 关闭session
$session->close();

// 销毁session中所有已注册的数据
$session->destroy();
```

多次调用[open()](https://www.yiichina.com/doc/api/2.0/yii-web-session#open()-detail) 和[close()](https://www.yiichina.com/doc/api/2.0/yii-web-session#close()-detail) 方法并不会产生错误， 因为方法内部会先检查session是否已经开启。

### 访问 Session 数据

可使用如下方式访问session中的数据：

```php
$session = Yii::$app->session;

// 获取session中的变量值，以下用法是相同的：
$language = $session->get('language');
$language = $session['language'];
$language = isset($_SESSION['language']) ? $_SESSION['language'] : null;

// 设置一个session变量，以下用法是相同的：
$session->set('language', 'en-US');
$session['language'] = 'en-US';
$_SESSION['language'] = 'en-US';

// 删除一个session变量，以下用法是相同的：
$session->remove('language');
unset($session['language']);
unset($_SESSION['language']);

// 检查session变量是否已存在，以下用法是相同的：
if ($session->has('language')) ...
if (isset($session['language'])) ...
if (isset($_SESSION['language'])) ...

// 遍历所有session变量，以下用法是相同的：
foreach ($session as $name => $value) ...
foreach ($_SESSION as $name => $value) ...
```

> **信息：** 当使用 `session` 组件访问 session 数据时候， 如果 session 没有开启会自动开启， 这和通过 `$_SESSION` 不同，`$_SESSION` 要求先执行 `session_start()`。

当 session 数据为数组时，`session` 组件会限制你直接修改数据中的单元项， 例如：

```php
$session = Yii::$app->session;

// 如下代码不会生效
$session['captcha']['number'] = 5;
$session['captcha']['lifetime'] = 3600;

// 如下代码会生效：
$session['captcha'] = [
    'number' => 5,
    'lifetime' => 3600,
];

// 如下代码也会生效：
echo $session['captcha']['lifetime'];
```

可使用以下任意一个变通方法来解决这个问题：

```php
$session = Yii::$app->session;

// 直接使用$_SESSION (确保Yii::$app->session->open() 已经调用)
$_SESSION['captcha']['number'] = 5;
$_SESSION['captcha']['lifetime'] = 3600;

// 先获取session数据到一个数组，修改数组的值，然后保存数组到session中
$captcha = $session['captcha'];
$captcha['number'] = 5;
$captcha['lifetime'] = 3600;
$session['captcha'] = $captcha;

// 使用ArrayObject 数组对象代替数组
$session['captcha'] = new \ArrayObject;
...
$session['captcha']['number'] = 5;
$session['captcha']['lifetime'] = 3600;

// 使用带通用前缀的键来存储数组
$session['captcha.number'] = 5;
$session['captcha.lifetime'] = 3600;
```

为更好的性能和可读性，推荐最后一种方案， 也就是不用存储 session 变量为数组， 而是将每个数组项变成有相同键前缀的 session 变量。

### 自定义 Session 存储

[yii\web\Session](https://www.yiichina.com/doc/api/2.0/yii-web-session) 类默认存储 session 数据为文件到服务器上， Yii 提供以下 session 类实现不同的 session 存储方式：

- [yii\web\DbSession](https://www.yiichina.com/doc/api/2.0/yii-web-dbsession)：存储 session 数据在数据表中
- [yii\web\CacheSession](https://www.yiichina.com/doc/api/2.0/yii-web-cachesession)：存储 session 数据到缓存中，缓存和配置中的[缓存组件](https://www.yiichina.com/doc/guide/2.0/caching-data#cache-components)相关
- yii\redis\Session：存储 session 数据到以 [redis](http://redis.io/) 作为存储媒介中
- yii\mongodb\Session：存储 session 数据到 [MongoDB](http://www.mongodb.org/)。

所有这些 session 类支持相同的 API 方法集，因此， 切换到不同的 session 存储介质不需要修改项目使用 session 的代码。

> **注意：** 如果通过`$_SESSION`访问使用自定义存储介质的session， 需要确保session已经用[yii\web\Session::open()](https://www.yiichina.com/doc/api/2.0/yii-web-session#open()-detail) 开启， 这是因为在该方法中注册自定义session存储处理器。

> **注意：** 如果使用自定义会话存储，则可能需要显式配置会话垃圾收集器。 一些安装的 PHP（例如 Debian）使用垃圾收集器概率为 0，并在计划任务中离线清理会话文件。 此过程不适用于您的自定义存储， 因此您需要配置 yii\web\Session::$GCProbability 以使用非零值。

学习如何配置和使用这些组件类请参考它们的 API 文档，如下为一个示例 显示如何在应用配置中配置 [yii\web\DbSession](https://www.yiichina.com/doc/api/2.0/yii-web-dbsession) 将数据表作为 session 存储介质。

```php
return [
    'components' => [
        'session' => [
            'class' => 'yii\web\DbSession',
            // 'db' => 'mydb',  // 数据库连接的应用组件ID，默认为'db'.
            // 'sessionTable' => 'my_session', // session 数据表名，默认为'session'.
        ],
    ],
];
```

也需要创建如下数据库表来存储 session 数据：

```sql
CREATE TABLE session
(
    id CHAR(40) NOT NULL PRIMARY KEY,
    expire INTEGER,
    data BLOB
)
```

其中 'BLOB' 对应你选择的数据库管理系统的 BLOB-type 类型，以下一些常用数据库管理系统的 BLOB 类型：

- MySQL: LONGBLOB
- PostgreSQL: BYTEA
- MSSQL: BLOB

> **注意：** 根据 php.ini 设置的 `session.hash_function`，你需要调整 `id` 列的长度， 例如，如果 `session.hash_function=sha256`， 应使用长度为 64 而不是 40 的 char 类型。

或者，可以通过以下迁移完成：

```php
<?php

use yii\db\Migration;

class m170529_050554_create_table_session extends Migration
{
    public function up()
    {
        $this->createTable('{{%session}}', [
            'id' => $this->char(64)->notNull(),
            'expire' => $this->integer(),
            'data' => $this->binary()
        ]);
        $this->addPrimaryKey('pk-id', '{{%session}}', 'id');
    }

    public function down()
    {
        $this->dropTable('{{%session}}');
    }
}
```

### Flash 数据

Flash 数据是一种特别的 session 数据，它一旦在某个请求中设置后， 只会在下次请求中有效，然后该数据就会自动被删除。 常用于实现只需显示给终端用户一次的信息， 如用户提交一个表单后显示确认信息。

可通过 `session` 应用组件设置或访问 `session`，例如：

```php
$session = Yii::$app->session;

// 请求 #1
// 设置一个名为"postDeleted" flash 信息
$session->setFlash('postDeleted', 'You have successfully deleted your post.');

// 请求 #2
// 显示名为"postDeleted" flash 信息
echo $session->getFlash('postDeleted');

// 请求 #3
// $result 为 false，因为flash信息已被自动删除
$result = $session->hasFlash('postDeleted');
```

和普通 session 数据类似，可将任意数据存储为 flash 数据。

当调用 [yii\web\Session::setFlash()](https://www.yiichina.com/doc/api/2.0/yii-web-session#setFlash()-detail) 时, 会自动覆盖相同名的已存在的任何数据， 为将数据追加到已存在的相同名 flash 中，可改为调用 [yii\web\Session::addFlash()](https://www.yiichina.com/doc/api/2.0/yii-web-session#addFlash()-detail)。 例如:

```php
session = Yii::$app->session;

// 请求 #1
// 在名称为"alerts"的flash信息增加数据
$session->addFlash('alerts', 'You have successfully deleted your post.');
$session->addFlash('alerts', 'You have successfully added a new friend.');
$session->addFlash('alerts', 'You are promoted.');

// 请求 #2
// $alerts 为名为'alerts'的flash信息，为数组格式
$alerts = $session->getFlash('alerts');
```

> **注意：** 不要在相同名称的 flash 数据中使用 [yii\web\Session::setFlash()](https://www.yiichina.com/doc/api/2.0/yii-web-session#setFlash()-detail) 的同时也使用 [yii\web\Session::addFlash()](https://www.yiichina.com/doc/api/2.0/yii-web-session#addFlash()-detail)， 因为后一个方法会自动将 flash 信息转换为数组以使新的 flash 数据可追加进来，因此， 当你调用 [yii\web\Session::getFlash()](https://www.yiichina.com/doc/api/2.0/yii-web-session#getFlash()-detail) 时， 会发现有时获取到一个数组，有时获取到一个字符串， 取决于你调用这两个方法的顺序。

**提示：** 要显示 Flash 消息，您可以通过以下方式使用 yii\bootstrap\Alert 小部件：

```php
echo Alert::widget([
   'options' => ['class' => 'alert-info'],
   'body' => Yii::$app->session->getFlash('postDeleted'),
]);
```

## Cookies

控制器是直接处理请求和响应的部分。 因此, 应当在控制器中读取和发送 cookie 。 (译者注：意思在控制器中处理 cookie 是安全的。)

### 读取Cookies

```php
// 从 "request" 组件中获取 cookie 集合(yii\web\CookieCollection)
$cookies = Yii::$app->request->cookies;

// 获取名为 "language" cookie 的值，如果不存在，返回默认值 "en"
$language = $cookies->getValue('language', 'en');

// 另一种方式获取名为 "language" cookie 的值
if (($cookie = $cookies->get('language')) !== null) {
    $language = $cookie->value;
}

// 可将 $cookies 当作数组使用
if (isset($cookies['language'])) {
    $language = $cookies['language']->value;
}

// 判断是否存在名为 "language" 的 cookie
if ($cookies->has('language')) ...
if (isset($cookies['language'])) ...
```

### 发送Cookies

```php
// 从 "response" 组件中获取 cookie 集合(yii\web\CookieCollection)
$cookies = Yii::$app->response->cookies;

// 在要发送的响应中添加一个新的 cookie
$cookies->add(new \yii\web\Cookie([
    'name' => 'language',
    'value' => 'zh-CN',
]));

// 删除一个 cookie
$cookies->remove('language');
// 等同于以下删除代码
unset($cookies['language']);
```

### Cookie验证

Cookie 验证默认启用，可以设置 [yii\web\Request::$enableCookieValidation](https://www.yiichina.com/doc/api/2.0/yii-web-request#$enableCookieValidation-detail) 属性为 false 来禁用它， 尽管如此，我们强烈建议启用它。

> **注意：** 直接通过 `$_COOKIE` 和 `setcookie()` 读取和发送的 Cookie 不会被验证。

当使用 cookie 验证时，必须指定 [yii\web\Request::$cookieValidationKey](https://www.yiichina.com/doc/api/2.0/yii-web-request#$cookieValidationKey-detail)，它是用来生成上述的哈希值， 可通过在应用配置中配置 `request` 组件。

```php
return [
    'components' => [
        'request' => [
            'cookieValidationKey' => 'fill in a secret key here',
        ],
    ],
];
```

# 错误处理

[error handler](https://www.yiichina.com/doc/api/2.0/yii-web-errorhandler) 错误处理器默认启用， 可通过在应用的[入口脚本](https://www.yiichina.com/doc/guide/2.0/structure-entry-scripts)中定义常量`YII_ENABLE_ERROR_HANDLER`来禁用。

## 使用错误处理器

```php
return [
    'components' => [
        'errorHandler' => [
            'maxSourceLines' => 20,
        ],
    ],
];
```

使用如上代码，异常页面最多显示20条源代码。

错误处理器将所有非致命PHP错误转换成可获取异常

```php
use Yii;
use yii\base\ErrorException;

try {
    10/0;
} catch (ErrorException $e) {
    Yii::warning("Division by zero.");		//这里不会直接显示在页面，只会在日志里记录错误信息
}
```

如果你想显示一个错误页面告诉用户请求是无效的或无法处理的， 可简单地抛出一个 [HTTP exception](https://www.yiichina.com/doc/api/2.0/yii-web-httpexception)异常， 如 [yii\web\NotFoundHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-notfoundhttpexception)。 错误处理器会正确地设置响应的HTTP状态码并使用合适的错误视图页面来显示错误信息。

```php
use yii\web\NotFoundHttpException;

throw new NotFoundHttpException();
```

## 自定义错误显示

[error handler](https://www.yiichina.com/doc/api/2.0/yii-web-errorhandler)错误处理器根据常量`YII_DEBUG`的值来调整错误显示， 当`YII_DEBUG` 为 true (表示在调试模式)， 错误处理器会显示异常以及详细的函数调用栈和源代码行数来帮助调试， 当`YII_DEBUG` 为 false，只有错误信息会被显示以防止应用的敏感信息泄漏。

> **信息：** 如果异常是继承 [yii\base\UserException](https://www.yiichina.com/doc/api/2.0/yii-base-userexception)， 不管`YII_DEBUG`为何值，函数调用栈信息都不会显示， 这是因为这种错误会被认为是用户产生的错误，开发人员不需要去修正。

[error handler](https://www.yiichina.com/doc/api/2.0/yii-web-errorhandler) 错误处理器默认使用两个[视图](https://www.yiichina.com/doc/guide/2.0/structure-views)显示错误:

- `@yii/views/errorHandler/error.php`: 显示不包含函数调用栈信息的错误信息是使用， 当`YII_DEBUG` 为 false时，所有错误都使用该视图。
- `@yii/views/errorHandler/exception.php`: 显示包含函数调用栈信息的错误信息时使用。

可以配置错误处理器的 [errorView](https://www.yiichina.com/doc/api/2.0/yii-web-errorhandler#$errorView-detail) 和 [exceptionView](https://www.yiichina.com/doc/api/2.0/yii-web-errorhandler#$exceptionView-detail) 属性 使用自定义的错误显示视图。

## 使用错误动作

使用指定的错误[操作](https://www.yiichina.com/doc/guide/2.0/structure-controllers) 来自定义错误显示更方便， 为此，首先配置`errorHandler`组件的 [errorAction](https://www.yiichina.com/doc/api/2.0/yii-web-errorhandler#$errorAction-detail) 属性

```php
return [
    'components' => [
        'errorHandler' => [
            'errorAction' => 'site/error',
        ],
    ]
];
```

[errorAction](https://www.yiichina.com/doc/api/2.0/yii-web-errorhandler#$errorAction-detail) 属性使用 [路由](https://www.yiichina.com/doc/guide/2.0/structure-controllers#routes)到一个操作， 上述配置表示不用显示函数调用栈信息的错误会通过执行`site/error`操作来显示。

```php
namespace app\controllers;

use Yii;
use yii\web\Controller;

class SiteController extends Controller
{
    public function actions()
    {
        return [
            'error' => [
                'class' => 'yii\web\ErrorAction',
            ],
        ];
    }
}
```

上述代码定义`error` 操作使用[yii\web\ErrorAction](https://www.yiichina.com/doc/api/2.0/yii-web-erroraction) 类， 该类渲染名为`error`视图来显示错误。

除了使用[yii\web\ErrorAction](https://www.yiichina.com/doc/api/2.0/yii-web-erroraction), 可定义`error` 动作使用类似如下的操作方法：

```php
public function actionError()
{
    $exception = Yii::$app->errorHandler->exception;
    if ($exception !== null) {
        return $this->render('error', ['exception' => $exception]);
    }
}
```

现在应创建一个视图文件为`views/site/error.php`，在该视图文件中，如果错误动作定义为[yii\web\ErrorAction](https://www.yiichina.com/doc/api/2.0/yii-web-erroraction)， 可以访问该动作中定义的如下变量：

- `name`: 错误名称
- `message`: 错误信息
- `exception`: 更多详细信息的异常对象，如HTTP 状态码， 错误码，错误调用栈等。

> **注意：** 如果您需要在错误处理程序中重定向，请执行以下操作：

```php
Yii::$app->getResponse()->redirect($url)->send();
return;
```

### 自定义错误格式

错误处理器根据[响应](https://www.yiichina.com/doc/guide/2.0/runtime-responses)设置的格式来显示错误， 如果[response format](https://www.yiichina.com/doc/api/2.0/yii-web-response#$format-detail) 响应格式为`html`, 会使用错误或异常视图来显示错误信息，如上一小节所述。 对于其他的响应格式，错误处理器会错误信息作为数组赋值 给[yii\web\Response::$data](https://www.yiichina.com/doc/api/2.0/yii-web-response#$data-detail)属性，然后转换到对应的格式， 例如，如果响应格式为`json`，可以看到如下响应信息：

```http
HTTP/1.1 404 Not Found
Date: Sun, 02 Mar 2014 05:31:43 GMT
Server: Apache/2.2.26 (Unix) DAV/2 PHP/5.4.20 mod_ssl/2.2.26 OpenSSL/0.9.8y
Transfer-Encoding: chunked
Content-Type: application/json; charset=UTF-8

{
    "name": "Not Found Exception",
    "message": "The requested resource was not found.",
    "code": 0,
    "status": 404
}
```

可在应用配置中响应`response`组件的 `beforeSend`事件来自定义错误响应格式。

```php
return [
    // ...
    'components' => [
        'response' => [
            'class' => 'yii\web\Response',
            'on beforeSend' => function ($event) {
                $response = $event->sender;
                if ($response->data !== null) {
                    $response->data = [
                        'success' => $response->isSuccessful,
                        'data' => $response->data,
                    ];
                    $response->statusCode = 200;
                }
            },
        ],
    ],
];
```

上述代码会重新格式化错误响应

```http
HTTP/1.1 200 OK
Date: Sun, 02 Mar 2014 05:31:43 GMT
Server: Apache/2.2.26 (Unix) DAV/2 PHP/5.4.20 mod_ssl/2.2.26 OpenSSL/0.9.8y
Transfer-Encoding: chunked
Content-Type: application/json; charset=UTF-8

{
    "success": false,
    "data": {
        "name": "Not Found Exception",
        "message": "The requested resource was not found.",
        "code": 0,
        "status": 404
    }
}
```

# 日志

## 日志消息

记录日志消息就跟调用下面的日志方法一样简单：

- [Yii::trace()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#trace()-detail)：记录一条消息去跟踪一段代码是怎样运行的。这主要在开发的时候使用。
- [Yii::info()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#info()-detail)：记录一条消息来传达一些有用的信息。
- [Yii::warning()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#warning()-detail)：记录一个警告消息用来指示一些已经发生的意外。
- [Yii::error()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#error()-detail)：记录一个致命的错误，这个错误应该尽快被检查。

这些日志记录方法针对 *严重程度* 和 *类别* 来记录日志消息。 它们共享相同的函数签名 `function ($message, $category = 'application')`，`$message`代表要被 记录的日志消息，而 `$category` 是日志消息的类别。在下面的示例代码中，在默认的类别 `application` 下 记录了一条跟踪消息：

```php
Yii::trace('start calculating average revenue');
```

> 日志消息可以是字符串，也可以是复杂的数据，诸如数组或者对象。 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 的义务是正确处理日志消息。默认情况下， 假如一条日志消息不是一个字符串，它将被导出为一个字符串，通过调用 [yii\helpers\VarDumper::export()](https://www.yiichina.com/doc/api/2.0/yii-helpers-basevardumper#export()-detail)。

为了更好地组织和过滤日志消息，我们建议您为每个日志消息指定一个适当的类别。您可以为类别选择一个分层命名方案， 这将使得 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 在基于它们的分类来过滤消息变得更加容易。 一个简单而高效的命名方案是使用PHP魔术常量 `__METHOD__` 作为分类名称。 这种方式也在Yii框架的核心代码中得到应用， 例如，

```php
Yii::trace('start calculating average revenue', __METHOD__);
```

`__METHOD__` 常量计算值作为该常量出现的地方的方法名（完全限定的类名前缀）。 例如，假如上面那行代码在这个方法内被调用，则它将等于字符串 `'app\controllers\RevenueController::calculate'`。

> **注意：** 上面所描述的日志方法实际上是 [logger object](https://www.yiichina.com/doc/api/2.0/yii-log-logger) 对象（一个通过表达式 `Yii::getLogger()` 可访问的单例） 的方法 [log()](https://www.yiichina.com/doc/api/2.0/yii-log-logger#log()-detail) 的一个快捷方式。当足够的消息被记录或者当应用结束时， 日志对象将会调用一个 [message dispatcher](https://www.yiichina.com/doc/api/2.0/yii-log-dispatcher) 调度对象将已经记录的日志消息发送到已注册的 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 目标中。

## 日志目标

一个日志目标是一个 [yii\log\Target](https://www.yiichina.com/doc/api/2.0/yii-log-target) 类或者它的子类的实例。 它将通过他们的严重层级和类别来过滤日志消息，然后将它们导出到一些媒介中。 例如，一个 [database target](https://www.yiichina.com/doc/api/2.0/yii-log-dbtarget) 目标导出已经过滤的日志消息到一个数据的表里面， 而一个 [email target](https://www.yiichina.com/doc/api/2.0/yii-log-emailtarget)目标将日志消息导出到指定的邮箱地址里。

在一个应用里，通过配置在应用配置里的 `log` [application component](https://www.yiichina.com/doc/guide/2.0/structure-application-components)，你可以注册多个日志目标。 就像下面这样

```php
return [
    // 必须在引导期间加载“log”组件
    'bootstrap' => ['log'],
    // “log”组件处理带时间戳的消息。 设置 PHP 时区以创建正确的时间戳
    'timeZone' => 'PRC',
    'components' => [
        'log' => [
            'targets' => [
                [
                    'class' => 'yii\log\DbTarget',
                    'levels' => ['error', 'warning'],
                ],
                [
                    'class' => 'yii\log\EmailTarget',
                    'levels' => ['error'],
                    'categories' => ['yii\db\*'],
                    'message' => [
                       'from' => ['log@example.com'],
                       'to' => ['admin@example.com', 'developer@example.com'],
                       'subject' => 'Database errors at example.com',
                    ],
                ],
            ],
        ],
    ],
];
```

#  日志

Yii提供了一个强大的日志框架，这个框架具有高度的可定制性和可扩展性。使用这个框架， 你可以轻松地记录各种类型的消息，过滤它们， 并且将它们收集到不同的目标，诸如文件，数据库，邮件。

使用Yii日志框架涉及下面的几个步骤：

- 在你代码里的各个地方记录 [log messages](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-messages)；
- 在应用配置里通过配置 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 来过滤和导出日志消息；
- 检查由不同的目标导出的已过滤的日志消息（例如：[Yii debugger](https://www.yiichina.com/doc/guide/2.0/tool-debugger)）。

在这部分，我们主要描述前两个步骤。

## 日志消息

记录日志消息就跟调用下面的日志方法一样简单：

- [Yii::trace()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#trace()-detail)：记录一条消息去跟踪一段代码是怎样运行的。这主要在开发的时候使用。
- [Yii::info()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#info()-detail)：记录一条消息来传达一些有用的信息。
- [Yii::warning()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#warning()-detail)：记录一个警告消息用来指示一些已经发生的意外。
- [Yii::error()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#error()-detail)：记录一个致命的错误，这个错误应该尽快被检查。

这些日志记录方法针对 *严重程度* 和 *类别* 来记录日志消息。 它们共享相同的函数签名 `function ($message, $category = 'application')`，`$message`代表要被 记录的日志消息，而 `$category` 是日志消息的类别。在下面的示例代码中，在默认的类别 `application` 下 记录了一条跟踪消息：

```php
Yii::trace('start calculating average revenue');
```

> **注意：** 日志消息可以是字符串，也可以是复杂的数据，诸如数组或者对象。 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 的义务是正确处理日志消息。默认情况下， 假如一条日志消息不是一个字符串，它将被导出为一个字符串，通过调用 [yii\helpers\VarDumper::export()](https://www.yiichina.com/doc/api/2.0/yii-helpers-basevardumper#export()-detail)。

为了更好地组织和过滤日志消息，我们建议您为每个日志消息指定一个适当的类别。您可以为类别选择一个分层命名方案， 这将使得 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 在基于它们的分类来过滤消息变得更加容易。 一个简单而高效的命名方案是使用PHP魔术常量 `__METHOD__` 作为分类名称。 这种方式也在Yii框架的核心代码中得到应用， 例如，

```php
Yii::trace('start calculating average revenue', __METHOD__);
```

`__METHOD__` 常量计算值作为该常量出现的地方的方法名（完全限定的类名前缀）。 例如，假如上面那行代码在这个方法内被调用，则它将等于字符串 `'app\controllers\RevenueController::calculate'`。

> **注意：** 上面所描述的日志方法实际上是 [logger object](https://www.yiichina.com/doc/api/2.0/yii-log-logger) 对象（一个通过表达式 `Yii::getLogger()` 可访问的单例） 的方法 [log()](https://www.yiichina.com/doc/api/2.0/yii-log-logger#log()-detail) 的一个快捷方式。当足够的消息被记录或者当应用结束时， 日志对象将会调用一个 [message dispatcher](https://www.yiichina.com/doc/api/2.0/yii-log-dispatcher) 调度对象将已经记录的日志消息发送到已注册的 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 目标中。

## 日志目标

一个日志目标是一个 [yii\log\Target](https://www.yiichina.com/doc/api/2.0/yii-log-target) 类或者它的子类的实例。 它将通过他们的严重层级和类别来过滤日志消息，然后将它们导出到一些媒介中。 例如，一个 [database target](https://www.yiichina.com/doc/api/2.0/yii-log-dbtarget) 目标导出已经过滤的日志消息到一个数据的表里面， 而一个 [email target](https://www.yiichina.com/doc/api/2.0/yii-log-emailtarget)目标将日志消息导出到指定的邮箱地址里。

在一个应用里，通过配置在应用配置里的 `log` [application component](https://www.yiichina.com/doc/guide/2.0/structure-application-components)，你可以注册多个日志目标。 就像下面这样：

```php
return [
    // 必须在引导期间加载“log”组件
    'bootstrap' => ['log'],
    // “log”组件处理带时间戳的消息。 设置 PHP 时区以创建正确的时间戳
    'timeZone' => 'PRC',
    'components' => [
        'log' => [
            'targets' => [
                [
                    'class' => 'yii\log\DbTarget',
                    'levels' => ['error', 'warning'],
                ],
                [
                    'class' => 'yii\log\EmailTarget',
                    'levels' => ['error'],
                    'categories' => ['yii\db\*'],
                    'message' => [
                       'from' => ['log@example.com'],
                       'to' => ['admin@example.com', 'developer@example.com'],
                       'subject' => 'Database errors at example.com',
                    ],
                ],
            ],
        ],
    ],
];
```

在上面的代码中，在 [yii\log\Dispatcher::$targets](https://www.yiichina.com/doc/api/2.0/yii-log-dispatcher#$targets-detail) 属性里有两个日志目标被注册：

- 第一个目标选择的是错误和警告层级的消息，并且在数据库表里保存他们；
- 第二个目标选择的是错误层级的消息并且是在以 `yii\db\` 开头的分类下，并且在一个邮件里将它们发送到 `admin@example.com` 和 `developer@example.com`。

Yii配备了以下的内建日志目标。请参考关于这些类的API文档， 并且学习怎样配置和使用他们。

- [yii\log\DbTarget](https://www.yiichina.com/doc/api/2.0/yii-log-dbtarget)：在数据库表里存储日志消息。
- [yii\log\EmailTarget](https://www.yiichina.com/doc/api/2.0/yii-log-emailtarget)：发送日志消息到预先指定的邮箱地址。
- [yii\log\FileTarget](https://www.yiichina.com/doc/api/2.0/yii-log-filetarget)：保存日志消息到文件中.
- [yii\log\SyslogTarget](https://www.yiichina.com/doc/api/2.0/yii-log-syslogtarget)：通过调用PHP函数 `syslog()` 将日志消息保存到系统日志里。

### 消息过滤

对于每一个日志目标，你可以配置它的 [levels](https://www.yiichina.com/doc/api/2.0/yii-log-target#$levels-detail) 和 [categories](https://www.yiichina.com/doc/api/2.0/yii-log-target#$categories-detail) 属性来指定哪个消息的严重程度和分类目标应该处理

[levels](https://www.yiichina.com/doc/api/2.0/yii-log-target#$levels-detail) 属性是由一个或者若干个以下值组成的数组：

- `error`：相应的消息通过 [Yii::error()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#error()-detail) 被记录。
- `warning`：相应的消息通过 [Yii::warning()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#warning()-detail) 被记录。
- `info`：相应的消息通过 [Yii::info()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#info()-detail) 被记录。
- `trace`：相应的消息通过 [Yii::trace()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#trace()-detail) 被记录。
- `profile`：相应的消息通过 [Yii::beginProfile()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#beginProfile()-detail) 和 [Yii::endProfile()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#endProfile()-detail) 被记录。更多细节将在 [Profiling](https://www.yiichina.com/doc/guide/2.0/runtime-logging#performance-profiling) 分段解释。

如果你没有指定 [levels](https://www.yiichina.com/doc/api/2.0/yii-log-target#$levels-detail) 的属性， 那就意味着目标将处理 *任何* 严重程度的消息。

[categories](https://www.yiichina.com/doc/api/2.0/yii-log-target#$categories-detail) 属性是一个包含消息分类名称或者模式的数组。 一个目标将只处理那些在这个数组中能够找到对应的分类或者其中一个相匹配的模式的消息。 一个分类模式是一个以星号 `*` 结尾的分类名前缀。假如一个分类名与分类模式具有相同的前缀， 那么该分类名将和分类模式相匹配。例如， `yii\db\Command::execute` 和 `yii\db\Command::query` 都是作为分类名称运用在 [yii\db\Command](https://www.yiichina.com/doc/api/2.0/yii-db-command) 类来记录日志消息的。 它们都是匹配模式 `yii\db\*`。

假如你没有指定 [categories](https://www.yiichina.com/doc/api/2.0/yii-log-target#$categories-detail) 属性， 这意味着目标将会处理 *任何* 分类的消息。

```php
[
    'class' => 'yii\log\FileTarget',
    'levels' => ['error', 'warning'],
    'categories' => [
        'yii\db\*',
        'yii\web\HttpException:*',
    ],
    'except' => [
        'yii\web\HttpException:404',
    ],
]
```

> **注意：** 当一个HTTP异常通过 [error handler](https://www.yiichina.com/doc/guide/2.0/runtime-handling-errors) 被捕获的时候，一个错误消息将以 `yii\web\HttpException:ErrorCode` 这样的格式的分类名被记录下来。例如，[yii\web\NotFoundHttpException](https://www.yiichina.com/doc/api/2.0/yii-web-notfoundhttpexception) 将会引发一个分类是 `yii\web\HttpException:404` 的 错误消息

### 消息格式化

日志目标以某种格式导出过滤过的日志消息。例如， 假如你安装一个 [yii\log\FileTarget](https://www.yiichina.com/doc/api/2.0/yii-log-filetarget) 类的日志目标， 你应该能找出一个日志消息类似下面的 `runtime/log/app.log` 文件

```php
2014-10-04 18:10:15 [::1][][-][trace][yii\base\Module::getModule] Loading module: debug
```

默认情况下，日志消息将被格式化，格式化的方式遵循 [yii\log\Target::formatMessage()](https://www.yiichina.com/doc/api/2.0/yii-log-target#formatMessage()-detail)：

```php
Timestamp [IP address][User ID][Session ID][Severity Level][Category] Message Text
```

你可以通过配置 [yii\log\Target::$prefix](https://www.yiichina.com/doc/api/2.0/yii-log-target#$prefix-detail) 的属性来自定义格式，这个属性是一个 PHP 可调用体返回的自定义消息前缀。 例如，下面的代码配置了一个日志目标的前缀是每个日志消息中当前用户的 ID（IP 地址和 Session ID 被删除是由于隐私的原因）

```php
[
    'class' => 'yii\log\FileTarget',
    'prefix' => function ($message) {
        $user = Yii::$app->has('user', true) ? Yii::$app->get('user') : null;
        $userID = $user ? $user->getId(false) : '-';
        return "[$userID]";
    }
]
```

除了消息前缀以外，日志目标也可以追加一些上下文信息到每组日志消息中。 默认情况下，这些全局的PHP变量的值被包含在：`$_GET`，`$_POST`，`$_FILES`，`$_COOKIE`，`$_SESSION` 和 `$_SERVER` 中。 你可以通过配置 [yii\log\Target::$logVars](https://www.yiichina.com/doc/api/2.0/yii-log-target#$logVars-detail) 属性适应这个行为， 这个属性是你想要通过日志目标包含的全局变量名称。 举个例子，下面的日志目标配置指明了只有 `$_SERVER` 变量的值将被追加到日志消息中。

```php
[
    'class' => 'yii\log\FileTarget',
    'logVars' => ['_SERVER'],
]
```

你可以将 `logVars` 配置成一个空数组来完全禁止上下文信息包含。 或者假如你想要实现你自己提供上下文信息的方式， 你可以重写 [yii\log\Target::getContextMessage()](https://www.yiichina.com/doc/api/2.0/yii-log-target#getContextMessage()-detail) 方法。

### 消息跟踪级别

在开发的时候，通常希望看到每个日志消息来自哪里。这个是能够被实现的，通过配置 `log` 组件的 [traceLevel](https://www.yiichina.com/doc/api/2.0/yii-log-dispatcher#$traceLevel-detail) 属性， 就像下面这样

```php
return [
    'bootstrap' => ['log'],
    'components' => [
        'log' => [
            'traceLevel' => YII_DEBUG ? 3 : 0,
            'targets' => [...],
        ],
    ],
];
```

上面的应用配置设置了 [traceLevel](https://www.yiichina.com/doc/api/2.0/yii-log-dispatcher#$traceLevel-detail) 的层级，假如 `YII_DEBUG` 开启则是3，否则是0。 这意味着，假如 `YII_DEBUG` 开启，每个日志消息在日志消息被记录的时候， 将被追加最多3个调用堆栈层级；假如 `YII_DEBUG` 关闭， 那么将没有调用堆栈信息被包含。

### 消息刷新和导出

通过 [logger object](https://www.yiichina.com/doc/api/2.0/yii-log-logger) 对象，日志消息被保存在一个数组里。 为了这个数组的内存消耗，当数组积累了一定数量的日志消息， 日志对象每次都将刷新被记录的消息到 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 中。 你可以通过配置 `log` 组件的 [flushInterval](https://www.yiichina.com/doc/api/2.0/yii-log-dispatcher#$flushInterval-detail) 属性来自定义数量：

```php
return [
    'bootstrap' => ['log'],
    'components' => [
        'log' => [
            'flushInterval' => 100,   // default is 1000
            'targets' => [...],
        ],
    ],
];
```

> **注意：** 当应用结束的时候，消息刷新也会发生，这样才能确保日志目标能够接收完整的日志消息。

当 [logger object](https://www.yiichina.com/doc/api/2.0/yii-log-logger) 对象刷新日志消息到 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 的时候，它们并 不能立即获取导出的消息。相反，消息导出仅仅在一个日志目标累积了一定数量的过滤消息的时候才会发生。你可以通过配置 个别的 [log targets](https://www.yiichina.com/doc/guide/2.0/runtime-logging#log-targets) 的 [exportInterval](https://www.yiichina.com/doc/api/2.0/yii-log-target#$exportInterval-detail) 属性来 自定义这个数量，就像下面这样：

```php
[
    'class' => 'yii\log\FileTarget',
    'exportInterval' => 100,  // default is 1000
]
```

因为刷新和导出层级的设置，默认情况下，当你调用 `Yii::trace()` 或者任何其他的记录方法，你将不能在日志目标中立即看到日志消息。 这对于一些长期运行的控制台应用来说可能是一个问题。为了让每个日志消息在日志目标中能够立即出现， 你应该设置 [flushInterval](https://www.yiichina.com/doc/api/2.0/yii-log-dispatcher#$flushInterval-detail) 和 [exportInterval](https://www.yiichina.com/doc/api/2.0/yii-log-target#$exportInterval-detail) 都为1， 就像下面这样：

```
return [
    'bootstrap' => ['log'],
    'components' => [
        'log' => [
            'flushInterval' => 1,
            'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                    'exportInterval' => 1,
                ],
            ],
        ],
    ],
];
```

### 切换日志目标

你可以通过配置 [enabled](https://www.yiichina.com/doc/api/2.0/yii-log-target#$enabled-detail) 属性来开启或者禁用日志目标。 你可以通过日志目标配置去做，或者是在你的代码中放入下面的PHP申明：

```
Yii::$app->log->targets['file']->enabled = false;
```

上面的代码要求您将目标命名为 `file`，像下面展示的那样， 在 `targets` 数组中使用字符串键：

```php
return [
    'bootstrap' => ['log'],
    'components' => [
        'log' => [
            'targets' => [
                'file' => [
                    'class' => 'yii\log\FileTarget',
                ],
                'db' => [
                    'class' => 'yii\log\DbTarget',
                ],
            ],
        ],
    ],
];
```



# 关键概念

## 配置

在 Yii 中，创建新对象和初始化已存在对象时广泛使用配置。 配置通常包含被创建对象的类名和一组将要赋值给对象 [属性](https://www.yiichina.com/doc/guide/2.0/concept-properties)的初始值。 还可能包含一组将被附加到对象[事件](https://www.yiichina.com/doc/guide/2.0/concept-events)上的句柄。 和一组将被附加到对象上的[行为](https://www.yiichina.com/doc/guide/2.0/concept-behaviors)。

以下代码中的配置被用来创建并初始化一个数据库连接：

```php
$config = [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=127.0.0.1;dbname=demo',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8',
];

$db = Yii::createObject($config);
```

[Yii::createObject()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#createObject()-detail) 方法接受一个配置数组并根据数组中指定的类名创建对象。 对象实例化后，剩余的参数被用来初始化对象的属性， 事件处理和行为。

对于已存在的对象，可以使用 [Yii::configure()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#configure()-detail) 方法根据配置去初始化其属性， 就像这样：

```php
Yii::configure($object, $config);
```

> 请注意，如果配置一个已存在的对象，那么配置数组中不应该包含指定类名的 `class` 元素。

## 配置的格式

一个配置的格式可以描述为以下形式

```php
[
    'class' => 'ClassName',
    'propertyName' => 'propertyValue',
    'on eventName' => $eventHandler,
    'as behaviorName' => $behaviorConfig,
]
```

其中

- `class` 元素指定了将要创建的对象的完全限定类名。
- `propertyName` 元素指定了对象属性的初始值。键名是属性名，值是该属性对应的初始值。 只有公共成员变量以及通过 getter/setter 定义的 [属性](https://www.yiichina.com/doc/guide/2.0/concept-properties)可以被配置。
- `on eventName` 元素指定了附加到对象[事件](https://www.yiichina.com/doc/guide/2.0/concept-events)上的句柄是什么。 请注意，数组的键名由 `on `前缀加事件名组成。 请参考[事件](https://www.yiichina.com/doc/guide/2.0/concept-events)章节了解事件句柄格式。
- `as behaviorName` 元素指定了附加到对象的[行为](https://www.yiichina.com/doc/guide/2.0/concept-behaviors)。 请注意，数组的键名由 `as `前缀加行为名组成。`$behaviorConfig` 值表示创建行为的配置信息，格式与我们之前描述的配置格式一样。

下面是一个配置了初始化属性值，事件句柄和行为的示例

```php
[
    'class' => 'app\components\SearchEngine',
    'apiKey' => 'xxxxxxxx',
    'on search' => function ($event) {
        Yii::info("搜索的关键词： " . $event->keyword);
    },
    'as indexer' => [
        'class' => 'app\components\IndexerBehavior',
        // ... 初始化属性值 ...
    ],
]
```

## 使用配置

Yii 中的配置可以用在很多场景。本章开头我们展示了如何使用 Yii::creatObject() 根据配置信息创建对象。本小节将介绍配置的两种 主要用法 —— 配置应用与配置小部件。

### 应用的配置

[应用](https://www.yiichina.com/doc/guide/2.0/structure-applications)的配置可能是最复杂的配置之一。 因为 [application](https://www.yiichina.com/doc/api/2.0/yii-web-application) 类拥有很多可配置的属性和事件。 更重要的是它的 [components](https://www.yiichina.com/doc/api/2.0/yii-di-servicelocator#$components-detail) 属性可以接收配置数组并通过应用注册为组件。 以下是一个针对[基础应用模板](https://www.yiichina.com/doc/guide/2.0/start-installation)的应用配置概要

```php
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'extensions' => require __DIR__ . '/../vendor/yiisoft/extensions.php',
    'components' => [
        'cache' => [
            'class' => 'yii\caching\FileCache',
        ],
        'mailer' => [
            'class' => 'yii\swiftmailer\Mailer',
        ],
        'log' => [
            'class' => 'yii\log\Dispatcher',
            'traceLevel' => YII_DEBUG ? 3 : 0,
            'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                ],
            ],
        ],
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=stay2',
            'username' => 'root',
            'password' => '',
            'charset' => 'utf8',
        ],
    ],
];
```

配置中没有 `class` 键的原因是这段配置应用在下面的入口脚本中， 类名已经指定了

```php
(new yii\web\Application($config))->run();
```

自版本 2.0.11 开始，系统配置支持使用 `container` 属性来配置[依赖注入容器](https://www.yiichina.com/doc/guide/2.0/concept-di-container) 例如：

```php
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'extensions' => require __DIR__ . '/../vendor/yiisoft/extensions.php',
    'container' => [
        'definitions' => [
            'yii\widgets\LinkPager' => ['maxButtonCount' => 5]
        ],
        'singletons' => [
            // 依赖注入容器单例配置
        ]
    ]
];
```

### 小部件的配置

使用[小部件](https://www.yiichina.com/doc/guide/2.0/structure-widgets)时，常常需要配置以便自定义其属性。 [yii\base\Widget::widget()](https://www.yiichina.com/doc/api/2.0/yii-base-widget#widget()-detail) 和 [yii\base\Widget::begin()](https://www.yiichina.com/doc/api/2.0/yii-base-widget#begin()-detail) 方法都可以用来创建小部件。 它们可以接受配置数组

```php
use yii\widgets\Menu;

echo Menu::widget([
    'activateItems' => false,
    'items' => [
        ['label' => 'Home', 'url' => ['site/index']],
        ['label' => 'Products', 'url' => ['product/index']],
        ['label' => 'Login', 'url' => ['site/login'], 'visible' => Yii::$app->user->isGuest],
    ],
]);
```

上述代码创建了一个小部件 `Menu` 并将其 `activateItems` 属性初始化为 false。 `item` 属性也配置成了将要显示的菜单条目。

请注意，代码中已经给出了类名 `yii\widgets\Menu`，配置数组**不应该**再包含 `class` 键。

## 配置文件

当配置的内容十分复杂，通用做法是将其存储在一或多个 PHP 文件中， 这些文件被称为*配置文件*。一个配置文件返回的是 PHP 数组。 例如，像这样把应用配置信息存储在名为 `web.php` 的文件中

```php
return [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'extensions' => require __DIR__ . '/../vendor/yiisoft/extensions.php',
    'components' => require __DIR__ . '/components.php',
];
```

鉴于 `components` 配置也很复杂，上述代码把它们存储在单独的 `components.php` 文件中，并且包含在 `web.php` 里。 `components.php` 的内容如下：

```php
return [
    'cache' => [
        'class' => 'yii\caching\FileCache',
    ],
    'mailer' => [
        'class' => 'yii\swiftmailer\Mailer',
    ],
    'log' => [
        'class' => 'yii\log\Dispatcher',
        'traceLevel' => YII_DEBUG ? 3 : 0,
        'targets' => [
            [
                'class' => 'yii\log\FileTarget',
            ],
        ],
    ],
    'db' => [
        'class' => 'yii\db\Connection',
        'dsn' => 'mysql:host=localhost;dbname=stay2',
        'username' => 'root',
        'password' => '',
        'charset' => 'utf8',
    ],
];
```

仅仅需要 “require”，就可以取得一个配置文件的配置内容，像这样：

```php
$config = require 'path/to/web.php';
(new yii\web\Application($config))->run();
```

## 默认配置

[Yii::createObject()](https://www.yiichina.com/doc/api/2.0/yii-baseyii#createObject()-detail) 方法基于[依赖注入容器](https://www.yiichina.com/doc/guide/2.0/concept-di-container)实现。 使用 Yii::creatObject() 创建对象时，可以附加一系列**默认配置**到指定类的任何实例。 默认配置还可以在[入口脚本](https://www.yiichina.com/doc/guide/2.0/runtime-bootstrapping) 中调用 `Yii::$container->set()` 来定义。

例如，如果你想自定义 [yii\widgets\LinkPager](https://www.yiichina.com/doc/api/2.0/yii-widgets-linkpager) 小部件，以便让分页器最多只显示 5 个翻页按钮（默认是 10 个）， 你可以用下述代码实现：

```php
\Yii::$container->set('yii\widgets\LinkPager', [
    'maxButtonCount' => 5,
]);
```

不使用默认配置的话，你就得在任何使用分页器的地方， 都配置 `maxButtonCount` 的值。

## 环境常量

配置经常要随着应用运行的不同环境更改。例如在开发环境中， 你可能使用名为 `mydb_dev` 的数据库， 而生产环境则使用 `mydb_prod` 数据库。 为了便于切换使用环境，Yii 提供了一个定义在入口脚本中的 `YII_ENV` 常量。 如下：

```php
defined('YII_ENV') or define('YII_ENV', 'dev');
```

你可以把 `YII_ENV` 定义成以下任何一种值

- `prod`：生产环境。常量 `YII_ENV_PROD` 将被看作 true。 如果你没修改过，这就是 `YII_ENV` 的默认值。
- `dev`：开发环境。常量 `YII_ENV_DEV` 将被看作 true。
- `test`：测试环境。常量 `YII_ENV_TEST` 将被看作 true。

有了这些环境常量，你就可以根据当下应用运行环境的不同，进行差异化配置。 例如，应用可以包含下述代码只在开发环境中开启 [调试工具](https://www.yiichina.com/doc/guide/2.0/tool-debugger)。

```php
$config = [...];

if (YII_ENV_DEV) {
    // 根据 `dev` 环境进行的配置调整
    $config['bootstrap'][] = 'debug';
    $config['modules']['debug'] = 'yii\debug\Module';
}

return $config;
```



# 数据库工作

## 数据库访问

### 创建数据库连接

想要访问数据库，你首先需要通过创建一个 [yii\db\Connection](https://www.yiichina.com/doc/api/2.0/yii-db-connection) 实例来与之建立连接。

```php
$db = new yii\db\Connection([
    'dsn' => 'mysql:host=localhost;dbname=example',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8',
]);
```

常见的做法是以[应用组件](https://www.yiichina.com/doc/guide/2.0/structure-application-components)的方式来配置它，如下:

```php
return [
    // ...
    'components' => [
        // ...
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=example',
            'username' => 'root',
            'password' => '',
            'charset' => 'utf8',
        ],
    ],
    // ...
];
```

之后你就可以通过语句 `Yii::$app->db` 来使用数据库连接了

> 如果你的应用需要访问多个数据库，你可以配置多个 DB 应用组件。

配置数据库连接时， 你应该总是通过 [dsn](https://www.yiichina.com/doc/api/2.0/yii-db-connection#$dsn-detail) 属性来指明它的数据源名称 (DSN) 。 不同的数据库有着不同的 DSN 格式。 请参考 [PHP manual](http://www.php.net/manual/en/function.PDO-construct.php) 来获得更多细节。下面是一些例子：

- MySQL, MariaDB: `mysql:host=localhost;dbname=mydatabase`
- SQLite: `sqlite:/path/to/database/file`
- PostgreSQL: `pgsql:host=localhost;port=5432;dbname=mydatabase`
- CUBRID: `cubrid:dbname=demodb;host=localhost;port=33000`
- MS SQL Server (via sqlsrv driver): `sqlsrv:Server=localhost;Database=mydatabase`
- MS SQL Server (via dblib driver): `dblib:host=localhost;dbname=mydatabase`
- MS SQL Server (via mssql driver): `mssql:host=localhost;dbname=mydatabase`
- Oracle: `oci:dbname=//localhost:1521/mydatabase`

> **信息：** 当你实例化一个 DB Connection 时，直到你第一次执行 SQL 或者你明确地调用 [open()](https://www.yiichina.com/doc/api/2.0/yii-db-connection#open()-detail) 方法时， 才建立起实际的数据库连接。

> **提示：** 有时你可能想要在建立起数据库连接时立即执行一些语句来初始化一些环境变量 (比如设置时区或者字符集), 你可以通过为数据库连接的 [afterOpen](https://www.yiichina.com/doc/api/2.0/yii-db-connection#EVENT_AFTER_OPEN-detail) 事件注册一个事件处理器来达到目的。 你可以像这样直接在应用配置中注册处理器：

```php
'db' => [
    // ...
    'on afterOpen' => function($event) {
        // $event->sender refers to the DB connection
        $event->sender->createCommand("SET time_zone = 'UTC'")->execute();
    }
],
```

### 执行 SQL 查询

一旦你拥有了 DB Connection 实例，你可以按照下列步骤来执行 SQL 查询：

1. 使用纯SQL查询来创建出 [yii\db\Command](https://www.yiichina.com/doc/api/2.0/yii-db-command)；
2. 绑定参数 (可选的)；
3. 调用 [yii\db\Command](https://www.yiichina.com/doc/api/2.0/yii-db-command) 里 SQL 执行方法中的一个。

下列例子展示了几种不同的从数据库取得数据的方法：

```php
// 返回多行. 每行都是列名和值的关联数组.
// 如果该查询没有结果则返回空数组
$posts = Yii::$app->db->createCommand('SELECT * FROM post')
            ->queryAll();

// 返回一行 (第一行)
// 如果该查询没有结果则返回 false
$post = Yii::$app->db->createCommand('SELECT * FROM post WHERE id=1')
           ->queryOne();

// 返回一列 (第一列)
// 如果该查询没有结果则返回空数组
$titles = Yii::$app->db->createCommand('SELECT title FROM post')
             ->queryColumn();

// 返回一个标量值
// 如果该查询没有结果则返回 false
$count = Yii::$app->db->createCommand('SELECT COUNT(*) FROM post')
             ->queryScalar();
```

> 注意：** 为了保持精度，即使对应的数据库列类型为数值型， 所有从数据库取得的数据都被表现为字符串。

### 绑定参数

当使用带参数的 SQL 来创建数据库命令时， 你几乎总是应该使用绑定参数的方法来防止 SQL 注入攻击

```
$post = Yii::$app->db->createCommand('SELECT * FROM post WHERE id=:id AND status=:status')
           ->bindValue(':id', $_GET['id'])
           ->bindValue(':status', 1)
           ->queryOne();
```

在 SQL 语句中，你可以嵌入一个或多个参数占位符(例如，上述例子中的 `:id` )。 一个参数占位符应该是以冒号开头的字符串。 之后你可以调用下面绑定参数的方法来绑定参数值：

- [bindValue()](https://www.yiichina.com/doc/api/2.0/yii-db-command#bindValue()-detail)：绑定一个参数值
- [bindValues()](https://www.yiichina.com/doc/api/2.0/yii-db-command#bindValues()-detail)：在一次调用中绑定多个参数值
- [bindParam()](https://www.yiichina.com/doc/api/2.0/yii-db-command#bindParam()-detail)：与 [bindValue()](https://www.yiichina.com/doc/api/2.0/yii-db-command#bindValue()-detail) 相似，但是也支持绑定参数引用。

下面的例子展示了几个可供选择的绑定参数的方法：

```php
$params = [':id' => $_GET['id'], ':status' => 1];

$post = Yii::$app->db->createCommand('SELECT * FROM post WHERE id=:id AND status=:status')
           ->bindValues($params)
           ->queryOne();
           
$post = Yii::$app->db->createCommand('SELECT * FROM post WHERE id=:id AND status=:status', $params)
           ->queryOne();
```

绑定参数是通过 [预处理语句](http://php.net/manual/en/mysqli.quickstart.prepared-statements.php) 实现的。 除了防止 SQL 注入攻击，它也可以通过一次预处理 SQL 语句， 使用不同参数多次执行，来提升性能。例如：

```php
$command = Yii::$app->db->createCommand('SELECT * FROM post WHERE id=:id');

$post1 = $command->bindValue(':id', 1)->queryOne();
$post2 = $command->bindValue(':id', 2)->queryOne();
// ...
```

因为 [bindParam()](https://www.yiichina.com/doc/api/2.0/yii-db-command#bindParam()-detail) 支持通过引用来绑定参数， 上述代码也可以像下面这样写：

```php
command = Yii::$app->db->createCommand('SELECT * FROM post WHERE id=:id')
              ->bindParam(':id', $id);

$id = 1;
$post1 = $command->queryOne();

$id = 2;
$post2 = $command->queryOne();
// ...
```

请注意，在执行语句前你将占位符绑定到 `$id` 变量， 然后在之后的每次执行前改变变量的值（这通常是用循环来完成的）。 以这种方式执行查询比为每个不同的参数值执行一次新的查询要高效得多得多。

### 执行非查询语句

上面部分中介绍的 `queryXyz()` 方法都处理的是从数据库返回数据的查询语句。对于那些不取回数据的语句， 你应该调用的是 [yii\db\Command::execute()](https://www.yiichina.com/doc/api/2.0/yii-db-command#execute()-detail) 方法。例如

```php
Yii::$app->db->createCommand('UPDATE post SET status=1 WHERE id=1')
   ->execute();
```

[yii\db\Command::execute()](https://www.yiichina.com/doc/api/2.0/yii-db-command#execute()-detail) 方法返回执行 SQL 所影响到的行数。

对于 INSERT, UPDATE 和 DELETE 语句，不再需要写纯SQL语句了， 你可以直接调用 [insert()](https://www.yiichina.com/doc/api/2.0/yii-db-command#insert()-detail)、[update()](https://www.yiichina.com/doc/api/2.0/yii-db-command#update()-detail)、[delete()](https://www.yiichina.com/doc/api/2.0/yii-db-command#delete()-detail)， 来构建相应的 SQL 语句。这些方法将正确地引用表和列名称以及绑定参数值。例如,

```php
// INSERT (table name, column values)
Yii::$app->db->createCommand()->insert('user', [
    'name' => 'Sam',
    'age' => 30,
])->execute();

// UPDATE (table name, column values, condition)
Yii::$app->db->createCommand()->update('user', ['status' => 1], 'age > 30')->execute();

// DELETE (table name, condition)
Yii::$app->db->createCommand()->delete('user', 'status = 0')->execute();
```

你也可以调用 [batchInsert()](https://www.yiichina.com/doc/api/2.0/yii-db-command#batchInsert()-detail) 来一次插入多行， 这比一次插入一行要高效得多：

```php
// table name, column names, column values
Yii::$app->db->createCommand()->batchInsert('user', ['name', 'age'], [
    ['Tom', 30],
    ['Jane', 20],
    ['Linda', 25],
])->execute();
```

另一个有用的方法是 [upsert()](https://www.yiichina.com/doc/api/2.0/yii-db-command#upsert()-detail)。Upsert 是一种原子操作，如果它们不存在（匹配唯一约束），则将行插入到数据库表中， 或者在它们执行时更新它们：

```php
Yii::$app->db->createCommand()->upsert('pages', [
    'name' => 'Front page',
    'url' => 'http://example.com/', // url is unique
    'visits' => 0,
], [
    'visits' => new \yii\db\Expression('visits + 1'),
], $params)->execute();
```

上面的代码将插入一个新的页面记录或自动增加访问计数器。

注意，上述的方法只是构建出语句， 你总是需要调用 [execute()](https://www.yiichina.com/doc/api/2.0/yii-db-command#execute()-detail) 来真正地执行它们。

### 引用表和列名称

当写与数据库无关的代码时，正确地引用表和列名称总是一件头疼的事， 因为不同的数据库有不同的名称引用规则，为了克服这个问题， 你可以使用下面由 Yii 提出的引用语法。

- `[[column name]]`: 使用两对方括号来将列名括起来;
- `{{table name}}`: 使用两对大括号来将表名括起来。

Yii DAO 将自动地根据数据库的具体语法来将这些结构转化为对应的 被引用的列或者表名称。 例如，

```php
// 在 MySQL 中执行该 SQL : SELECT COUNT(`id`) FROM `employee`
$count = Yii::$app->db->createCommand("SELECT COUNT([[id]]) FROM {{employee}}")
            ->queryScalar();
```

### 使用表前缀

如果你的数据库表名大多都拥有一个共同的前缀， 你可以使用 Yii DAO 所提供的表前缀功能。

首先，通过应用配置中的 [yii\db\Connection::$tablePrefix](https://www.yiichina.com/doc/api/2.0/yii-db-connection#$tablePrefix-detail) 属性来指定表前缀：

```php
return [
    // ...
    'components' => [
        // ...
        'db' => [
            // ...
            'tablePrefix' => 'tbl_',
        ],
    ],
];
```

接着在你的代码中，当你需要涉及到一张表名中包含该前缀的表时， 应使用语法 `{{%table_name}}`。百分号将被自动地替换为你在配置 DB 组件时指定的表前缀。 例如，

```php
// 在 MySQL 中执行该 SQL: SELECT COUNT(`id`) FROM `tbl_employee`
$count = Yii::$app->db->createCommand("SELECT COUNT([[id]]) FROM {{%employee}}")
            ->queryScalar();
```

### 执行事务

当顺序地执行多个相关的语句时， 你或许需要将它们包在一个事务中来保证数据库的完整性和一致性。 如果这些语句中的任何一个失败了， 数据库将回滚到这些语句执行前的状态。

下面的代码展示了一个使用事务的典型方法：

```php
Yii::$app->db->transaction(function($db) {
    $db->createCommand($sql1)->execute();
    $db->createCommand($sql2)->execute();
    // ... executing other SQL statements ...
});
```

上述代码等价于下面的代码，但是下面的代码给予了你对于错误处理代码的更多掌控：

```php
$db = Yii::$app->db;
$transaction = $db->beginTransaction();
try {
    $db->createCommand($sql1)->execute();
    $db->createCommand($sql2)->execute();
    // ... executing other SQL statements ...
    
    $transaction->commit();
} catch(\Exception $e) {
    $transaction->rollBack();
    throw $e;
} catch(\Throwable $e) {
    $transaction->rollBack();
    throw $e;
}
```

通过调用 [beginTransaction()](https://www.yiichina.com/doc/api/2.0/yii-db-connection#beginTransaction()-detail) 方法， 一个新事务开始了。 事务被表示为一个存储在 `$transaction` 变量中的 [yii\db\Transaction](https://www.yiichina.com/doc/api/2.0/yii-db-transaction) 对象。 然后，被执行的语句都被包含在一个 `try...catch...` 块中。 如果所有的语句都被成功地执行了， [commit()](https://www.yiichina.com/doc/api/2.0/yii-db-transaction#commit()-detail) 将被调用来提交这个事务。 否则， 如果异常被触发并被捕获， [rollBack()](https://www.yiichina.com/doc/api/2.0/yii-db-transaction#rollBack()-detail) 方法将被调用， 来回滚事务中失败语句之前所有语句所造成的改变。 `throw $e` 将重新抛出该异常， 就好像我们没有捕获它一样， 因此正常的错误处理程序将处理它。

### 指定隔离级别

Yii 也支持为你的事务设置[隔离级别](http://en.wikipedia.org/wiki/Isolation_(database_systems)#Isolation_levels)。默认情况下，当我们开启一个新事务， 它将使用你的数据库所设定的隔离级别。你也可以向下面这样重载默认的隔离级别，

```php
$isolationLevel = \yii\db\Transaction::REPEATABLE_READ;

Yii::$app->db->transaction(function ($db) {
    ....
}, $isolationLevel);
 
// or alternatively

$transaction = Yii::$app->db->beginTransaction($isolationLevel);
```

Yii 为四个最常用的隔离级别提供了常量：

- [yii\db\Transaction::READ_UNCOMMITTED](https://www.yiichina.com/doc/api/2.0/yii-db-transaction#READ_UNCOMMITTED-detail) - 最弱的隔离级别，脏读、不可重复读以及幻读都可能发生。
- [yii\db\Transaction::READ_COMMITTED](https://www.yiichina.com/doc/api/2.0/yii-db-transaction#READ_COMMITTED-detail) - 避免了脏读。
- [yii\db\Transaction::REPEATABLE_READ](https://www.yiichina.com/doc/api/2.0/yii-db-transaction#REPEATABLE_READ-detail) - 避免了脏读和不可重复读。
- [yii\db\Transaction::SERIALIZABLE](https://www.yiichina.com/doc/api/2.0/yii-db-transaction#SERIALIZABLE-detail) - 最强的隔离级别， 避免了上述所有的问题。

除了使用上述的常量来指定隔离级别，你还可以使用你的数据库所支持的具有有效语法的字符串。 比如，在 PostgreSQL 中，你可以使用 `SERIALIZABLE READ ONLY DEFERRABLE`。

请注意，一些数据库只允许为整个连接设置隔离级别， 即使你之后什么也没指定，后来的事务都将获得与之前相同的隔离级别。 使用此功能时，你需要为所有的事务明确地设置隔离级别来避免冲突的设置。 在本文写作之时，只有 MSSQL 和 SQLite 受这些限制的影响。

### 嵌套事务

如果你的数据库支持保存点，你可以像下面这样嵌套多个事务

```php
Yii::$app->db->transaction(function ($db) {
    // outer transaction
    
    $db->transaction(function ($db) {
        // inner transaction
    });
});
```

或者

```php
$db = Yii::$app->db;
$outerTransaction = $db->beginTransaction();
try {
    $db->createCommand($sql1)->execute();

    $innerTransaction = $db->beginTransaction();
    try {
        $db->createCommand($sql2)->execute();
        $innerTransaction->commit();
    } catch (\Exception $e) {
        $innerTransaction->rollBack();
        throw $e;
    } catch (\Throwable $e) {
        $innerTransaction->rollBack();
        throw $e;
    }

    $outerTransaction->commit();
} catch (\Exception $e) {
    $outerTransaction->rollBack();
    throw $e;
} catch (\Throwable $e) {
    $outerTransaction->rollBack();
    throw $e;
}
```

### 复制和读写分离

许多数据库支持[数据库复制](http://en.wikipedia.org/wiki/Replication_(computing)#Database_replication)来获得更好的数据库可用性， 以及更快的服务器响应时间。通过数据库复制功能， 数据从所谓的主服务器被复制到从服务器。所有的写和更新必须发生在主服务器上， 而读可以发生在从服务器上。

为了利用数据库复制并且完成读写分离， 你可以按照下面的方法来配置 [yii\db\Connection](https://www.yiichina.com/doc/api/2.0/yii-db-connection) 组件：

```php
[
    'class' => 'yii\db\Connection',

    // 主库的配置
    'dsn' => 'dsn for master server',
    'username' => 'master',
    'password' => '',

    // 从库的通用配置
    'slaveConfig' => [
        'username' => 'slave',
        'password' => '',
        'attributes' => [
            // 使用一个更小的连接超时
            PDO::ATTR_TIMEOUT => 10,
        ],
    ],

    // 从库的配置列表
    'slaves' => [
        ['dsn' => 'dsn for slave server 1'],
        ['dsn' => 'dsn for slave server 2'],
        ['dsn' => 'dsn for slave server 3'],
        ['dsn' => 'dsn for slave server 4'],
    ],
]
```

> **信息：** 通过调用 [yii\db\Command::execute()](https://www.yiichina.com/doc/api/2.0/yii-db-command#execute()-detail) 来执行的语句都被视为写操作， 而其他所有通过调用 [yii\db\Command](https://www.yiichina.com/doc/api/2.0/yii-db-command) 中任一 "query" 方法来执行的语句都被视为读操作。 你可以通过 `Yii::$app->db->slave` 来获取当前有效的从库连接。

`Connection` 组件支持从库间的负载均衡和失效备援， 当第一次执行读操作时，`Connection` 组件将随机地挑选出一个从库并尝试与之建立连接， 如果这个从库被发现为”挂掉的“，将尝试连接另一个从库。 如果没有一个从库是连接得上的，那么将试着连接到主库上。 通过配置 [server status cache](https://www.yiichina.com/doc/api/2.0/yii-db-connection#$serverStatusCache-detail)， 一个“挂掉的”服务器将会被记住，因此，在一个 yii\db\Connection::serverRetryInterval 内将不再试着连接该服务器。

> **信息：** 在上面的配置中， 每个从库都共同地指定了 10 秒的连接超时时间，这意味着，如果一个从库在 10 秒内不能被连接上，它将被视为“挂掉的”。 你可以根据你的实际环境来调整该参数

你也可以配置多主多从。例如，

```php
[
    'class' => 'yii\db\Connection',

    // 主库通用的配置
    'masterConfig' => [
        'username' => 'master',
        'password' => '',
        'attributes' => [
            // use a smaller connection timeout
            PDO::ATTR_TIMEOUT => 10,
        ],
    ],

    // 主库配置列表
    'masters' => [
        ['dsn' => 'dsn for master server 1'],
        ['dsn' => 'dsn for master server 2'],
    ],

    // 从库的通用配置
    'slaveConfig' => [
        'username' => 'slave',
        'password' => '',
        'attributes' => [
            // use a smaller connection timeout
            PDO::ATTR_TIMEOUT => 10,
        ],
    ],

    // 从库配置列表
    'slaves' => [
        ['dsn' => 'dsn for slave server 1'],
        ['dsn' => 'dsn for slave server 2'],
        ['dsn' => 'dsn for slave server 3'],
        ['dsn' => 'dsn for slave server 4'],
    ],
]
```

上述配置指定了两个主库和两个从库。 `Connection` 组件在主库之间，也支持如从库间般的负载均衡和失效备援。 唯一的差别是，如果没有主库可用，将抛出一个异常。

> **注意：** 当你使用 [masters](https://www.yiichina.com/doc/api/2.0/yii-db-connection#$masters-detail) 属性来配置一个或多个主库时， 所有其他指定数据库连接的属性 (例如 `dsn`, `username`, `password`) 与 `Connection` 对象本身将被忽略。

默认情况下，事务使用主库连接， 一个事务内，所有的数据库操作都将使用主库连接，例如，

```php
$db = Yii::$app->db;
// 在主库上启动事务
$transaction = $db->beginTransaction();

try {
    // 两个语句都是在主库上执行的
    $rows = $db->createCommand('SELECT * FROM user LIMIT 10')->queryAll();
    $db->createCommand("UPDATE user SET username='demo' WHERE id=1")->execute();

    $transaction->commit();
} catch(\Exception $e) {
    $transaction->rollBack();
    throw $e;
} catch(\Throwable $e) {
    $transaction->rollBack();
    throw $e;
}
```

如果你想在从库上开启事务，你应该明确地像下面这样做：

```php
$transaction = Yii::$app->db->slave->beginTransaction();
```

有时，你或许想要强制使用主库来执行读查询。 这可以通过 `useMaster()` 方法来完成：

```php
$rows = Yii::$app->db->useMaster(function ($db) {
    return $db->createCommand('SELECT * FROM user LIMIT 10')->queryAll();
});
```

你也可以明确地将 `Yii::$app->db->enableSlaves` 设置为 false 来将所有的读操作指向主库连接。

## 查询构建器

查询构建器建立在 [Database Access Objects](https://www.yiichina.com/doc/guide/2.0/db-dao) 基础之上，可让你创建 程序化的、DBMS无关的SQL语句。相比于原生的SQL语句，查询构建器可以帮你 写出可读性更强的SQL相关的代码，并生成安全性更强的SQL语句。

使用查询构建器通常包含以下两个步骤

1. 创建一个 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 对象来代表一条 SELECT SQL 语句的不同子句（例如 `SELECT`, `FROM`）。
2. 执行 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 的一个查询方法（例如：`all()`）从数据库当中检索数据。

如下所示代码是查询构造器的一个典型用法

```php
$rows = (new \yii\db\Query())
    ->select(['id', 'email'])
    ->from('user')
    ->where(['last_name' => 'Smith'])
    ->limit(10)
    ->all();
```

上面的代码将会生成并执行如下的SQL语句，其中 `:last_name` 参数绑定了 字符串 `'Smith'`。

```php
SELECT `id`, `email` 
FROM `user`
WHERE `last_name` = :last_name
LIMIT 10
```

> **提示：** 你平时更多的时候会使用 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 而不是 [yii\db\QueryBuilder](https://www.yiichina.com/doc/api/2.0/yii-db-querybuilder)。 当你调用其中一个查询方法时，后者将会被前者隐式的调用。[yii\db\QueryBuilder](https://www.yiichina.com/doc/api/2.0/yii-db-querybuilder)主要负责将 DBMS 不相关的 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 对象转换成 DBMS 相关的 SQL 语句（例如， 以不同的方式引用表或字段名称）。

### 创建查询

为了创建一个 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 对象，你需要调用不同的查询构建方法来代表SQL语句的不同子句。 这些方法的名称集成了在SQL语句相应子句中使用的关键字。例如，为了指定 SQL 语句当中的 `FROM` 子句，你应该调用 `from()` 方法。所有的查询构建器方法返回的是查询对象本身， 也就是说，你可以把多个方法的调用串联起来。

接下来，我们会对这些查询构建器方法进行一一讲解：

#### select()

[select()](https://www.yiichina.com/doc/api/2.0/yii-db-query#select()-detail) 方法用来指定 SQL 语句当中的 `SELECT` 子句。 你可以像下面的例子一样使用一个数组或者字符串来定义需要查询的字段。当 SQL 语句 是由查询对象生成的时候，被查询的字段名称将会自动的被引号括起来

```php
$query->select(['id', 'email']);

// 等同于：

$query->select('id, email');
```

就像写原生 SQL 语句一样，被选取的字段可以包含表前缀，以及/或者字段别名。 例如：

```php
$query->select(['user.id AS user_id', 'email']);

// 等同于：

$query->select('user.id AS user_id, email');
```

如果使用数组格式来指定字段，你可以使用数组的键值来表示字段的别名。 例如，上面的代码可以被重写为如下形式

```php
$query->select(['user_id' => 'user.id', 'email']);
```

如果你在组建查询时没有调用 [select()](https://www.yiichina.com/doc/api/2.0/yii-db-query#select()-detail) 方法，那么选择的将是 `'*'` ， 也即选取的是所有的字段。

除了字段名称以外，你还可以选择数据库的表达式。当你使用到包含逗号的数据库表达式的时候， 你必须使用数组的格式，以避免自动的错误的引号添加。例如：

```php
$query->select(["CONCAT(first_name, ' ', last_name) AS full_name", 'email']); 
```

与所有涉及原始 SQL 的地方一样，当在 select 中编写 DB 表达式时，可以对表名和列名使用 [与 DBMS 无关的引用语法](https://www.yiichina.com/doc/guide/2.0/db-dao#quoting-table-and-column-names)。

从 2.0.1 的版本开始你就可以使用子查询了。在定义每一个子查询的时候， 你应该使用 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 对象。例如：

```php
$subQuery = (new Query())->select('COUNT(*)')->from('user');

// SELECT `id`, (SELECT COUNT(*) FROM `user`) AS `count` FROM `post`
$query = (new Query())->select(['id', 'count' => $subQuery])->from('post');
```

你应该调用 [distinct()](https://www.yiichina.com/doc/api/2.0/yii-db-query#distinct()-detail) 方法来去除重复行，如下所示

```php
// SELECT DISTINCT `user_id` ...
$query->select('user_id')->distinct();
```

你可以调用 [addSelect()](https://www.yiichina.com/doc/api/2.0/yii-db-query#addSelect()-detail) 方法来选取附加字段，例如：

```php
$query->select(['id', 'username'])
    ->addSelect(['email']);
```

#### from()

[from()](https://www.yiichina.com/doc/api/2.0/yii-db-query#from()-detail) 方法指定了 SQL 语句当中的 `FROM` 子句。例如：

```php
// SELECT * FROM `user`
$query->from('user');
```

你可以通过字符串或者数组的形式来定义被查询的表名称。就像你写原生的 SQL 语句一样， 表名称里面可包含数据库前缀，以及/或者表别名。例如：

```php
$query->from(['public.user u', 'public.post p']);

// 等同于：

$query->from('public.user u, public.post p');
```

如果你使用的是数组的格式，那么你同样可以用数组的键值来定义表别名，如下所示：

```php
$query->from(['u' => 'public.user', 'p' => 'public.post']);
```

除了表名以外，你还可以从子查询中再次查询，这里的子查询是由 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 创建的对象。 例如：

```php
$subQuery = (new Query())->select('id')->from('user')->where('status=1');

// SELECT * FROM (SELECT `id` FROM `user` WHERE status=1) u 
$query->from(['u' => $subQuery]);
```

#### 前缀

`from` 还可以应用默认的 [tablePrefix](https://www.yiichina.com/doc/api/2.0/yii-db-connection#$tablePrefix-detail) 前缀，实现细节请参考 [“数据库访问对象指南”的“Quoting Tables”章节](https://www.yiichina.com/doc/guide/2.0/guide-db-dao.html#quoting-table-and-column-names).

#### where()

[where()](https://www.yiichina.com/doc/api/2.0/yii-db-query#where()-detail) 方法定义了 SQL 语句当中的 `WHERE` 子句。 你可以使用如下四种格式来定义 `WHERE` 条件：

- 字符串格式，例如：`'status=1'`
- 哈希格式，例如： `['status' => 1, 'type' => 2]`
- 操作符格式，例如：`['like', 'name', 'test']`
- 对象格式，例如：`new LikeCondition('name', 'LIKE', 'test')`

##### 字符串格式

在定义非常简单的查询条件的时候，字符串格式是最合适的。 它看起来和原生 SQL 语句差不多。例如：

```php
$query->where('status=1');

// 或使用参数绑定来绑定动态参数值
$query->where('status=:status', [':status' => $status]);

// 原生 SQL 在日期字段上使用 MySQL YEAR() 函数
$query->where('YEAR(somedate) = 2015');
```

千万不要像如下的例子一样直接在条件语句当中嵌入变量，特别是当这些变量来源于终端用户输入的时候， 因为这样我们的软件将很容易受到 SQL 注入的攻击。

```php
// 危险！千万别这样干，除非你非常的确定 $status 是一个整型数值。
$query->where("status=$status");
```

当使用 `参数绑定` 的时候，你可以调用 [params()](https://www.yiichina.com/doc/api/2.0/yii-db-query#params()-detail) 或者 [addParams()](https://www.yiichina.com/doc/api/2.0/yii-db-query#addParams()-detail) 方法 来分别绑定不同的参数。

```php
$query->where('status=:status')
    ->addParams([':status' => $status]);
```

与涉及原生 SQL 的所有地方一样，在以字符串格式写入条件时，可以对表名和列名使用 [与 DBMS 无关的引用语法](https://www.yiichina.com/doc/guide/2.0/db-dao#quoting-table-and-column-names)。

##### 哈希格式

哈希格式最适合用来指定多个 `AND` 串联起来的简单的"等于断言"子条件。 它是以数组的形式来书写的，数组的键表示字段的名称，而数组的值则表示 这个字段需要匹配的值。例如：

```php
// ...WHERE (`status` = 10) AND (`type` IS NULL) AND (`id` IN (4, 8, 15))
$query->where([
    'status' => 10,
    'type' => null,
    'id' => [4, 8, 15],
]);
```

就像你所看到的一样，查询构建器非常的智能，能恰当地处理数值当中的空值和数组。

你也可以像下面那样在子查询当中使用哈希格式：

```php
$userQuery = (new Query())->select('id')->from('user');

// ...WHERE `id` IN (SELECT `id` FROM `user`)
$query->where(['id' => $userQuery]);
```

使用哈希格式，Yii 在内部对相应的值进行参数绑定，与 [字符串格式](https://www.yiichina.com/doc/guide/2.0/db-query-builder#string-format) 相比， 此处你不需要手动添加参数绑定。但请注意，Yii 不会帮你转义列名，所以如果你 从用户端获得的变量作为列名而没有进行任何额外的检查，对于 SQL 注入攻击， 你的程序将变得很脆弱。为了保证应用程序的安全，请不要将变量用作列名 或者你必须用白名单过滤变量。如果你实在需要从用户获取列名，请阅读 [过滤数据](https://www.yiichina.com/doc/guide/2.0/output-data-widgets#filtering-data) 章节。例如，以下代码易受攻击：

```php
// 易受攻击的代码：
$column = $request->get('column');
$value = $request->get('value');
$query->where([$column => $value]);
// $value 是安全的，但是 $column 名不会被转义处理！
```

##### 操作符格式

操作符格式允许你指定类程序风格的任意条件语句，如下所示：

```
[操作符, 操作数1, 操作数2, ...]
```

其中每个操作数可以是字符串格式、哈希格式或者嵌套的操作符格式， 而操作符可以是如下列表中的一个：

- `and`：操作数会被 `AND` 关键字串联起来。例如，`['and', 'id=1', 'id=2']` 将会生成 `id=1 AND id=2`。如果操作数是一个数组，它也会按上述规则转换成 字符串。例如，`['and', 'type=1', ['or', 'id=1', 'id=2']]` 将会生成 `type=1 AND (id=1 OR id=2)`。 这个方法不会自动加引号或者转义。
- `or`：用法和 `and` 操作符类似，这里就不再赘述。
- `not`：只需要操作数 1，它将包含在 `NOT()` 中。例如，`['not'，'id = 1']` 将生成 `NOT (id=1)`。操作数 1 也可以是个描述多个表达式的数组。例如 `['not', ['status' => 'draft', 'name' => 'example']]` 将生成 `NOT ((status='draft') AND (name='example'))`。
- `between`：第一个操作数为字段名称，第二个和第三个操作数代表的是这个字段 的取值范围。例如，`['between', 'id', 1, 10]` 将会生成 `id BETWEEN 1 AND 10`。 如果你需要建立一个值在两列之间的查询条件（比如 `11 BETWEEN min_id AND max_id`）， 你应该使用 [BetweenColumnsCondition](https://www.yiichina.com/doc/api/2.0/yii-db-conditions-betweencolumnscondition)。 请参阅 [条件-对象格式](https://www.yiichina.com/doc/guide/2.0/db-query-builder#object-format) 一章以了解有关条件的对象定义的更多信息。
- `not between`：与 `between` 类似，除了 `BETWEEN` 被 `NOT BETWEEN` 替换 在生成条件时。
- `in`：第一个操作数应为字段名称或者 DB 表达式。第二个操作符既可以是一个数组， 也可以是一个 `Query` 对象。它会转换成`IN` 条件语句。如果第二个操作数是一个 数组，那么它代表的是字段或 DB 表达式的取值范围。如果第二个操作数是 `Query` 对象，那么这个子查询的结果集将会作为第一个操作符的字段或者 DB 表达式的取值范围。 例如， `['in', 'id', [1, 2, 3]]` 将生成 `id IN (1, 2, 3)`。 该方法将正确地为字段名加引号以及为取值范围转义。`in` 操作符还支持组合字段，此时， 操作数1应该是一个字段名数组，而操作数2应该是一个数组或者 `Query` 对象， 代表这些字段的取值范围。
- `not in`：用法和 `in` 操作符类似，这里就不再赘述。
- `like`：第一个操作数应为一个字段名称或 DB 表达式， 第二个操作数可以使字符串或数组， 代表第一个操作数需要模糊查询的值。比如，`['like', 'name', 'tester']` 会生成 `name LIKE '%tester%'`。 如果范围值是一个数组，那么将会生成用 `AND` 串联起来的 多个 `like` 语句。例如，`['like', 'name', ['test', 'sample']]` 将会生成 `name LIKE '%test%' AND name LIKE '%sample%'`。 你也可以提供第三个可选的操作数来指定应该如何转义数值当中的特殊字符。 该操作数是一个从需要被转义的特殊字符到转义副本的数组映射。 如果没有提供这个操作数，将会使用默认的转义映射。如果需要禁用转义的功能， 只需要将参数设置为 `false` 或者传入一个空数组即可。需要注意的是， 当使用转义映射（又或者没有提供第三个操作数的时候），第二个操作数的值的前后 将会被加上百分号。

>  当使用 PostgreSQL 的时候你还可以使用 [`ilike`](http://www.postgresql.org/docs/8.3/static/functions-matching.html#FUNCTIONS-LIKE)， 该方法对大小写不敏感

- `or like`：用法和 `like` 操作符类似，区别在于当第二个操作数为数组时， 会使用 `OR` 来串联多个 `LIKE` 条件语句。
- `not like`：用法和 `like` 操作符类似，区别在于会使用 `NOT LIKE` 来生成条件语句。
- `or not like`：用法和 `not like` 操作符类似，区别在于会使用 `OR` 来串联多个 `NOT LIKE` 条件语句。
- `exists`：需要一个操作数，该操作数必须是代表子查询 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 的一个实例， 它将会构建一个 `EXISTS (sub-query)` 表达式。
- `not exists`：用法和 `exists` 操作符类似，它将创建一个 `NOT EXISTS (sub-query)` 表达式。
- `>`，`<=` 或者其他包含两个操作数的合法 DB 操作符：第一个操作数必须为字段的名称， 而第二个操作数则应为一个值。例如，`['>', 'age', 10]` 将会生成 `age>10`。

使用操作符格式，Yii 在内部对相应的值进行参数绑定，因此与 [字符串格式](https://www.yiichina.com/doc/guide/2.0/db-query-builder#string-format) 相比， 此处你不需要手动添加参数。但请注意，Yii 不会帮你转义列名，所以如果你 从用户端获得的变量作为列名而没有进行任何额外的检查，对于 SQL 注入攻击， 你的程序将变得很脆弱。为了保证应用程序的安全，请不要将变量用作列名 或者你必须用白名单过滤变量。如果你实在需要从用户获取列名，请阅读 [过滤数据](https://www.yiichina.com/doc/guide/2.0/output-data-widgets#filtering-data) 章节。例如，以下代码易受攻击：

```php
// 易受攻击的代码：
$column = $request->get('column');
$value = $request->get('value);
$query->where(['=', $column, $value]);
// $value 是安全的，但是 $column 名不会被转义处理！
```

##### 对象格式

对象格式自 2.0.14 开始提供，是定义条件的最强大和最复杂的方法。 如果你要在查询构建器上构建自己的抽象方法或者如果你要实现自己的复杂条件， 你需要它（Object Form）

条件类的实例是不可变的。他们唯一的用途是存储条件数据 并为条件构建器提供 getters 属性。条件构建器是一个包含转换逻辑的类， 它将存储的条件数据转换为 SQL 表达式。

在内部，上面描述的格式在构建 SQL 之前被隐式转换为对象格式， 因此可以在单一条件语句下组合适合的格式：

```php
$query->andWhere(new OrCondition([
    new InCondition('type', 'in', $types),
    ['like', 'name', '%good%'],
    'disabled=false'
]))
```

操作符格式与对象格式的对应关系是在 [QueryBuilder::conditionClasses](https://www.yiichina.com/doc/api/2.0/yii-db-querybuilder#$conditionClasses-detail) 属性中定义， 这里列举一些比较有代表性的映射关系：

- `AND`, `OR` -> `yii\db\conditions\ConjunctionCondition`
- `NOT` -> `yii\db\conditions\NotCondition`
- `IN`, `NOT IN` -> `yii\db\conditions\InCondition`
- `BETWEEN`, `NOT BETWEEN` -> `yii\db\conditions\BetweenCondition`

等等

使用对象格式可以定义自己的条件集，并且可以更容易维护别人定义的条件集。（注：这里是说对象比数组更可靠） 更多细节请参考 [Adding Custom Conditions and Expressions](https://www.yiichina.com/doc/guide/2.0/db-query-builder#adding-custom-conditions-and-expressions) 章节。

##### 附加条件

你可以使用 [andWhere()](https://www.yiichina.com/doc/api/2.0/yii-db-query#andWhere()-detail) 或者 [orWhere()](https://www.yiichina.com/doc/api/2.0/yii-db-query#orWhere()-detail) 在原有条件的基础上 附加额外的条件。你可以多次调用这些方法来分别追加不同的条件。 例如，

```php
$status = 10;
$search = 'yii';

$query->where(['status' => $status]);

if (!empty($search)) {
    $query->andWhere(['like', 'title', $search]);
}
```

如果 `$search` 不为空，那么将会生成如下 SQL 语句：

```php
... WHERE (`status` = 10) AND (`title` LIKE '%yii%')
```

##### 过滤条件

当 `WHERE` 条件来自于用户的输入时，你通常需要忽略用户输入的空值。 例如，在一个可以通过用户名或者邮箱搜索的表单当中，用户名或者邮箱 输入框没有输入任何东西，这种情况下你想要忽略掉对应的搜索条件， 那么你就可以使用 [filterWhere()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#filterWhere()-detail) 方法来实现这个目的：

```php
// $username 和 $email 来自于用户的输入
$query->filterWhere([
    'username' => $username,
    'email' => $email,		
]);
```

[filterWhere()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#filterWhere()-detail) 和 [where()](https://www.yiichina.com/doc/api/2.0/yii-db-query#where()-detail) 唯一的不同就在于，前者 将忽略在条件当中的[hash format](https://www.yiichina.com/doc/guide/2.0/db-query-builder#hash-format)的空值。所以如果 `$email` 为空而 `$username` 不为空，那么上面的代码最终将生产如下 SQL `...WHERE username=:username`。

> **提示：** 当一个值为 `null`、空数组、空字符串或者一个只包含空格的字符串时，那么它将被判定为空值。

类似于 [andWhere()](https://www.yiichina.com/doc/api/2.0/yii-db-query#andWhere()-detail) 和 [orWhere()](https://www.yiichina.com/doc/api/2.0/yii-db-query#orWhere()-detail)， 你可以使用 [andFilterWhere()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#andFilterWhere()-detail) 和 [orFilterWhere()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#orFilterWhere()-detail) 方法 来追加额外的过滤条件。

此外，[yii\db\Query::andFilterCompare()](https://www.yiichina.com/doc/api/2.0/yii-db-query#andFilterCompare()-detail) 可以根据值中的内容智能地确定运算符：

```php
$query->andFilterCompare('name', 'John Doe');
$query->andFilterCompare('rating', '>9');
$query->andFilterCompare('value', '<=100');
```

您还可以显式指定运算符：

```php
$query->andFilterCompare('name', 'Doe', 'like');
```

Yii 自 2.0.11 版起 ，提供了 `HAVING` 条件的一些构建方法：

- [filterHaving()](https://www.yiichina.com/doc/api/2.0/yii-db-query#filterHaving()-detail)
- [andFilterHaving()](https://www.yiichina.com/doc/api/2.0/yii-db-query#andFilterHaving()-detail)
- [orFilterHaving()](https://www.yiichina.com/doc/api/2.0/yii-db-query#orFilterHaving()-detail)

#### orderBy()

[orderBy()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#orderBy()-detail) 方法是用来指定 SQL 语句当中的 `ORDER BY` 子句的。例如，

```php
// ... ORDER BY `id` ASC, `name` DESC
$query->orderBy([
    'id' => SORT_ASC,
    'name' => SORT_DESC,
]);
```

如上所示，数组当中的键指代的是字段名称，而数组当中的值则表示的是排序的方式。 PHP 的常量 `SORT_ASC` 指的是升序排列，`SORT_DESC` 指的则是降序排列。

如果 `ORDER BY` 仅仅包含简单的字段名称，你可以使用字符串来声明它， 就像写原生的 SQL 语句一样。例如，

```php
$query->orderBy('id ASC, name DESC');
```

你可以调用 [addOrderBy()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#addOrderBy()-detail) 来为 `ORDER BY` 片断添加额外的子句。 例如，

```php
$query->orderBy('id ASC')
    ->addOrderBy('name DESC');
```

#### groupBy()

[groupBy()](https://www.yiichina.com/doc/api/2.0/yii-db-query#groupBy()-detail) 方法是用来指定 SQL 语句当中的 `GROUP BY` 片断的。例如，

```php
// ... GROUP BY `id`, `status`
$query->groupBy(['id', 'status']);
```

如果 `GROUP BY` 仅仅包含简单的字段名称，你可以使用字符串来声明它， 就像写原生的 SQL 语句一样。例如，

```php
$query->groupBy('id, status');
```

> **注意：** 当 `GROUP BY` 语句包含一些 DB 表达式的时候，你应该使用数组的格式。

你可以调用 [addOrderBy()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#addOrderBy()-detail) 来为 `GROUP BY` 子句添加额外的字段。例如，

```php
$query->groupBy(['id', 'status'])
    ->addGroupBy('age');
```

#### having()

[having()](https://www.yiichina.com/doc/api/2.0/yii-db-query#having()-detail) 方法是用来指定 SQL 语句当中的 `HAVING` 子句。它带有一个条件， 和 [where()](https://www.yiichina.com/doc/guide/2.0/db-query-builder#where) 中指定条件的方法一样。例如，

```php
// ... HAVING `status` = 1
$query->having(['status' => 1]);
```

请查阅 [where()](https://www.yiichina.com/doc/guide/2.0/db-query-builder#where) 的文档来获取更多有关于如何指定一个条件的细节。

你可以调用 [andHaving()](https://www.yiichina.com/doc/api/2.0/yii-db-query#andHaving()-detail) 或者 [orHaving()](https://www.yiichina.com/doc/api/2.0/yii-db-query#orHaving()-detail) 方法来为 `HAVING` 子句追加额外的条件，例如，

```php
// ... HAVING (`status` = 1) AND (`age` > 30)
$query->having(['status' => 1])
    ->andHaving(['>', 'age', 30]);
```

#### limit()和offset()

[limit()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#limit()-detail) 和 [offset()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#offset()-detail) 是用来指定 SQL 语句当中 的 `LIMIT` 和 `OFFSET` 子句的。例如，

```php
// ... LIMIT 10 OFFSET 20
$query->limit(10)->offset(20);
```

如果你指定了一个无效的 limit 或者 offset（例如，一个负数），那么它将会被忽略掉。

> **提示：** 在不支持 `LIMIT` 和 `OFFSET` 的 DBMS 中（例如，MSSQL）， 查询构建器将生成一条模拟 `LIMIT`/`OFFSET` 行为的 SQL 语句。

#### joint()

[join()](https://www.yiichina.com/doc/api/2.0/yii-db-query#join()-detail) 是用来指定 SQL 语句当中的 `JOIN` 子句的。例如

```php
// ... LEFT JOIN `post` ON `post`.`user_id` = `user`.`id`
$query->join('LEFT JOIN', 'post', 'post.user_id = user.id');
```

[join()](https://www.yiichina.com/doc/api/2.0/yii-db-query#join()-detail) 带有四个参数：

- `$type`：连接类型，例如，`'INNER JOIN'`，`'LEFT JOIN'`。
- `$table`：将要连接的表名称。
- `$on`：可选的，连接条件，即 `ON` 片段。有关指定条件的详细信息，请参阅 [where()](https://www.yiichina.com/doc/guide/2.0/db-query-builder#where)。 请注意，数组语法 **不能** 用于指定基于列的条件， 例如，`['user.id' => 'comment.userId']` 将导致用户 id 必须等于字符串 `'comment.userId'` 的情况。您应该使用字符串语法，并将条件指定为 `'user.id = comment.userId'`。
- `$params`：可选参数，与连接条件绑定的参数。

你可以分别调用如下的快捷方法来指定 `INNER JOIN`, `LEFT JOIN` 和 `RIGHT JOIN`。

- [innerJoin()](https://www.yiichina.com/doc/api/2.0/yii-db-query#innerJoin()-detail)
- [leftJoin()](https://www.yiichina.com/doc/api/2.0/yii-db-query#leftJoin()-detail)
- [rightJoin()](https://www.yiichina.com/doc/api/2.0/yii-db-query#rightJoin()-detail)

例如

```php
$query->leftJoin('post', 'post.user_id = user.id');
```

可以通过多次调用如上所述的连接方法来连接多张表，每连接一张表调用一次。

除了连接表以外，你还可以连接子查询。方法如下，将需要被连接的子查询指定 为一个 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 对象，例如，

```php
$subQuery = (new \yii\db\Query())->from('post');
$query->leftJoin(['u' => $subQuery], 'u.id = author_id');
```

在这个例子当中，你应该将子查询放到一个数组当中，而数组当中的键，则为这个子查询的别名。

#### union()

[union()](https://www.yiichina.com/doc/api/2.0/yii-db-query#union()-detail) 方法是用来指定 SQL 语句当中的 `UNION` 子句的。例如，

```php
$query1 = (new \yii\db\Query())
    ->select("id, category_id AS type, name")
    ->from('post')
    ->limit(10);

$query2 = (new \yii\db\Query())
    ->select('id, type, name')
    ->from('user')
    ->limit(10);

$query1->union($query2);
```

你可以通过多次调用 [union()](https://www.yiichina.com/doc/api/2.0/yii-db-query#union()-detail) 方法来追加更多的 `UNION` 子句。

### 查询方法

[yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 提供了一整套的用于不同查询目的的方法。

- [all()](https://www.yiichina.com/doc/api/2.0/yii-db-query#all()-detail)：将返回一个由行组成的数组，每一行是一个由名称和值构成的关联数组（译者注：省略键的数组称为索引数组）。
- [one()](https://www.yiichina.com/doc/api/2.0/yii-db-query#one()-detail)：返回结果集的第一行。
- [column()](https://www.yiichina.com/doc/api/2.0/yii-db-query#column()-detail)：返回结果集的第一列。
- [scalar()](https://www.yiichina.com/doc/api/2.0/yii-db-query#scalar()-detail)：返回结果集的第一行第一列的标量值。
- [exists()](https://www.yiichina.com/doc/api/2.0/yii-db-query#exists()-detail)：返回一个表示该查询是否包含结果集的值。
- [count()](https://www.yiichina.com/doc/api/2.0/yii-db-query#count()-detail)：返回 `COUNT` 查询的结果。
- 其它集合查询方法：包括 [sum($q)](https://www.yiichina.com/doc/api/2.0/yii-db-query#sum()-detail), [average($q)](https://www.yiichina.com/doc/api/2.0/yii-db-query#average()-detail), [max($q)](https://www.yiichina.com/doc/api/2.0/yii-db-query#max()-detail), [min($q)](https://www.yiichina.com/doc/api/2.0/yii-db-query#min()-detail) 等。`$q` 是一个必选参数， 既可以是一个字段名称，又可以是一个 DB 表达式。

```
// SELECT `id`, `email` FROM `user`
$rows = (new \yii\db\Query())
    ->select(['id', 'email'])
    ->from('user')
    ->all();
    
// SELECT * FROM `user` WHERE `username` LIKE `%test%`
$row = (new \yii\db\Query())
    ->from('user')
    ->where(['like', 'username', 'test'])
    ->one();
```

> **注意：** [one()](https://www.yiichina.com/doc/api/2.0/yii-db-query#one()-detail) 方法只返回查询结果当中的第一条数据， 条件语句中不会加上 `LIMIT 1` 条件。如果你清楚的知道查询将会只返回一行或几行数据 （例如， 如果你是通过某些主键来查询的），这很好也提倡这样做。但是，如果查询结果 有机会返回大量的数据时，那么你应该显示调用 `limit(1)` 方法，以改善性能。 例如，`(new \yii\db\Query())->from('user')->limit(1)->one()`。

所有的这些查询方法都有一个可选的参数 `$db`, 该参数指代的是 [DB connection](https://www.yiichina.com/doc/api/2.0/yii-db-connection)， 执行一个 DB 查询时会用到。如果你省略了这个参数，那么 `db` [application component](https://www.yiichina.com/doc/guide/2.0/structure-application-components) 将会被用作 默认的 DB 连接。 如下是另外一个使用 `count()` 查询的例子：

```php
// 执行 SQL: SELECT COUNT(*) FROM `user` WHERE `last_name`=:last_name
$count = (new \yii\db\Query())
    ->from('user')
    ->where(['last_name' => 'Smith'])
    ->count();
```

当你调用 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 当中的一个查询方法的时候，实际上内在的运作机制如下：

- 在当前 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 的构造基础之上，调用 [yii\db\QueryBuilder](https://www.yiichina.com/doc/api/2.0/yii-db-querybuilder) 来生成一条 SQL 语句；
- 利用生成的 SQL 语句创建一个 [yii\db\Command](https://www.yiichina.com/doc/api/2.0/yii-db-command) 对象；
- 调用 [yii\db\Command](https://www.yiichina.com/doc/api/2.0/yii-db-command) 的查询方法（例如，`queryAll()`）来执行这条 SQL 语句，并检索数据。

有时候，你也许想要测试或者使用一个由 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query) 对象创建的 SQL 语句。 你可以使用以下的代码来达到目的：

```php
$command = (new \yii\db\Query())
    ->select(['id', 'email'])
    ->from('user')
    ->where(['last_name' => 'Smith'])
    ->limit(10)
    ->createCommand();
    
// 打印 SQL 语句
echo $command->sql;
// 打印被绑定的参数
print_r($command->params);

// 返回查询结果的所有行
$rows = $command->queryAll();
```

### 索引查询结果

当你在调用 [all()](https://www.yiichina.com/doc/api/2.0/yii-db-query#all()-detail) 方法时，它将返回一个以连续的整型数值为索引的数组。 而有时候你可能希望使用一个特定的字段或者表达式的值来作为索引结果集数组。那么你可以在调用 [all()](https://www.yiichina.com/doc/api/2.0/yii-db-query#all()-detail) 之前使用 [indexBy()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#indexBy()-detail) 方法来达到这个目的。 例如，

```php
// 返回 [100 => ['id' => 100, 'username' => '...', ...], 101 => [...], 103 => [...], ...]
$query = (new \yii\db\Query())
    ->from('user')
    ->limit(10)
    ->indexBy('id')
    ->all();
```

如需使用表达式的值做为索引，那么只需要传递一个匿名函数给 [indexBy()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#indexBy()-detail) 方法即可：

```php
$query = (new \yii\db\Query())
    ->from('user')
    ->indexBy(function ($row) {
        return $row['id'] . $row['username'];
    })->all();
```

该匿名函数将带有一个包含了当前行的数据的 `$row` 参数，并且返回用作当前行索引的 标量值（译者注：就是简单的数值或者字符串，而不是其他复杂结构，例如数组）。

> **注意：** 与 [groupBy()](https://www.yiichina.com/doc/api/2.0/yii-db-query#groupBy()-detail) 或者 [orderBy()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#orderBy()-detail) 等查询方法不同， 他们将转换为 SQL 查询语句的一部分，而这个方法（indexBy）在从数据库取回数据后才生效执行的。 这意味着只能使用那些在你的 SELECT 查询中的列名。 此外，你用表名连接取列名的时候，比如 `customer.id`，结果中将只包含 `id` 列，因此你必须调用 `->indexBy('id')` 不要带表名前缀。

### 批处理查询

当需要处理大数据的时候，像 [yii\db\Query::all()](https://www.yiichina.com/doc/api/2.0/yii-db-query#all()-detail) 这样的方法就不太合适了， 因为它们会把所有查询的数据都读取到客户端内存上。为了解决这个问题， Yii 提供了批处理查询的支持。服务端先保存查询结果，然后客户端使用游标（cursor） 每次迭代出固定的一批结果集回来。

> **警告：** MySQL 批处理查询的实现存在已知的局限性和变通方法。见下文。

批处理查询的用法如下：

```php
use yii\db\Query;

$query = (new Query())
    ->from('user')
    ->orderBy('id');

foreach ($query->batch() as $users) {
    // $users 是一个包含100条或小于100条用户表数据的数组
}

// or to iterate the row one by one
foreach ($query->each() as $user) {
    // 数据从服务端中以 100 个为一组批量获取，
    // 但是 $user 代表 user 表里的一行数据
}
```

[yii\db\Query::batch()](https://www.yiichina.com/doc/api/2.0/yii-db-query#batch()-detail) 和 [yii\db\Query::each()](https://www.yiichina.com/doc/api/2.0/yii-db-query#each()-detail) 方法将会返回一个实现了`Iterator` 接口 [yii\db\BatchQueryResult](https://www.yiichina.com/doc/api/2.0/yii-db-batchqueryresult) 的对象，可以用在 `foreach` 结构当中使用。在第一次迭代取数据的时候， 数据库会执行一次 SQL 查询，然后在剩下的迭代中，将直接从结果集中批量获取数据。默认情况下， 一批的大小为 100，也就意味着一批获取的数据是 100 行。你可以通过给 `batch()` 或者 `each()` 方法的第一个参数传值来改变每批行数的大小。

相对于 [yii\db\Query::all()](https://www.yiichina.com/doc/api/2.0/yii-db-query#all()-detail) 方法，批处理查询每次只读取 100 行的数据到内存。

如果你通过 [yii\db\Query::indexBy()](https://www.yiichina.com/doc/api/2.0/yii-db-querytrait#indexBy()-detail) 方法为查询结果指定了索引字段， 那么批处理查询将仍然保持相对应的索引方案， 例如，

```php
$query = (new \yii\db\Query())
    ->from('user')
    ->indexBy('username');

foreach ($query->batch() as $users) {
    // $users 的 “username” 字段将会成为索引
}

foreach ($query->each() as $username => $user) {
    // ...
}
```

### MySQL中批量查询的局限性

MySQL 是通过 PDO 驱动库实现批量查询的。默认情况下，MySQL 查询是 [`带缓存的`](http://php.net/manual/en/mysqlinfo.concepts.buffering.php)， 这违背了使用游标（cursor）获取数据的目的， 因为它不阻止驱动程序将整个结果集加载到客户端的内存中

> **注意：** 当使用 `libmysqlclient` 时（PHP5 的标配），计算 PHP 的内存限制时，用于数据结果集的内存不会计算在内。 看上去批量查询是正确运行的，实际上整个数据集都被加载到了客户端的内存中， 而且这个使用量可能还会再增长。

要禁用缓存并减少客户端内存的需求量，PDO 连接属性 `PDO::MYSQL_ATTR_USE_BUFFERED_QUERY` 必须设置为 `false`。 这样，直到整个数据集被处理完毕前，通过此连接是无法创建其他查询的。 这样的操作可能会阻碍 `ActiveRecord` 执行表结构查询。 如果这不构成问题（表结构已被缓存过了）， 我们可以通过切换原本的连接到非缓存模式，然后在批量查询完成后再切换回来。

```php
Yii::$app->db->pdo->setAttribute(\PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, false);

// 执行批量查询

Yii::$app->db->pdo->setAttribute(\PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, true);
```

> **注意：** 对于 MyISAM，在执行批量查询的过程中，表可能将被锁， 将延迟或拒绝其他连接的写入操作。 当使用非缓存查询时，尽量缩短游标打开的时间。

如果表结构没有被缓存，或在批量查询被处理过程中需要执行其他查询， 你可以创建一个单独的非缓存链接到数据库：

```php
$unbufferedDb = new \yii\db\Connection([
    'dsn' => Yii::$app->db->dsn,
    'username' => Yii::$app->db->username,
    'password' => Yii::$app->db->password,
    'charset' => Yii::$app->db->charset,
]);
$unbufferedDb->open();
$unbufferedDb->pdo->setAttribute(\PDO::MYSQL_ATTR_USE_BUFFERED_QUERY, false);
```

如果你除了 `PDO::MYSQL_ATTR_USE_BUFFERED_QUERY` 是 `false` 之外， 要确保 `$unbufferedDb` 拥有和原来缓存 `$db` 完全一样的属性， 请参阅[实现 `$db` 的深度拷贝](https://github.com/yiisoft/yii2/issues/8420#issuecomment-301423833)， 手动方法将它设置为 false 即可。

然后使用此连接正常创建查询，新连接用于运行批量查询， 逐条或批量进行结果处理：

```php
// 获取 1000 为一组的批量数据
foreach ($query->batch(1000, $unbufferedDb) as $users) {
    // ...
}


// 每次从服务端批量获取1000个数据，但是逐个遍历进行处理
foreach ($query->each(1000, $unbufferedDb) as $user) {
    // ...
}
```

当结果集已处理完毕不再需要连接时，可以关闭它：

```php
$unbufferedDb->close();
```

> **注意：** 非缓存查询在 PHP 端使用更少的缓存，但会增加 MySQL 服务器端的负载。 建议您使用生产实践设计自己的代码以获取额外的海量数据，[例如，将数字键分段，使用非缓存的查询遍历](https://github.com/yiisoft/yii2/issues/8420#issuecomment-296109257)。

### 添加自定义查询条件和表达式

我们在 [查询条件-对象格式](https://www.yiichina.com/doc/guide/2.0/db-query-builder#object-format) 章节中提到过，可以创建自定义的查询条件类。 举个栗子，我们需要创建一个查询条件，它可以检查某些字段小于特定值的情况。 当使用操作符格式时，代码如下：

```php
[
    'and',
    '>', 'posts', $minLimit,
    '>', 'comments', $minLimit,
    '>', 'reactions', $minLimit,
    '>', 'subscriptions', $minLimit
]
```

当这样的查询条件仅被应用一次，没什么问题。当它在一个查询语句中被多次使用时，就有很多优化点了。 我们创建一个自定义查询条件对象来证实它。

Yii 有 [ConditionInterface](https://www.yiichina.com/doc/api/2.0/yii-db-conditions-conditioninterface) 接口类，必须用它来标识这是一个表示查询条件的类。 它需要实现 `fromArrayDefinition()` 方法，用来从数组格式创建查询条件。 如果我们不需要它，抛出一个异常来完成此方法即可。

创建自定义查询条件类，我们就可以构建最适合当前需求的 API。

```php
namespace app\db\conditions;

class AllGreaterCondition implements \yii\db\conditions\ConditionInterface
{
    private $columns;
    private $value;

    /**
     * @param string[] $columns 要大于 $value 的字段名数组
     * @param mixed $value 每个 $column 要比较的数值
     */
    public function __construct(array $columns, $value)
    {
        $this->columns = $columns;
        $this->value = $value;
    }
    
    public static function fromArrayDefinition($operator, $operands)
    {
        throw new InvalidArgumentException('Not implemented yet, but we will do it later');
    }
    
    public function getColumns() { return $this->columns; }
    public function getValue() { return $this->vaule; }
}
```

我们现在创建了一个查询条件对象：

```php
$conditon = new AllGreaterCondition(['col1', 'col2'], 42);
```

但是 `QueryBuilder` 还不知道怎样从此对象生成 SQL 查询条件。 因此我们还需要为这个条件对象创建一个构建器（Builder）。 这个构建器必须实现 [yii\db\ExpressionBuilderInterface](https://www.yiichina.com/doc/api/2.0/yii-db-expressionbuilderinterface) 接口和 `build()` 方法。

```php
namespace app\db\conditions;

class AllGreaterConditionBuilder implements \yii\db\ExpressionBuilderInterface
{
    use \yii\db\ExpressionBuilderTrait; // Contains constructor and `queryBuilder` property.

    /**
     * @param ExpressionInterface $condition 要构建的查询条件对象
     * @param array $params 绑定的参数
     * @return AllGreaterCondition
     */ 
    public function build(ExpressionInterface $expression, array &$params = [])
    {
        $value = $condition->getValue();
        
        $conditions = [];
        foreach ($expression->getColumns() as $column) {
            $conditions[] = new SimpleCondition($column, '>', $value);
        }

        return $this->queryBuilder->buildCondition(new AndCondition($conditions), $params);
    }
}
```

接下来，让 [QueryBuilder](https://www.yiichina.com/doc/api/2.0/yii-db-querybuilder) ]知道我们的新查询条件对象 – 添加一个映射到 `expressionBuilders` 数组中，在应用配置（application config）中完成即可：

```php
'db' => [
    'class' => 'yii\db\mysql\Connection',
    // ...
    'queryBuilder' => [
        'expressionBuilders' => [
            'app\db\conditions\AllGreaterCondition' => 'app\db\conditions\AllGreaterConditionBuilder',
        ],
    ],
],
```

现在我们可以在 `where()` 中使用此查询条件对象了：

```php
$query->andWhere(new AllGreaterCondition(['posts', 'comments', 'reactions', 'subscriptions'], $minValue));
```

如果我们想要自定义操作符查询条件，可以在 [QueryBuilder::conditionClasses](https://www.yiichina.com/doc/api/2.0/yii-db-querybuilder#$conditionClasses-detail) 中 这样声明

```php
'db' => [
    'class' => 'yii\db\mysql\Connection',
    // ...
    'queryBuilder' => [
        'expressionBuilders' => [
            'app\db\conditions\AllGreaterCondition' => 'app\db\conditions\AllGreaterConditionBuilder',
        ],
        'conditionClasses' => [
            'ALL>' => 'app\db\conditions\AllGreaterCondition',
        ],
    ],
],
```

并在 `app\db\conditions\AllGreaterCondition` 对象中实现 `AllGreaterCondition::fromArrayDefinition()`方法：

```php
namespace app\db\conditions;

class AllGreaterCondition implements \yii\db\conditions\ConditionInterface
{
    // ... 这里省略其他方法的实现
     
    public static function fromArrayDefinition($operator, $operands)
    {
        return new static($operands[0], $operands[1]);
    }
}
```

然后呢，我们就可以使用更简短的操作符格式来创建自定义查询条件了：

```php
$query->andWhere(['ALL>', ['posts', 'comments', 'reactions', 'subscriptions'], $minValue]);
```

你可能注意到了，这里使用到了两个概念：表达式对象和条件对象。表达式对象实现了 [yii\db\ExpressionInterface](https://www.yiichina.com/doc/api/2.0/yii-db-expressioninterface) 接口， 它还依赖于一个表达式构建器（Expression Builder）来执行构建逻辑，而表达式构建器实现了 [yii\db\ExpressionBuilderInterface](https://www.yiichina.com/doc/api/2.0/yii-db-expressionbuilderinterface) 接口。 而条件对象实现了 yii\db\condition\ConditionInterface 接口，它是继承了 [ExpressionInterface](https://www.yiichina.com/doc/api/2.0/yii-db-expressioninterface) 接口， 如上面的栗子所写的，它用于数组定义的条件的场景，当然条件对象也需要构建器。

## 活动记录

[Active Record](http://zh.wikipedia.org/wiki/Active_Record) 提供了一个面向对象的接口， 用以访问和操作数据库中的数据。Active Record 类与数据库表关联， Active Record 实例对应于该表的一行， Active Record 实例的*属性*表示该行中特定列的值。 您可以访问 Active Record 属性并调用 Active Record 方法来访问和操作存储在数据库表中的数据， 而不用编写原始 SQL 语句。

例如，假定 `Customer` Active Record 类关联着 `customer` 表， 且该类的 `name` 属性代表 `customer` 表的 `name` 列。 你可以写以下代码来哉 `customer` 表里插入一行新的记录：

```php
$customer = new Customer();
$customer->name = 'Qiang';
$customer->save();
```

对于 MySQL，上面的代码和使用下面的原生 SQL 语句是等效的，但显然前者更直观， 更不易出错，并且面对不同的数据库系统（DBMS, Database Management System）时更不容易产生兼容性问题。

```php
$db->createCommand('INSERT INTO `customer` (`name`) VALUES (:name)', [
    ':name' => 'Qiang',
])->execute();
```

Yii 为以下关系数据库提供 Active Record 支持：

- MySQL 4.1 及以上：通过 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 支持
- PostgreSQL 7.3 及以上：通过 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 支持
- SQLite 2 and 3：通过 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 支持
- Microsoft SQL Server 2008 及以上：通过 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 支持
- Oracle：通过 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 支持
- CUBRID 9.3 及以上：通过 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 支持 (提示， 由于 CUBRID PDO 扩展的 [bug](http://jira.cubrid.org/browse/APIS-658)， 给变量加引用将不起作用，所以你得使用 CUBRID 9.3 客户端及服务端。
- Sphinx：通过 yii\sphinx\ActiveRecord 支持，依赖 `yii2-sphinx` 扩展
- ElasticSearch：通过 yii\elasticsearch\ActiveRecord 支持, 依赖 `yii2-elasticsearch` 扩展

此外，Yii 的 Active Record 功能还支持以下 NoSQL 数据库：

- Redis 2.6.12 及以上：通过 yii\redis\ActiveRecord 支持，依赖 `yii2-redis` 扩展
- MongoDB 1.3.0 及以上：通过 yii\mongodb\ActiveRecord 支持，依赖 `yii2-mongodb` 扩展

在本教程中，我们会主要描述对关系型数据库的 Active Record 用法。 然而，绝大多数的内容在 NoSQL 的 Active Record 里同样适用。

### 声明 Active Record 类

要想声明一个 Active Record 类，你需要声明该类继承 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord)。

#### 设置表的名称（Setting a table name）

默认的，每个 Active Record 类关联各自的数据库表。 经过 [yii\helpers\Inflector::camel2id()](https://www.yiichina.com/doc/api/2.0/yii-helpers-baseinflector#camel2id()-detail) 处理，[tableName()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#tableName()-detail) 方法默认返回的表名称是通过类名转换来得。 如果这个默认名称不正确，你得重写这个方法。

此外，[tablePrefix](https://www.yiichina.com/doc/api/2.0/yii-db-connection#$tablePrefix-detail) 表前缀也会起作用。例如，如果 [tablePrefix](https://www.yiichina.com/doc/api/2.0/yii-db-connection#$tablePrefix-detail) 表前缀是 `tbl_`，`Customer` 的类名将转换成 `tbl_customer` 表名，`OrderItem` 转换成 `tbl_order_item`。

如果你定义的表名是 `{{%TableName}}`，百分比字符 `%` 会被替换成表前缀。 例如，`{{%post}}` 会变成 `{{tbl_post}}`。表名两边的括号会被 [SQL 查询引用](https://www.yiichina.com/doc/guide/2.0/db-dao#quoting-table-and-column-names) 处理。

下面的例子中，我们给 `customer` 数据库表定义叫 `Customer` 的 Active Record 类。

```php
namespace app\models;

use yii\db\ActiveRecord;

class Customer extends ActiveRecord
{
    const STATUS_INACTIVE = 0;
    const STATUS_ACTIVE = 1;
    
    /**
     * @return string Active Record 类关联的数据库表名称
     */
    public static function tableName()
    {
        return '{{customer}}';
    }
}
```

#### 将 Active Record 称为模型

Active Record 实例称为[模型](https://www.yiichina.com/doc/guide/2.0/structure-models)。因此, 我们通常将 Active Record 类 放在 `app\models` 命名空间下（或者其他保存模型的命名空间）。

因为 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 继承了模型 [yii\base\Model](https://www.yiichina.com/doc/api/2.0/yii-base-model)，它就拥有所有[模型](https://www.yiichina.com/doc/guide/2.0/structure-models)特性， 比如说属性（attributes），验证规则（rules），数据序列化（data serialization），等等

### 建立数据库连接

活动记录 Active Record 默认使用 `db` [组件](https://www.yiichina.com/doc/guide/2.0/structure-application-components) 作为连接器 [DB connection](https://www.yiichina.com/doc/api/2.0/yii-db-connection) 访问和操作数据库数据。 基于[数据库访问](https://www.yiichina.com/doc/guide/2.0/db-dao)中的解释，你可以在系统配置中 这样配置 `db` 组件。

```php
return [
    'components' => [
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=testdb',
            'username' => 'demo',
            'password' => 'demo',
        ],
    ],
];
```

如果你要用不同的数据库连接，而不仅仅是 `db` 组件， 你可以重写 [getDb()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#getDb()-detail) 方法

```php
class Customer extends ActiveRecord
{
    // ...

    public static function getDb()
    {
        // 使用 "db2" 组件
        return \Yii::$app->db2;  
    }
}
```

### 查询数据

定义 Active Record 类后，你可以从相应的数据库表中查询数据。 查询过程大致如下三个步骤：

1. 通过 [yii\db\ActiveRecord::find()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#find()-detail) 方法创建一个新的查询生成器对象；
2. 使用[查询生成器的构建方法](https://www.yiichina.com/doc/guide/2.0/db-query-builder#building-queries)来构建你的查询；
3. 调用[查询生成器的查询方法](https://www.yiichina.com/doc/guide/2.0/db-query-builder#query-methods)来取出数据到 Active Record 实例中。

正如你看到的，是不是跟[查询生成器](https://www.yiichina.com/doc/guide/2.0/db-query-builder)的步骤差不多。 唯一有区别的地方在于你用 [yii\db\ActiveRecord::find()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#find()-detail) 去获得一个新的查询生成器对象，这个对象是 [yii\db\ActiveQuery](https://www.yiichina.com/doc/api/2.0/yii-db-activequery)， 而不是使用 `new` 操作符创建一个查询生成器对象。

下面是一些例子，介绍如何使用 Active Query 查询数据

```php
// 返回 ID 为 123 的客户：
// SELECT * FROM `customer` WHERE `id` = 123
$customer = Customer::find()
    ->where(['id' => 123])
    ->one();

// 取回所有活跃客户并以他们的 ID 排序：
// SELECT * FROM `customer` WHERE `status` = 1 ORDER BY `id`
$customers = Customer::find()
    ->where(['status' => Customer::STATUS_ACTIVE])
    ->orderBy('id')
    ->all();

// 取回活跃客户的数量：
// SELECT COUNT(*) FROM `customer` WHERE `status` = 1
$count = Customer::find()
    ->where(['status' => Customer::STATUS_ACTIVE])
    ->count();

// 以客户 ID 索引结果集：
// SELECT * FROM `customer`
$customers = Customer::find()
    ->indexBy('id')
    ->all();
```

上述代码中，`$customer` 是个 `Customer` 对象，而 `$customers` 是个以 `Customer` 对象为元素的数组。 它们两都是以 `customer` 表中取回的数据结果集填充的。

> **提示：** 由于 [yii\db\ActiveQuery](https://www.yiichina.com/doc/api/2.0/yii-db-activequery) 继承 [yii\db\Query](https://www.yiichina.com/doc/api/2.0/yii-db-query)， 你可以使用 [Query Builder](https://www.yiichina.com/doc/guide/2.0/db-query-builder) 章节里所描述的*所有*查询方法。

根据主键获取数据行是比较常见的操作，所以 Yii 提供了两个快捷方法：

- [yii\db\ActiveRecord::findOne()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#findOne()-detail)：返回一个 Active Record 实例，填充于查询结果的第一行数据。
- [yii\db\ActiveRecord::findAll()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#findAll()-detail)：返回一个 Active Record 实例的数据，填充于查询结果的全部数据

这两个方法的传参格式如下：

- 标量值：这个值会当作主键去查询。 Yii 会通过读取数据库模式信息来识别主键列。
- 标量值的数组：这数组里的值都当作要查询的主键的值。
- 关联数组：键值是表的列名，元素值是相应的要查询的条件值。 可以到 [哈希格式](https://www.yiichina.com/doc/guide/2.0/db-query-builder#hash-format) 查看更多信息。

如下代码描述如何使用这些方法：

```php
// 返回 id 为 123 的客户 
// SELECT * FROM `customer` WHERE `id` = 123
$customer = Customer::findOne(123);

// 返回 id 是 100, 101, 123, 124 的客户
// SELECT * FROM `customer` WHERE `id` IN (100, 101, 123, 124)
$customers = Customer::findAll([100, 101, 123, 124]);

// 返回 id 是 123 的活跃客户
// SELECT * FROM `customer` WHERE `id` = 123 AND `status` = 1
$customer = Customer::findOne([
    'id' => 123,
    'status' => Customer::STATUS_ACTIVE,
]);

// 返回所有不活跃的客户
// SELECT * FROM `customer` WHERE `status` = 0
$customers = Customer::findAll([
    'status' => Customer::STATUS_INACTIVE,
]);
```

> **警告：** 如果你需要将用户输入传递给这些方法，请确保输入值是标量或者是 数组条件，确保数组结构不能被外部所改变：

```php
// yii\web\Controller 确保了 $id 是标量
public function actionView($id)
{
    $model = Post::findOne($id);
    // ...
}

// 明确了指定要搜索的列，在此处传递标量或数组将始终只是查找出单个记录而已
$model = Post::findOne(['id' => Yii::$app->request->get('id')]);

// 不要使用下面的代码！可以注入一个数组条件来匹配任意列的值！
$model = Post::findOne(Yii::$app->request->get('id'));
```

> **提示：** [yii\db\ActiveRecord::findOne()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#findOne()-detail) 和 [yii\db\ActiveQuery::one()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#one()-detail) 都不会添加 `LIMIT 1` 到 生成的 SQL 语句中。如果你的查询会返回很多行的数据， 你明确的应该加上 `limit(1)` 来提高性能，比如 `Customer::find()->limit(1)->one()`。

除了使用查询生成器的方法之外，你还可以书写原生的 SQL 语句来查询数据，并填充结果集到 Active Record 对象中。 通过使用 [yii\db\ActiveRecord::findBySql()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#findBySql()-detail) 方法:

```php
// 返回所有不活跃的客户
$sql = 'SELECT * FROM customer WHERE status=:status';
$customers = Customer::findBySql($sql, [':status' => Customer::STATUS_INACTIVE])->all();
```

不要在 [findBySql()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#findBySql()-detail) 方法后加其他查询方法了， 多余的查询方法都会被忽略。

### 访问数据

如上所述，从数据库返回的数据被填充到 Active Record 实例中， 查询结果的每一行对应于单个 Active Record 实例。 您可以通过 Active Record 实例的属性来访问列值，例如，

```php
// "id" 和 "email" 是 "customer" 表中的列名
$customer = Customer::findOne(123);
$id = $customer->id;
$email = $customer->email;
```

> **提示：** Active Record 的属性以区分大小写的方式为相关联的表列命名的。 Yii 会自动为关联表的每一列定义 Active Record 中的一个属性。 您不应该重新声明任何属性。

由于 Active Record 的属性以表的列名命名，可能你会发现你正在编写像这样的 PHP 代码： `$customer->first_name`，如果你的表的列名是使用下划线分隔的，那么属性名中的单词 以这种方式命名。 如果您担心代码风格一致性的问题，那么你应当重命名相应的表列名 （例如使用骆驼拼写法）。

### 数据转换

常常遇到，要输入或显示的数据是一种格式，而要将其存储在数据库中是另一种格式。 例如，在数据库中，您将客户的生日存储为 UNIX 时间戳（虽然这不是一个很好的设计）， 而在大多数情况下，你想以字符串 `'YYYY/MM/DD'` 的格式处理生日数据。 为了实现这一目标，您可以在 `Customer` 中定义 *数据转换* 方法 定义 Active Record 类如下：

```php
class Customer extends ActiveRecord
{
    // ...

    public function getBirthdayText()
    {
        return date('Y/m/d', $this->birthday);
    }
    
    public function setBirthdayText($value)
    {
        $this->birthday = strtotime($value);
    }
}
```

现在你的 PHP 代码中，你可以访问 `$customer->birthdayText`， 来以 `'YYYY/MM/DD'` 的格式输入和显示客户生日，而不是访问 `$customer->birthday`。

> **提示：** 上述示例显示了以不同格式转换数据的通用方法。如果你正在使用 日期值，您可以使用 [DateValidator](https://www.yiichina.com/doc/guide/2.0/tutorial-core-validators#date) 和 yii\jui\DatePicker 来操作， 这将更易用，更强大。

### 以数组形式获取数据

通过 Active Record 对象获取数据十分方便灵活，与此同时，当你需要返回大量的数据的时候， 这样的做法并不令人满意，因为这将导致大量内存占用。在这种情况下，您可以 在查询方法前调用 [asArray()](https://www.yiichina.com/doc/api/2.0/yii-db-activequerytrait#asArray()-detail) 方法，来获取 PHP 数组形式的结果：

```php
// 返回所有客户
// 每个客户返回一个关联数组
$customers = Customer::find()
    ->asArray()
    ->all();
```

> **提示：** 虽然这种方法可以节省内存并提高性能，但它更靠近较低的 DB 抽象层 你将失去大部分的 Active Record 提供的功能。 一个非常重要的区别在于列值的数据类型。 当您在 Active Record 实例中返回数据时，列值将根据实际列类型，自动类型转换； 然而，当您以数组返回数据时，列值将为 字符串（因为它们是没有处理过的 PDO 的结果），不管它们的实际列是什么类型。

### 批量获取数据

在 [查询生成器](https://www.yiichina.com/doc/guide/2.0/db-query-builder) 中，我们已经解释说可以使用 *批处理查询* 来最小化你的内存使用， 每当从数据库查询大量数据。你可以在 Active Record 中使用同样的技巧。例如

```php
// 每次获取 10 条客户数据
foreach (Customer::find()->batch(10) as $customers) {
    // $customers 是个最多拥有 10 条数据的数组
}

// 每次获取 10 条客户数据，然后一条一条迭代它们
foreach (Customer::find()->each(10) as $customer) {
    // $customer 是个 `Customer` 对象
}

// 贪婪加载模式的批处理查询
foreach (Customer::find()->with('orders')->each() as $customer) {
    // $customer 是个 `Customer` 对象，并附带关联的 `'orders'`
}
```

### 保存数据

使用 Active Record，您可以通过以下步骤轻松地将数据保存到数据库：

1. 准备一个 Active Record 实例
2. 将新值赋给 Active Record 的属性
3. 调用 [yii\db\ActiveRecord::save()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#save()-detail) 保存数据到数据库中。

例如，

```php
// 插入新记录
$customer = new Customer();
$customer->name = 'James';
$customer->email = 'james@example.com';
$customer->save();

// 更新已存在的记录
$customer = Customer::findOne(123);
$customer->email = 'james@newexample.com';
$customer->save();
```

[save()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#save()-detail) 方法可能插入或者更新表的记录，这取决于 Active Record 实例的状态。 如果实例通过 `new` 操作符实例化，调用 [save()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#save()-detail) 方法将插入新记录； 如果实例是一个查询方法的结果，调用 [save()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#save()-detail) 方法 将更新这个实例对应的表记录行。

你可以通过检查 Active Record 实例的 [isNewRecord](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#$isNewRecord-detail) 属性值来区分这两个状态。 此属性也被使用在 [save()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#save()-detail) 方法内部，

```php
public function save($runValidation = true, $attributeNames = null)
{
    if ($this->getIsNewRecord()) {
        return $this->insert($runValidation, $attributeNames);
    } else {
        return $this->update($runValidation, $attributeNames) !== false;
    }
}
```

> **提示：** 你可以直接调用 [insert()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#insert()-detail) 或者 [update()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#update()-detail) 方法来插入或更新一条记录。

> 注意：表里所要更新的字段不要在model里设置同样的属性值，属性为public，否则会在插入或者更新时造成字段为空



### 数据验证

因为 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 继承于 [yii\base\Model](https://www.yiichina.com/doc/api/2.0/yii-base-model)，它共享相同的 [输入验证](https://www.yiichina.com/doc/guide/2.0/input-validation) 功能。 你可以通过重写 [rules()](https://www.yiichina.com/doc/api/2.0/yii-base-model#rules()-detail) 方法声明验证规则并执行， 通过调用 [validate()](https://www.yiichina.com/doc/api/2.0/yii-base-model#validate()-detail) 方法进行数据验证。

当你调用 [save()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#save()-detail) 时，默认情况下会自动调用 [validate()](https://www.yiichina.com/doc/api/2.0/yii-base-model#validate()-detail)。 只有当验证通过时，它才会真正地保存数据; 否则将简单地返回 `false`， 您可以检查 [errors](https://www.yiichina.com/doc/api/2.0/yii-base-model#$errors-detail) 属性来获取验证过程的错误消息。

> **提示：** 如果你确定你的数据不需要验证（比如说数据来自可信的场景）， 你可以调用 `save(false)` 来跳过验证过程。

### 块赋值

和普通的 [模型](https://www.yiichina.com/doc/guide/2.0/structure-models) 一样，你亦可以享受 Active Record 实例的 [块赋值](https://www.yiichina.com/doc/guide/2.0/structure-models#massive-assignment) 特性。 使用此功能，您可以在单个 PHP 语句中，给 Active Record 实例的多个属性批量赋值， 如下所示。 记住，只有 [安全属性](https://www.yiichina.com/doc/guide/2.0/structure-models#safe-attributes) 才可以批量赋值。

```php
values = [
    'name' => 'James',
    'email' => 'james@example.com',
];

$customer = new Customer();

$customer->attributes = $values;
$customer->save();
```

> 注意：要使用块赋值，必须在model里定义好rules()里的规则字段，否则所更新的字段全部为Null

### 更新计数

在数据库表中增加或减少一个字段的值是个常见的任务。我们将这些列称为“计数列”。 您可以使用 [updateCounters()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#updateCounters()-detail) 更新一个或多个计数列。 例如

```php
$post = Post::findOne(100);

// UPDATE `post` SET `view_count` = `view_count` + 1 WHERE `id` = 100
$post->updateCounters(['view_count' => 1]);
```

> **注意：** 如果你使用 [yii\db\ActiveRecord::save()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#save()-detail) 更新一个计数列，你最终将得到错误的结果， 因为可能发生这种情况，多个请求间并发读写同一个计数列。

### 脏属性

当您调用 [save()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#save()-detail) 保存 Active Record 实例时，只有 *脏属性* 被保存。如果一个属性的值已被修改，则会被认为是 *脏*，因为它是从 DB 加载出来的或者 刚刚保存到 DB 。请注意，无论如何 Active Record 都会执行数据验证 不管有没有脏属性。

Active Record 自动维护脏属性列表。它保存所有属性的旧值， 并其与最新的属性值进行比较。你可以调用 [yii\db\ActiveRecord::getDirtyAttributes()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#getDirtyAttributes()-detail) 获取当前的脏属性。你也可以调用 [yii\db\ActiveRecord::markAttributeDirty()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#markAttributeDirty()-detail) 将属性显式标记为脏。

如果你有需要获取属性原先的值，你可以调用 [getOldAttributes()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#getOldAttributes()-detail) 或者 [getOldAttribute()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#getOldAttribute()-detail)。

> 注：属性新旧值的比较是用 `===` 操作符，所以一样的值但类型不同， 依然被认为是脏的。当模型从 HTML 表单接收用户输入时，通常会出现这种情况， 其中每个值都表示为一个字符串类型。 为了确保正确的类型，比如，整型需要用[过滤验证器](https://www.yiichina.com/doc/guide/2.0/input-validation#data-filtering)： `['attributeName', 'filter', 'filter' => 'intval']`。其他 PHP 类型转换函数一样适用，像 [intval()](http://php.net/manual/en/function.intval.php)， [floatval()](http://php.net/manual/en/function.floatval.php)， [boolval](http://php.net/manual/en/function.boolval.php)，等等

### 默认属性值

某些表列可能在数据库中定义了默认值。有时，你可能想预先填充 具有这些默认值的 Active Record 实例的 Web 表单。 为了避免再次写入相同的默认值， 您可以调用 [loadDefaultValues()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#loadDefaultValues()-detail) 来填充 DB 定义的默认值 进入相应的 Active Record 属性：

```php
$customer = new Customer();
$customer->loadDefaultValues();
// $customer->xyz 将被 “zyz” 列定义的默认值赋值
```

### 属性类型转换

在查询结果填充 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) 时，将自动对其属性值执行类型转换，基于 [数据库表模式](https://www.yiichina.com/doc/guide/2.0/db-dao#database-schema) 中的信息。 这允许从数据表中获取数据， 声明为整型的，使用 PHP 整型填充 ActiveRecord 实例，布尔值（boolean）的也用布尔值填充，等等。 但是，类型转换机制有几个限制：

- 浮点值不被转换，并且将被表示为字符串，否则它们可能会使精度降低。
- 整型值的转换取决于您使用的操作系统的整数容量。尤其是： 声明为“无符号整型”或“大整型”的列的值将仅转换为 64 位操作系统的 PHP 整型， 而在 32 位操作系统中 - 它们将被表示为字符串。

值得注意的是，只有在从查询结果填充 ActiveRecord 实例时才执行属性类型转换。 而从 HTTP 请求加载的值或直接通过属性访问赋值的，没有自动转换。 在准备用于在 ActiveRecord 保存时，准备 SQL 语句还使用了表模式，以确保查询时 值绑定到具有正确类型的。但是，ActiveRecord 实例的属性值不会 在保存过程中转换。

> **提示：** 你可以使用 [yii\behaviors\AttributeTypecastBehavior](https://www.yiichina.com/doc/api/2.0/yii-behaviors-attributetypecastbehavior) 来简化属性的类型转换 在 ActiveRecord 验证或者保存过程中。

从 2.0.14 开始，Yii ActiveRecord 支持了更多的复杂数据类型，例如 JSON 或多维数组。

### 更新多个数据行

上述方法都可以用于单个 Active Record 实例，以插入或更新单条 表数据行。 要同时更新多个数据行，你应该调用 [updateAll()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#updateAll()-detail) 这是一个静态方法。

```php
// UPDATE `customer` SET `status` = 1 WHERE `email` LIKE `%@example.com%`
Customer::updateAll(['status' => Customer::STATUS_ACTIVE], ['like', 'email', '@example.com']);
```

同样，你可以调用 [updateAllCounters()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#updateAllCounters()-detail) 同时更新多条记录的计数列。

```php
// UPDATE `customer` SET `age` = `age` + 1
Customer::updateAllCounters(['age' => 1]);
```

### 删除数据

要删除单行数据，首先获取与该行对应的 Active Record 实例，然后调用 [yii\db\ActiveRecord::delete()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#delete()-detail) 方法。

```php
$customer = Customer::findOne(123);
$customer->delete();
```

你可以调用 [yii\db\ActiveRecord::deleteAll()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#deleteAll()-detail) 方法删除多行甚至全部的数据。例如，

```php
Customer::deleteAll(['status' => Customer::STATUS_INACTIVE]);
```

> **提示：** 调用 [deleteAll()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#deleteAll()-detail) 时要非常小心，因为如果在指定条件时出错， 它可能会完全擦除表中的所有数据。

### Active Record 的生命周期

当你实现各种功能的时候，会发现了解 Active Record 的生命周期很重要。 在每个生命周期中，一系列的方法将被调用执行，您可以重写这些方法 以定制你要的生命周期。您还可以响应触发某些 Active Record 事件 以便在生命周期中注入您的自定义代码。这些事件在开发 Active Record 的 [行为](https://www.yiichina.com/doc/guide/2.0/concept-behaviors)时特别有用， 通过行为可以定制 Active Record 生命周期的 。

下面，我们将总结各种 Active Record 的生命周期，以及生命周期中 所涉及的各种方法、事件。

#### 实例化生命周期

当通过 `new` 操作符新建一个 Active Record 实例时，会发生以下生命周期：

1. 类的构造函数调用.
2. [init()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#init()-detail)：触发 [EVENT_INIT](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_INIT-detail) 事件。

#### 查询数据生命周期

当通过 [查询方法](https://www.yiichina.com/doc/guide/2.0/db-active-record#querying-data) 查询数据时，每个新填充出来的 Active Record 实例 将发生下面的生命周期：

1. 类的构造函数调用。
2. [init()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#init()-detail)：触发 [EVENT_INIT](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_INIT-detail) 事件。
3. [afterFind()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#afterFind()-detail)：触发 [EVENT_AFTER_FIND](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_AFTER_FIND-detail) 事件。

#### 保存数据生命周期

当通过 [save()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#save()-detail) 插入或更新 Active Record 实例时 会发生以下生命周期：

1. [beforeValidate()](https://www.yiichina.com/doc/api/2.0/yii-base-model#beforeValidate()-detail)：触发 [EVENT_BEFORE_VALIDATE](https://www.yiichina.com/doc/api/2.0/yii-base-model#EVENT_BEFORE_VALIDATE-detail) 事件。如果这方法返回 `false` 或者 [yii\base\ModelEvent::$isValid](https://www.yiichina.com/doc/api/2.0/yii-base-modelevent#$isValid-detail) 值为 `false`，接下来的步骤都会被跳过。
2. 执行数据验证。如果数据验证失败，步骤 3 之后的步骤将被跳过。
3. [afterValidate()](https://www.yiichina.com/doc/api/2.0/yii-base-model#afterValidate()-detail)：触发 [EVENT_AFTER_VALIDATE](https://www.yiichina.com/doc/api/2.0/yii-base-model#EVENT_AFTER_VALIDATE-detail) 事件。
4. [beforeSave()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#beforeSave()-detail)：触发 [EVENT_BEFORE_INSERT](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_BEFORE_INSERT-detail) 或者 [EVENT_BEFORE_UPDATE](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_BEFORE_UPDATE-detail) 事件。 如果这方法返回 `false` 或者 [yii\base\ModelEvent::$isValid](https://www.yiichina.com/doc/api/2.0/yii-base-modelevent#$isValid-detail) 值为 `false`，接下来的步骤都会被跳过。
5. 执行真正的数据插入或者更新。
6. [afterSave()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#afterSave()-detail)：触发 [EVENT_AFTER_INSERT](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_AFTER_INSERT-detail) 或者 [EVENT_AFTER_UPDATE](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_AFTER_UPDATE-detail) 事件。

#### 删除数据生命周期

当通过 [delete()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#delete()-detail) 删除 Active Record 实例时， 会发生以下生命周期：

1. [beforeDelete()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#beforeDelete()-detail)：触发 [EVENT_BEFORE_DELETE](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_BEFORE_DELETE-detail) 事件。 如果这方法返回 `false` 或者 [yii\base\ModelEvent::$isValid](https://www.yiichina.com/doc/api/2.0/yii-base-modelevent#$isValid-detail) 值为 `false`，接下来的步骤都会被跳过。
2. 执行真正的数据删除。
3. [afterDelete()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#afterDelete()-detail)：触发 [EVENT_AFTER_DELETE](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_AFTER_DELETE-detail) 事件。

> **提示：** 调用以下方法则不会启动上述的任何生命周期， 因为这些方法直接操作数据库，而不是基于 Active Record 模型：
>
> - [yii\db\ActiveRecord::updateAll()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#updateAll()-detail)
> - [yii\db\ActiveRecord::deleteAll()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#deleteAll()-detail)
> - [yii\db\ActiveRecord::updateCounters()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#updateCounters()-detail)
> - [yii\db\ActiveRecord::updateAllCounters()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#updateAllCounters()-detail)

#### 刷新数据生命周期

当通过 [refresh()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#refresh()-detail) 刷新 Active Record 实例时， 如刷新成功方法返回 `true`，那么 [EVENT_AFTER_REFRESH](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#EVENT_AFTER_REFRESH-detail) 事件将被触发。

### 事务操作

Active Record 有两种方式来使用[事务](https://www.yiichina.com/doc/guide/2.0/db-dao#performing-transactions)。

第一种方法是在事务块中显式地包含 Active Record 的各个方法调用，如下所示，

```php
$customer = Customer::findOne(123);

Customer::getDb()->transaction(function($db) use ($customer) {
    $customer->id = 200;
    $customer->save();
    // ...其他 DB 操作...
});

// 或者

$transaction = Customer::getDb()->beginTransaction();
try {
    $customer->id = 200;
    $customer->save();
    // ...other DB operations...
    $transaction->commit();
} catch(\Exception $e) {
    $transaction->rollBack();
    throw $e;
} catch(\Throwable $e) {
    $transaction->rollBack();
    throw $e;
}
```

> **提示：** 在上面的代码中，我们有两个catch块用于兼容 PHP 5.x 和 PHP 7.x。 `\Exception` 继承于 [`\Throwable` interface](http://php.net/manual/en/class.throwable.php) 由于 PHP 7.0 的改动，如果您的应用程序仅使用 PHP 7.0 及更高版本，您可以跳过 `\Exception` 部分

第二种方法是在 [yii\db\ActiveRecord::transactions()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#transactions()-detail) 方法中列出需要事务支持的 DB 操作。 例如

```php
class Customer extends ActiveRecord
{
    public function transactions()
    {
        return [
            'admin' => self::OP_INSERT,
            'api' => self::OP_INSERT | self::OP_UPDATE | self::OP_DELETE,
            // 上面等价于：
            // 'api' => self::OP_ALL,
        ];
    }
}
```

[yii\db\ActiveRecord::transactions()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#transactions()-detail) 方法应当返回以 [场景](https://www.yiichina.com/doc/guide/2.0/structure-models#scenarios) 为键、 以需要放到事务中的 DB 操作为值的数组。以下的常量 可以表示相应的 DB 操作：

- [OP_INSERT](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#OP_INSERT-detail)：插入操作用于执行 [insert()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#insert()-detail)；
- [OP_UPDATE](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#OP_UPDATE-detail)：更新操作用于执行 [update()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#update()-detail)；
- [OP_DELETE](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#OP_DELETE-detail)：删除操作用于执行 [delete()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#delete()-detail)。

使用 `|` 运算符连接上述常量来表明多个操作。您也可以使用 快捷常量 [OP_ALL](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#OP_ALL-detail) 来指代上述所有的三个操作。

这个事务方法的原理是：相应的事务在调用 [beforeSave()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#beforeSave()-detail) 方法时开启， 在调用 [afterSave()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#afterSave()-detail) 方法时被提交。

### 乐观锁

乐观锁是一种防止此冲突的方法：一行数据 同时被多个用户更新。例如，同一时间内，用户 A 和用户 B 都在编辑 相同的 wiki 文章。用户 A 保存他的编辑后，用户 B 也点击“保存”按钮来 保存他的编辑。实际上，用户 B 正在处理的是过时版本的文章， 因此最好是，想办法阻止他保存文章并向他提示一些信息。

乐观锁通过使用一个字段来记录每行的版本号来解决上述问题。 当使用过时的版本号保存一行数据时，[yii\db\StaleObjectException](https://www.yiichina.com/doc/api/2.0/yii-db-staleobjectexception) 异常 将被抛出，这阻止了该行的保存。乐观锁只支持更新 [yii\db\ActiveRecord::update()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#update()-detail) 或者删除 [yii\db\ActiveRecord::delete()](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord#delete()-detail) 已经存在的单条数据行。

使用乐观锁的步骤，

1. 在与 Active Record 类相关联的 DB 表中创建一个列，以存储每行的版本号。 这个列应当是长整型（在 MySQL 中是 `BIGINT DEFAULT 0`）。
2. 重写 [yii\db\ActiveRecord::optimisticLock()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#optimisticLock()-detail) 方法返回这个列的命名。
3. 在你的 Model 类里实现 [OptimisticLockBehavior](https://www.yiichina.com/doc/api/2.0/yii-behaviors-optimisticlockbehavior) 行为（注：这个行为类在 2.0.16 版本加入），以便从请求参数里自动解析这个列的值。 然后从验证规则中删除 version 属性，因为 [OptimisticLockBehavior](https://www.yiichina.com/doc/api/2.0/yii-behaviors-optimisticlockbehavior) 已经处理它了.
4. 在用于用户填写的 Web 表单中，添加一个隐藏字段（hidden field）来存储正在更新的行的当前版本号。
5. 在使用 Active Record 更新数据的控制器动作中，要捕获（try/catch） [yii\db\StaleObjectException](https://www.yiichina.com/doc/api/2.0/yii-db-staleobjectexception) 异常。 实现一些业务逻辑来解决冲突（例如合并更改，提示陈旧的数据等等）。

例如，假定版本列被命名为 `version`。您可以使用下面的代码来实现乐观锁

```php
// ------ 视图层代码 -------

use yii\helpers\Html;

// ...其他输入栏
echo Html::activeHiddenInput($model, 'version');


// ------ 控制器代码 -------

use yii\db\StaleObjectException;

public function actionUpdate($id)
{
    $model = $this->findModel($id);

    try {
        if ($model->load(Yii::$app->request->post()) && $model->save()) {
            return $this->redirect(['view', 'id' => $model->id]);
        } else {
            return $this->render('update', [
                'model' => $model,
            ]);
        }
    } catch (StaleObjectException $e) {
        // 解决冲突的代码
    }
}

// ------ Model 代码 -------

use yii\behaviors\OptimisticLockBehavior;

public function behaviors()
{
    return [
        OptimisticLockBehavior::className(),
    ];
}
```

> **注意：** 因为 [OptimisticLockBehavior](https://www.yiichina.com/doc/api/2.0/yii-behaviors-optimisticlockbehavior) 仅仅在保存记录的时候被确认， 如果用户提交的有效版本号被直接解析 ：[getBodyParam()](https://www.yiichina.com/doc/api/2.0/yii-web-request#getBodyParam()-detail)， 那么你的 Model 将扩展成这样：触发在步骤 3 中子类的行为，与此同时，调用步骤 2 中的父类的定义， 这样你在把 Model 绑定到负责接收用户输入的控制器的同时，有一个专门用于内部逻辑处理的实例， 或者，您可以通过配置其 [value](https://www.yiichina.com/doc/api/2.0/yii-behaviors-optimisticlockbehavior#$value-detail) 的属性来实现自己的逻辑。（注：这一堆都是在解释 Behaviors 的原理）

### 使用关联数据

除了处理单个数据库表之外，Active Record 还可以将相关数据集中进来， 使其可以通过原始数据轻松访问。 例如，客户数据与订单数据相关 因为一个客户可能已经存放了一个或多个订单。这种关系通过适当的声明， 你可以使用 `$customer->orders` 表达式访问客户的订单信息 这表达式将返回包含 `Order` Active Record 实例的客户订单信息的数组

#### 声明关联关系

你必须先在 Active Record 类中定义关联关系，才能使用 Active Record 的关联数据。 简单地为每个需要定义关联关系声明一个 *关联方法* 即可，如下所示，

```php
class Customer extends ActiveRecord
{
    // ...

    public function getOrders()
    {
        return $this->hasMany(Order::className(), ['customer_id' => 'id']);
    }
}

class Order extends ActiveRecord
{
    // ...

    public function getCustomer()
    {
        return $this->hasOne(Customer::className(), ['id' => 'customer_id']);
    }
}
```

上述的代码中，我们为 `Customer` 类声明了一个 `orders` 关联， 和为 `Order` 声明了一个 `customer` 关联。

每个关联方法必须这样命名：`getXyz`。然后我们通过 `xyz`（首字母小写）调用这个关联名。 请注意关联名是大小写敏感的。

当声明一个关联关系的时候，必须指定好以下的信息：

- 关联的对应关系：通过调用 [hasMany()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#hasMany()-detail) 或者 [hasOne()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#hasOne()-detail) 指定。在上面的例子中，您可以很容易看出这样的关联声明： 一个客户可以有很多订单，而每个订单只有一个客户。

- 相关联 Active Record 类名：用来指定为 [hasMany()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#hasMany()-detail) 或者 [hasOne()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#hasOne()-detail) 方法的第一个参数。 推荐的做法是调用 `Xyz::className()` 来获取类名称的字符串，以便您 可以使用 IDE 的自动补全，以及让编译阶段的错误检测生效。

- 两组数据的关联列：用以指定两组数据相关的列（hasOne()/hasMany() 的第二个参数）。 数组的值填的是主数据的列（当前要声明关联的 Active Record 类为主数据）， 而数组的键要填的是相关数据的列。

  一个简单的口诀，先附表的主键，后主表的主键。 正如上面的例子，`customer_id` 是 `Order` 的属性，而 `id`是 `Customer` 的属性。 （译者注：hasMany() 的第二个参数，这个数组键值顺序不要弄反了）

#### 访问关联数据

定义了关联关系后，你就可以通过关联名访问相应的关联数据了。就像 访问一个由关联方法定义的对象一样，具体概念请查看 [属性](https://www.yiichina.com/doc/guide/2.0/concept-properties)。 因此，现在我们可以称它为 *关联属性* 了。

```php
// SELECT * FROM `customer` WHERE `id` = 123
$customer = Customer::findOne(123);

// SELECT * FROM `order` WHERE `customer_id` = 123
// $orders 是由 Order 类组成的数组
$orders = $customer->orders;
```

> **提示：** 当你通过 getter 方法 `getXyz()` 声明了一个叫 `xyz` 的关联属性，你就可以像 [属性](https://www.yiichina.com/doc/guide/2.0/concept-properties) 那样访问 `xyz`。注意这个命名是区分大小写的。

如果使用 [hasMany()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#hasMany()-detail) 声明关联关系，则访问此关联属性 将返回相关的 Active Record 实例的数组； 如果使用 [hasOne()](https://www.yiichina.com/doc/api/2.0/yii-db-baseactiverecord#hasOne()-detail) 声明关联关系，访问此关联属性 将返回相关的 Active Record 实例，如果没有找到相关数据的话，则返回 `null`。

当你第一次访问关联属性时，将执行 SQL 语句获取数据，如 上面的例子所示。如果再次访问相同的属性，将返回先前的结果，而不会重新执行 SQL 语句。要强制重新执行 SQL 语句，你应该先 unset 这个关联属性， 如：`unset（$ customer-> orders）`。

> **提示：** 虽然这个概念跟 这个 [属性](https://www.yiichina.com/doc/guide/2.0/concept-properties) 特性很像， 但是还是有一个很重要的区别。普通对象属性的属性值与其定义的 getter 方法的类型是相同的。 而关联方法返回的是一个 [yii\db\ActiveQuery](https://www.yiichina.com/doc/api/2.0/yii-db-activequery) 活动查询生成器的实例。只有当访问关联属性的的时候， 才会返回 [yii\db\ActiveRecord](https://www.yiichina.com/doc/api/2.0/yii-db-activerecord) Active Record 实例，或者 Active Record 实例组成的数组。
>
> ```php
> $customer->orders; // 获得 `Order` 对象的数组
> $customer->getOrders(); // 返回 ActiveQuery 类的实例
> ```
>
> 这对于创建自定义查询很有用，下一节将对此进行描述。

#### 动态关联查询

由于关联方法返回 [yii\db\ActiveQuery](https://www.yiichina.com/doc/api/2.0/yii-db-activequery) 的实例，因此你可以在执行 DB 查询之前， 使用查询构建方法进一步构建此查询。例如，

```php
$customer = Customer::findOne(123);

// SELECT * FROM `order` WHERE `customer_id` = 123 AND `subtotal` > 200 ORDER BY `id`
$orders = $customer->getOrders()
    ->where(['>', 'subtotal', 200])
    ->orderBy('id')
    ->all();
```

与访问关联属性不同，每次通过关联方法执行动态关联查询时， 都会执行 SQL 语句，即使你之前执行过相同的动态关联查询。

有时你可能需要给你的关联声明传递参数，以便您能更方便地执行 动态关系查询。例如，您可以声明一个 `bigOrders` 关联如下，

```php
class Customer extends ActiveRecord
{
    public function getBigOrders($threshold = 100) // 老司机的提醒：$threshold 参数一定一定要给个默认值
    {
        return $this->hasMany(Order::className(), ['customer_id' => 'id'])
            ->where('subtotal > :threshold', [':threshold' => $threshold])
            ->orderBy('id');
    }
}
```

然后你就可以执行以下关联查询

```php
// SELECT * FROM `order` WHERE `customer_id` = 123 AND `subtotal` > 200 ORDER BY `id`
$orders = $customer->getBigOrders(200)->all();

// SELECT * FROM `order` WHERE `customer_id` = 123 AND `subtotal` > 100 ORDER BY `id`
$orders = $customer->bigOrders;
```

#### 中间关联表

在数据库建模中，当两个关联表之间的对应关系是多对多时， 通常会引入一个[连接表](https://en.wikipedia.org/wiki/Junction_table)。例如，`order` 表 和 `item` 表可以通过名为 `order_item` 的连接表相关联。一个 order 将关联多个 order items， 而一个 order item 也会关联到多个 orders。

当声明这种表关联后，您可以调用 [via()](https://www.yiichina.com/doc/api/2.0/yii-db-activerelationtrait#via()-detail) 或 [viaTable()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#viaTable()-detail) 指明连接表。[via()](https://www.yiichina.com/doc/api/2.0/yii-db-activerelationtrait#via()-detail) 和 [viaTable()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#viaTable()-detail) 之间的区别是 前者是根据现有的关联名称来指定连接表，而后者直接使用 连接表。例如

```php
class Order extends ActiveRecord
{
    public function getItems()
    {
        return $this->hasMany(Item::className(), ['id' => 'item_id'])
            ->viaTable('order_item', ['order_id' => 'id']);
    }
}
```

或者

```php
class Order extends ActiveRecord
{
    public function getOrderItems()
    {
        return $this->hasMany(OrderItem::className(), ['order_id' => 'id']);
    }

    public function getItems()
    {
        return $this->hasMany(Item::className(), ['id' => 'item_id'])
            ->via('orderItems');
    }
}
```

使用连接表声明的关联和正常声明的关联是等同的，例如，

```php
// SELECT * FROM `order` WHERE `id` = 100
$order = Order::findOne(100);

// SELECT * FROM `order_item` WHERE `order_id` = 100
// SELECT * FROM `item` WHERE `item_id` IN (...)
// 返回 Item 类组成的数组
$items = $order->items;
```

#### 通过多个表来连接关联声明

通过使用 [via()](https://www.yiichina.com/doc/api/2.0/yii-db-activerelationtrait#via()-detail) 方法，它还可以通过多个表来定义关联声明。 再考虑考虑上面的例子，我们有 `Customer`，`Order` 和 `Item` 类。 我们可以添加一个关联关系到 `Customer` 类，这个关联可以列出了 `Customer`（客户） 的订单下放置的所有 `Item`（商品）， 这个关联命名为 `getPurchasedItems()`，关联声明如下代码示例所示：

```php
class Customer extends ActiveRecord
{
    // ...

    public function getPurchasedItems()
    {
        // 客户的商品，将 Item 中的 'id' 列与 OrderItem 中的 'item_id' 相匹配
        return $this->hasMany(Item::className(), ['id' => 'item_id'])
                    ->via('orderItems');
    }

    public function getOrderItems()
    {
        // 客户订单中的商品，将 `Order` 的 'id' 列和 OrderItem 的 'order_id' 列相匹配
        return $this->hasMany(OrderItem::className(), ['order_id' => 'id'])
                    ->via('orders');
    }

    public function getOrders()
    {
        // 见上述列子
        return $this->hasMany(Order::className(), ['customer_id' => 'id']);
    }
}
```

#### 延迟加载和即时加载

在 [访问关联数据](https://www.yiichina.com/doc/guide/2.0/db-active-record#accessing-relational-data) 中，我们解释说可以像问正常的对象属性那样 访问 Active Record 实例的关联属性。SQL 语句仅在 你第一次访问关联属性时执行。我们称这种关联数据访问方法为 *延迟加载*。 例如，

```php
// SELECT * FROM `customer` WHERE `id` = 123
$customer = Customer::findOne(123);

// SELECT * FROM `order` WHERE `customer_id` = 123
$orders = $customer->orders;

// 没有 SQL 语句被执行
$orders2 = $customer->orders;
```

延迟加载使用非常方便。但是，当你需要访问相同的具有多个 Active Record 实例的关联属性时， 可能会遇到性能问题。请思考一下以下代码示例。 有多少 SQL 语句会被执行？

```php
// SELECT * FROM `customer` LIMIT 100
$customers = Customer::find()->limit(100)->all();

foreach ($customers as $customer) {
    // SELECT * FROM `order` WHERE `customer_id` = ...
    $orders = $customer->orders;
}
```

你瞅瞅，上面的代码会产生 101 次 SQL 查询！ 这是因为每次你访问 for 循环中不同的 `Customer` 对象的 `orders` 关联属性时，SQL 语句 都会被执行一次。

为了解决上述的性能问题，你可以使用所谓的 *即时加载*，如下所示，

```php
// SELECT * FROM `customer` LIMIT 100;
// SELECT * FROM `orders` WHERE `customer_id` IN (...)
$customers = Customer::find()
    ->with('orders')
    ->limit(100)
    ->all();

foreach ($customers as $customer) {
    // 没有任何的 SQL 执行
    $orders = $customer->orders;
}
```

通过调用 [yii\db\ActiveQuery::with()](https://www.yiichina.com/doc/api/2.0/yii-db-activequerytrait#with()-detail) 方法，你使 Active Record 在一条 SQL 语句里就返回了这 100 位客户的订单。 结果就是，你把要执行的 SQL 语句从 101 减少到 2 条！

你可以即时加载一个或多个关联。 你甚至可以即时加载 *嵌套关联* 。嵌套关联是一种 在相关的 Active Record 类中声明的关联。例如，`Customer` 通过 `orders` 关联属性 与 `Order` 相关联， `Order` 与 `Item` 通过 `items` 关联属性相关联。 当查询 `Customer` 时，您可以即时加载 通过嵌套关联符 `orders.items` 关联的 `items`。

以下代码展示了 [with()](https://www.yiichina.com/doc/api/2.0/yii-db-activequerytrait#with()-detail) 的各种用法。我们假设 `Customer` 类 有两个关联 `orders` 和 `country`，而 `Order` 类有一个关联 `items`。

```php
//  即时加载 "orders" and "country"
$customers = Customer::find()->with('orders', 'country')->all();
// 等同于使用数组语法 如下
$customers = Customer::find()->with(['orders', 'country'])->all();
// 没有任何的 SQL 执行
$orders= $customers[0]->orders;
// 没有任何的 SQL 执行
$country = $customers[0]->country;

// 即时加载“订单”和嵌套关系“orders.items”
$customers = Customer::find()->with('orders.items')->all();
// 访问第一个客户的第一个订单中的商品
// 没有 SQL 查询执行
$items = $customers[0]->orders[0]->items;
```

你也可以即时加载更深的嵌套关联，比如 `a.b.c.d`。所有的父关联都会被即时加载。 那就是, 当你调用 [with()](https://www.yiichina.com/doc/api/2.0/yii-db-activequerytrait#with()-detail) 来 with `a.b.c.d`, 你将即时加载 `a`, `a.b`, `a.b.c` and `a.b.c.d`。

> **提示：** 一般来说，当即时加载 `N` 个关联，另有 `M` 个关联 通过 [连接表](https://www.yiichina.com/doc/guide/2.0/db-active-record#junction-table) 声明，则会有 `N+M+1` 条 SQL 语句被执行。 请注意这样的的嵌套关联 `a.b.c.d` 算四个关联。

当即时加载一个关联，你可以通过匿名函数自定义相应的关联查询。 例如，

```php
// 查找所有客户，并带上他们国家和活跃订单
// SELECT * FROM `customer`
// SELECT * FROM `country` WHERE `id` IN (...)
// SELECT * FROM `order` WHERE `customer_id` IN (...) AND `status` = 1
$customers = Customer::find()->with([
    'country',
    'orders' => function ($query) {
        $query->andWhere(['status' => Order::STATUS_ACTIVE]);
    },
])->all();
```

自定义关联查询时，应该将关联名称指定为数组的键 并使用匿名函数作为相应的数组的值。匿名函数将接受一个 `$query` 参数 它用于表示这个自定义的关联执行关联查询的 [yii\db\ActiveQuery](https://www.yiichina.com/doc/api/2.0/yii-db-activequery) 对象。 在上面的代码示例中，我们通过附加一个关于订单状态的附加条件来修改关联查询。

> **提示：** 如果你在即时加载的关联中调用 [select()](https://www.yiichina.com/doc/api/2.0/yii-db-query#select()-detail) 方法，你要确保 在关联声明中引用的列必须被 select。否则，相应的模型（Models）可能 无法加载。例如，

```php
$orders = Order::find()->select(['id', 'amount'])->with('customer')->all();
// $orders[0]->customer 会一直是 `null`。你应该这样写，以解决这个问题：
$orders = Order::find()->select(['id', 'amount', 'customer_id'])->with('customer')->all();
```

#### 关联关系的 JOIN 查询

> **提示：** 这小节的内容仅仅适用于关系数据库， 比如 MySQL，PostgreSQL 等等。

到目前为止，我们所介绍的关联查询，仅仅是使用主表列 去查询主表数据。实际应用中，我们经常需要在关联表中使用这些列。例如， 我们可能要取出至少有一个活跃订单的客户。为了解决这个问题，我们可以 构建一个 join 查询，如下所示：

```php
// SELECT `customer`.* FROM `customer`
// LEFT JOIN `order` ON `order`.`customer_id` = `customer`.`id`
// WHERE `order`.`status` = 1
// 
// SELECT * FROM `order` WHERE `customer_id` IN (...)
$customers = Customer::find()
    ->select('customer.*')
    ->leftJoin('order', '`order`.`customer_id` = `customer`.`id`')
    ->where(['order.status' => Order::STATUS_ACTIVE])
    ->with('orders')
    ->all();
```

> **提示：** 在构建涉及 JOIN SQL 语句的连接查询时，清除列名的歧义很重要。 通常的做法是将表名称作为前缀加到对应的列名称前。

但是，更好的方法是通过调用 [yii\db\ActiveQuery::joinWith()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#joinWith()-detail) 来利用已存在的关联声明：

```php
$customers = Customer::find()
    ->joinWith('orders')
    ->where(['order.status' => Order::STATUS_ACTIVE])
    ->all();
```

两种方法都执行相同的 SQL 语句集。然而，后一种方法更干净、简洁。

默认的，[joinWith()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#joinWith()-detail) 会使用 `LEFT JOIN` 去连接主表和关联表。 你可以通过 `$joinType` 参数指定不同的连接类型（比如 `RIGHT JOIN`）。 如果你想要的连接类型是 `INNER JOIN`，你可以直接用 [innerJoinWith()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#innerJoinWith()-detail) 方法代替。

调用 [joinWith()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#joinWith()-detail) 方法会默认 [即时加载](https://www.yiichina.com/doc/guide/2.0/db-active-record#lazy-eager-loading) 相应的关联数据。 如果你不需要那些关联数据，你可以指定它的第二个参数 `$eagerLoading` 为 `false`。

> **注意：** 即使在启用即时加载的情况下使用 [joinWith()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#joinWith()-detail) 或 [innerJoinWith()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#innerJoinWith()-detail)， 相应的关联数据也**不会**从这个 `JOIN` 查询的结果中填充。 因此，每个连接关系还有一个额外的查询，正如[即时加载](https://www.yiichina.com/doc/guide/2.0/db-active-record#lazy-eager-loading)部分所述。

和 [with()](https://www.yiichina.com/doc/api/2.0/yii-db-activequerytrait#with()-detail) 一样，你可以 join 多个关联表；你可以动态的自定义 你的关联查询；你可以使用嵌套关联进行 join。你也可以将 [with()](https://www.yiichina.com/doc/api/2.0/yii-db-activequerytrait#with()-detail) 和 [joinWith()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#joinWith()-detail) 组合起来使用。例如：

```php
$customers = Customer::find()->joinWith([
    'orders' => function ($query) {
        $query->andWhere(['>', 'subtotal', 100]);
    },
])->with('country')
    ->all();
```

有时，当连接两个表时，你可能需要在 JOIN 查询的 `ON` 部分中指定一些额外的条件。 这可以通过调用 [yii\db\ActiveQuery::onCondition()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#onCondition()-detail) 方法来完成，如下所示：

```php
// SELECT `customer`.* FROM `customer`
// LEFT JOIN `order` ON `order`.`customer_id` = `customer`.`id` AND `order`.`status` = 1 
// 
// SELECT * FROM `order` WHERE `customer_id` IN (...)
$customers = Customer::find()->joinWith([
    'orders' => function ($query) {
        $query->onCondition(['order.status' => Order::STATUS_ACTIVE]);
    },
])->all();
```

以上查询取出 *所有* 客户，并为每个客户取回所有活跃订单。 请注意，这与我们之前的例子不同，后者仅取出至少有一个活跃订单的客户。

> **提示：** 当通过 [onCondition()](https://www.yiichina.com/doc/api/2.0/yii-db-activequery#onCondition()-detail) 修改 [yii\db\ActiveQuery](https://www.yiichina.com/doc/api/2.0/yii-db-activequery) 时， 如果查询涉及到 JOIN 查询，那么条件将被放在 `ON` 部分。如果查询不涉及 JOIN ，条件将自动附加到查询的 `WHERE` 部分。 因此，它可以只包含 包含了关联表的列 的条件。（译者注：意思是 onCondition() 中可以只写关联表的列，主表的列写不写都行）

#### 关联表别名

如前所述，当在查询中使用 JOIN 时，我们需要消除列名的歧义。因此通常为一张表定义 一个别名。可以通过以下列方式自定义关联查询来设置关联查询的别名：

```php
$query->joinWith([
    'orders' => function ($q) {
        $q->from(['o' => Order::tableName()]);
    },
])
```

然而，这看起来很复杂和耦合，不管是对表名使用硬编码或是调用 `Order::tableName()`。 从 2.0.7 版本起，Yii 为此提供了一个快捷方法。您现在可以定义和使用关联表的别名，如下所示：

```php
// 连接 `orders` 关联表并根据 `orders.id` 排序
$query->joinWith(['orders o'])->orderBy('o.id');
```

上述语法适用于简单的关联。如果在 join 嵌套关联时， 需要用到中间表的别名，例如 `$query->joinWith(['orders.product'])`， 你需要嵌套 joinWith 调用，如下例所示：

```php
$query->joinWith(['orders o' => function($q) {
        $q->joinWith('product p');
    }])
    ->where('o.amount > 100');
```



