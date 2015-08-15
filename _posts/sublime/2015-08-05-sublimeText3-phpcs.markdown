---
layout: post
title:  "Sublime Text 3配置php语法错误提示插件PHPCS(win7)"
date:   2015-08-05 10:49
categories: sublime
tags: php
---
> 本文记录sublime text3 配置php语法错误提示插件 PHP_CodeSniffer的安装


## 第一步 下载安装插件

使用 Sublime Text’s Package Control 搜索 phpcs，安装

[插件地址-sublime-phpcs](http://benmatselby.github.io/sublime-phpcs/)

## 第二步 下载 `php-cs-fixer.phar`

[点击下载-php-cs-fixer.phar](http://cs.sensiolabs.org/get/php-cs-fixer.phar)
下载完成后，将`php-cs-fixer.phar`放入你的php.exe安装目录。
（例如我的php.exe安装目录是`E:/wamp/bin/php/php5.4.16/php.exe`）

## 第三步 下载 `PHP_CodeSniffer`

[点击下载PHP_CodeSniffer](http://download.pear.php.net/package/PHP_CodeSniffer-1.5.0RC4.tgz)
下载完成后，解压该压缩包，然后复制文件夹 `PHP_CodeSniffer-1.5.0RC4`到合适的目录（我的存放目录是`E:\wamp\bin\PHP_CodeSniffer-1.5.0RC4`），修改`E:\wamp\bin\PHP_CodeSniffer-1.5.0RC4\scripts\phpcs.bat`文件：

1. `@php_bin@` 替换为本地PHP执行文件路径：`E:\wamp\bin\php\php5.4.16\php.exe`
2. `@php_dir@` 替换为PHP CodeSniffer目录：`E:\wamp\bin\PHP_CodeSniffer-1.5.0RC4`
3. `@bin_dir@\phpcs` 替换为phpcs脚本路径：`E:\wamp\bin\PHP_CodeSniffer-1.5.0RC4\scripts\phpcs`

`注：以上是我本地环境下的路径，请参照自己实际路径进行修改`

## 第四步  修改sublime text3 插件配置

打开`Sublime Text 3->Preferences->Package Settings -> Php Code Sniffer -> Settings - User `,
复制以下配置，修改其中php.exe的安装路径,改为你的php.exe实际安装路径

    {

        // Path to php on windows installation
        // This is needed as we cannot run phars on windows, so we run it through php
        "phpcs_php_prefix_path": "E:\\wamp\\bin\\php\\php5.4.16\\php.exe",

        // This is the path to the bat file when we installed PHP_CodeSniffer
        "phpcs_executable_path": "E:\\wamp\\bin\\PHP_CodeSniffer-1.5.0RC4\\scripts\\phpcs.bat",

        // PHP-CS-Fixer settings
        // Don't want to auto fix issue with php-cs-fixer
        "php_cs_fixer_on_save": false,

        // Show the quick panel
        "php_cs_fixer_show_quick_panel": true,

        // The fixer phar file is stored here:
        "php_cs_fixer_executable_path": "E:\\wamp\\bin\\php\\php5.4.16\\php-cs-fixer.phar",

        // PHP Linter settings
        // Yes, lets lint the files
        "phpcs_linter_run": true,

        // And execute that on each file when saved (php only as per extensions_to_execute)
        "phpcs_linter_command_on_save": true,

        // Path to php
        "phpcs_php_path": "E:\\wamp\\bin\\php\\php5.4.16\\php.exe",

        // This is the regex format of the errors
        "phpcs_linter_regex": "(?P<message>.*) on line (?P<line>\\d+)",


        // PHP Mess Detector settings
        // Not turning on the mess detector here
        "phpmd_run": false,
        "phpmd_command_on_save": false,
        "phpmd_executable_path": "",
        "phpmd_additional_args": {}
    }


## 最后 测试效果
用sublimeText3随便打开一个php程序文件，右键选择`php Code Sniffer > sniff this file`，如果不符合规范就可以看到错误提示了
![](/assets/img/sublime/phpcs.png)