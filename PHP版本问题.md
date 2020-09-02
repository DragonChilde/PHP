# PHP7.2

## 废弃each()

使用代替

```php
foreach()
```

## 废弃create_function()

使用代替:**匿名函数**

```php
$greet = function($name1,$name2)
{
    printf("Hello %s\r%s\n", $name1,$name2);
};

$greet('World','PHP');	//Hello World PHP
```

