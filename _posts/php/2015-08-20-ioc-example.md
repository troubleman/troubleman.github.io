---
layout: post
title: IOC容器实例讲解
date:   2015-08-20 14:26
category: php
tags: IOC
keywords: IOC 控制器反转
description:
---
## 原文地址
[Dependency Injection: Huh? -- Jeffrey Way](http://code.tutsplus.com/tutorials/dependency-injection-huh--net-26903)

## 相关文章
* [Learning about Dependency Injection and PHP](http://ralphschindler.com/2011/05/18/learning-about-dependency-injection-and-php)
* [What is Dependency Injection?](http://fabien.potencier.org/what-is-dependency-injection.html)
* [Dependency Injection: An analogy](https://mwop.net/blog/260-Dependency-Injection-An-analogy.html)
* [Dependency Injection: Huh?](http://code.tutsplus.com/tutorials/dependency-injection-huh--net-26903)
* [Dependency Injection as a tool for testing](http://philipobenito.github.io/dependency-injection-as-a-tool-for-testing/)


#### 先看一个例子

{% highlight ruby %}
class Photo {
    /**
     * @var PDO The connection to the database
     */
    protected $db;
 
    /**
     * Construct.
     */
    public function __construct()
    {
        $this->db = DB::getInstance();
    }
}
{% endhighlight %}

上面这种写法是我们比较常见的一种写法：在Photo 类中通过`DB::getInstance();`获得一个数据库实例，方便进行数据库操作。看着没什么问题，其实，我们硬编码（hardcoded ）了一个数据依赖，这样造成了`Photo`类和`DB`耦合比较高，如果我们现在要换一个数据库该怎么解决？可能需要修改此处的实例，但是这只是一个Photo类，如果还有其他许许多多类都是通过这种方式进行数据库连接怎么办？一个一个替换显然不是最优的解决方法。这里我们需要了解注入的概念，介绍两种注入的实现`Constructor Injection `和`setter injection`。

Constructor Injection

{% highlight ruby %}
class Photo {
    /**
     * @var PDO The connection to the database
     */
    protected $db;
 
    /**
     * Construct.
     * @param PDO $db_conn The database connection
     */
    public function __construct($dbConn)
    {
        $this->db = $dbConn;
    }
}
 
$photo = new Photo($dbConn);
{% endhighlight %}

上面我们在实例`Photo`对象的时候，把需要的依赖注入到类中，这样就把依赖和类的实现分离开来了，比硬编码（hardcoding）好多了。

Setter Injection

{% highlight ruby %}
class Photo {
    /**
     * @var PDO The connection to the database
     */
    protected $db;
 
    public function __construct() {}
 
    /**
     * Sets the database connection
     * @param PDO $dbConn The connection to the database.
     */
    public function setDB($dbConn)
    {
        $this->db = $dbConn;
    }
}
 
$photo = new Photo;
$photo->setDB($dbConn);
{% endhighlight %}

看上去也不错，现在问题来了。如果我们有不止一个依赖，按照上门的方法，我们可能需要这么写：

{% highlight ruby %}
$photo = new Photo;
$photo->setDB($dbConn);
$photo->setConfig($config);
$photo->setResponse($response);
{% endhighlight %}

看着有一丝丝不爽，这里就需要了解一下容器的概念了。`IoC容器` 或 `DI容器`。看定义：

> In software engineering, Inversion of Control (IoC) is an object-oriented programming practice where the object coupling is bound at run time by an assembler object and is typically not known at compile time using static analysis.


直接上代码

{% highlight ruby %}
class Ioc {
/**
* @var 注册的依赖数组
*/
 
   protected static $registry = array();
 
   /**
    * 添加一个resolve到registry数组中
    * @param  string $name 依赖标识
    * @param  object $resolve 一个匿名函数用来创建实例
    * @return void
    */
   public static function register($name, Closure $resolve)
   {
      static::$registry[$name] = $resolve;
   }
 
   /**
     * 返回一个实例
     * @param  string $name 依赖的标识
     * @return mixed
     */
   public static function resolve($name)
   {
       if ( static::registered($name) )
       {
          $name = static::$registry[$name];
          return $name();
       }
       throw new Exception('Nothing registered with that name, fool.');
   }
   /**
    * 查询某个依赖实例是否存在
    * @param  string $name id
    * @return bool 
    */
   public static function registered($name)
   {
      return array_key_exists($name, static::$registry);
   }
}
{% endhighlight %}

现在就可以通过如下方式来注册和注入一个依赖

{% highlight ruby %}
$book = Ioc::registry('photo', function(){
    $book = new Photo;
    $book->setdb('...');
    $book->setprice('...');
    return $photo;
});
{% endhighlight %}

这里，我们把`photo`类所需要的所有依赖注册到容器中，并建立关系，使用的时候只要：

{% highlight ruby %}
//注入依赖
$photo = Ioc::resolve('photo');
{% endhighlight %}

就可以获得一个实例了。还有，因为注册的时候传递的是一个闭包类（Closure），所以注册的时候并不会实例photo,只有在我们调用（resolve）的时候才会实例化。

关于依赖注入，需要好好理解