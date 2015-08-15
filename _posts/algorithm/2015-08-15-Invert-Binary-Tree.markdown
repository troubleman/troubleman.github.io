---
layout: post
title:  "算法之翻转二叉树"
date:   2015-08-15 17:35
categories: 算法
tags: algorithm invertBinaryTree
---

2015 年 6 月 10 日，Homebrew 的作者 @Max Howell 在 twitter 上发表了如下一内容：

> Google: 90% of our engineers use the software you wrote (Homebrew), but you can’t invert a binary tree on a whiteboard so fuck off.

题目要求（简单介绍）：
--------

翻转前：

         4
       /   \
      2     7
     / \   / \
    1   3 6   9


翻转后：

         4
       /   \
      7     2
     / \   / \
    9   6 3   1

解题思路：
---------

这道题主要是考查了递归的思想。

1.翻转左右子树
2.将左右子树交换

第一步和第二步解决过程一样，所以用递归解决。注意递归条件和结束条件。

具体步骤：
----------

用php数组来模拟数据结构，如下：

```php
    $root = array(
        'val'   =>4,
        'left'  =>array('val'=>2,'left'=>1,'right'=>3),
        'right' =>array('val'=>7,'left'=>6,'right'=>9)
    );
```

有了数据的结构接下来就是具体的函数实现了：

```php

function invertTree($root){
    if(!empty($root['val'])){
        $tmp = invertTree($root['left']);
        $root['left']  = invertTree($root['right']);
        $root['right'] = $tmp;

    }
    return $root;
}
```

测试一下：

```php

echo "翻转前：";
print_r($root);
echo "翻转后：";
print_r(invertTree($root));

```

结果：

    翻转前：
    Array
    (
        [val] => 4
        [left] => Array
            (
                [val] => 2
                [left] => 1
                [right] => 3
            )

        [right] => Array
            (
                [val] => 7
                [left] => 6
                [right] => 9
            )

    )

    翻转后：
    Array
    (
        [val] => 4
        [left] => Array
            (
                [val] => 7
                [left] => 9
                [right] => 6
            )

        [right] => Array
            (
                [val] => 2
                [left] => 3
                [right] => 1
            )

    )

总结
-----
多多学习算法知识和数据结构！！！