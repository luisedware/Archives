---
title: 使用 Sublime Text 3 开发 PHP 的心得
date: 2016-03-13 22:34:20
tags: Sublime Text 3
category: Tooling
---

PHPStorm 不卡之日，SublimeText 舍弃之时

## 环境

- 系统：OSX Yosemite 10.10.5
- 版本：Sublime Text 3 Build 3103


## 安装

使用 Homebrew 安装 Sublime Text 3
```shell
brew install Caskroom/cask/sublime-text
```

## 终端配置

执行以下代码，即可在终端使用 `Sublime Text 3`

```
// 创建程序链接
ln -s  "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl

// 查看使用帮助
subl -h
```



## 快捷键

### 常用

|按键|命令|
|--|---|
|⌘ + C|复制|
|⌘ + ⇧ + D|复制粘贴|
|⌘ + X|剪切|
|⌘ + V|粘贴|
|⌘ + ⇧ + V|保持缩进粘贴|
|⌘ + ⌥ + V|打开粘贴面板|
|⌘ + F|查找|
|⌘ + /|注释行|
|⌘ + ⌥ + /|生成注释|
|⌘ + ⇧ + F|全局查找、替换|
|⌘ + ↩|在光标下插入行|
|⌘ + ⇧ + ↩|在光标上插入行|
|⌘ + ⇧ + t|打开关闭的标签页|
|⌘ + [NUM]|选取对应数字的标签页|
|⌘ + ⇧ + [|上一个标签页|
|⌘ + ⇧ + ]|下一个标签页|
|⌃ + ⇧ + ↑|向上扩展光标|
|⌃ + ⇧ + ↓|向下扩展光标|

### 书签

|按键|命令|
|--|---|
|⌘ + F2|生成或撤销书签|
|⇧ + F2|上一个书签|
|F2|下一个书签|


### 删除

|按键|命令|
|--|---|
|⌃ + ⇧ + K|删除当前行|
|⌘ + K + ⌫| 删除光标之后的内容|
|⌘ + K + ⌘ + K|删除光标之后的内容|

### 跳转

|按键|命令|
|--|---|
|⌘ + P , ⌘ + T|跳转到文件|
|⌘ + R|跳转到方法|
|⌃ + G|跳转到行数|
|⌃ + ⇧ + P|切换工作空间|
|⌘ + ⇧ + P|打开命令面板|
|⌃ + -|向后跳转至修改处|
|⌃ + ⇧ + -|向前跳转至修改处|


### 光标移动

|按键|命令|
|--|---|
|⌃ + P|光标向上移动|
|⌃ + N|光标向下移动|
|⌃ + B|光标向左移动|
|⌃ + F|光标向右移动|
|⌃ + A , ⌘ + ←|光标移动到行最左|
|⌃ + E , ⌘ + →|光标移动到行最右|
|⌃ + M|光标在括号里闭合跳转|
|⌘ + ⌃ + ↑|向上移动选中行|
|⌘ + ⌃ + ↓|向下移动选中行|


### 选取

|按键|命令|
|--|---|
|⌘ + L|选取行|
|⌘ + D|选取一个相同的文本|
|⌃ + ⇧ + G|选取所有相同的文本|
|⌘ + A|选取所有内容|
|⌘ + ⇧ + ↑|向上选取所有内容|
|⌘ + ⇧ + ↓|向下选取所有内容|
|⌘ + ⇧ + K|选取当前行HTML标签|
|⌘ + ⇧ + A|选取HTML标签闭合内容|
|⌘ + ⇧ + J|选取块闭合内容（可以折叠的内容为块）|
|⌃ + ⇧ + M|选取括号内容|
|⌃ + ⇧ + A|选取光标之前内容|
|⌃ + ⇧ + E|选取光标之后内容|

### 窗口

|按键|命令|
|--|---|
|⌘ + K + ⌘ + B|显示左侧栏|
|⌃ + `|显示控制台|
|⌘ + B|执行编译|
|⌃ + ⌘ + F	|全屏模式|
|⌃ + ⇧ + ⌘ + F|无干扰全屏模式|
|⌃ + ⇧ + 2	|进行左右分屏|
|⌃ + ⇧ + 5	|进行上下分屏|
|⌃ + ⇧ + 1	|取消所有分屏|
|⌃ + Num	|根据数字跳转至对应分屏|




## 插件

### PackageControl 插件

按下 `⌃ + ~` 打开控制台，复制粘贴以下代码并按下 `↩`：
```
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

### Material Theme 插件

`Material Theme` 是一款编辑器主题插件

可以使用 `Package Control` 插件安装 `Material-Theme` 插件

`Material-Theme` 插件可提供三种外观样式：
```
//default
"theme": "Material-Theme.sublime-theme",
"color_scheme": "Packages/Material Theme/schemes/Material-Theme.tmTheme",

//Darker
"theme": "Material-Theme-Darker.sublime-theme",
"color_scheme": "Packages/Material Theme/schemes/Material-Theme-Darker.tmTheme",

//Lighter
"theme": "Material-Theme-Lighter.sublime-theme",
"color_scheme": "Packages/Material Theme/schemes/Material-Theme-Lighter.tmTheme",
```

### AdvanceNewFile 插件

`AdvanceNewFile` 是一款快速操作文件的插件

可以使用 `Package Control` 安装 `AdvanceNewFile` 插件

安装完毕之后，可以按下 `⌘ + ⇧ + P` 调出命令面板，执行以下命令对文件进行操作

```
//新增
ANF:New File

//删除
ANF:Delete File

//重命名
ANF:Rename File

//复制当前文件
ANF:Copy Current File

//删除当前文件
ANF:Delete Current File
```

亦可使用快捷键 `⌘ + ⌥ + N` 快速新建文件


### PHP Getters and Setters 插件

`PHP Getters and Setters` 是一款根据类文件的成员属性，快速生成 Getter 方法和 Setter 方法的插件

可以使用 `Package Control` 安装 `PHP Getters and Setters` 插件

执行以下命令快速生成 Getter 和 Setter 的代码
```
// 生成 Getter 方法
PHP:Generate Getters
// 生成 Setter 方法
PHP:Generate Setters

// 同时生成 Getter 和 Setter
PHP:Generate Getters and Setters
```

### PHP Companion 插件

`PHP Companion` 插件提供以下功能

- `find_use`
生成所选类的use声明

- `expand_fqcn`
生成所选类的引用路径

- `import_namespace`
生成所选文件的命名空间

- `goto_definition_scope`
寻找所选代码的定义源头

- `insert_php_constructor_property`
生成当前类的构造方法

在 `Preferences` -> `Key Bindings - User` 新增以下快捷键调用上述插件命令
```json
//PHP Companion
{"keys":["f9"],"command":"find_use"},
{"keys":["f10"],"command":"expand_fqcn"},
{"keys":["f11"],"command":"goto_definition_scope"},
{"keys":["f12"],"command":"insert_php_constructor_property"},
```

### Laravel Artisan 插件

`Laravel Artisan` 是一款让开发可以不使用终端就能运行 `Artisan CLI` 的插件

命令与在终端使用 `php artisan` 基本一致，可以按下 `⌘ + ⇧ + P` 打开命令面板，执行以下命令：

```json
// 生成控制器
Laravel Artisan5:Make:Controller

// 生成请求
Laravel Artisan5:Make:Request

// 生成服务提供者
Laravel Artisan5:Make:Provider

// 生成数据迁移文件
Laravel Artisan5:Make:Mrgiation
```

### DocBlockr 插件

`DocBlockr` 是一款根据代码自动生成注释的插件

### All Autocomplete 插件

`All Autocomplete` 是一款自动补全的插件，它会查找你打开的所有文件的代码，然后进行代码补全


### SublimeLinter-php 插件

`SublimeLinter` 是一款检查PHP语言错误的插件

使用 `Package Control` 安装 `SublimeLinter` 插件
使用 `Package Control` 安装 `SublimeLinter-php` 插件

在 `Preferences` -> `Package Settings` -> `SublimeLinter` -> `Settings - User` 新增以下代码：

```
"show_errors_on_save": true,
```
上述操作会让编辑器在保存文件的时候，提示错误信息



## 代码片段

`Sublime Text` 可以创建自定义的代码片段

点击菜单栏 `Tools` -> `New Snippet` ，创建文件 `PHP Public Method.sublime-snippet`
```
<snippet>
	<content><![CDATA[
public function ${1:function_name}(${2:param}){
	${3:return result}
}
]]></content>
	<!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
	<tabTrigger>pubf</tabTrigger>
	<!-- Optional: Set a scope to limit where the snippet will trigger -->
	<scope>source.php</scope>
</snippet>
```
当在PHP文件写代码时，输入 `pubf` 并按下 `Tab` 键，会生成相对应的代码片段
```
public function function_name(param){
	return true
}
```


## 格式化代码

在终端执行以下命令安装 `php-cs-fix`
```shell
brew install php-cs-fixer
```

假如要对某个 PHP 文件进行格式化操作，运行以下命令：
```
cd dir
php-cs-fixer fix fileName.php --level="psr2"
```

使用 `Sublime Text 3` 的 `Build System` 格式化代码
点击 `Tools` -> `Build System` -> `New Bulid System ...`，打开代码如下：
```
    "shell_cmd":"make"
```
修改上述代码如下：
```
{
	"shell_cmd": "php-cs-fixer fix $file --level=psr2"
}
```
保存之后，在 `Tools`-`Build System` 选择 `PSR-2`，按 `⌘ + B `对当前文件进行 `PSR-2` 格式化操作。

觉得格式化提示信息麻烦，可以在`Preferences`-`Settings - User`添加
```
"show_panel_on_build":false
```





