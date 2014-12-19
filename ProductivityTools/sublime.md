# Sublime Text2

## 设置 tab 为 4 个空格

* Preferences/Settings - Default

```
// The number of spaces a tab is considered equal to
"tab_size": 4,

// Set to true to insert spaces when tab is pressed
"translate_tabs_to_spaces": true,
```

### Reference

* [Settings](https://www.sublimetext.com/docs/2/settings.html)

## 设置 vim 模式

* Preferences/Settings - Default

```
"ignored_packages": ["Vintage"]
```

to:

```
"ignored_packages": []
```

* Preferences/Settings - User

默认打开文件进入 insert 模式，可设置默认打开文件为 command 模式

add:

```
"vintage_start_in_command_mode": true
```

### Reference

* [Vintage Mode](https://www.sublimetext.com/docs/2/vintage.html)

## 列编辑模式

* 鼠标右键 + Shift OR 鼠标中键
* Add to selection: Ctrl

### Reference

* [Column Selection](https://www.sublimetext.com/docs/2/column_selection.html)

## Markdown Preview 插件

* Use Ctrl+Shift+P then Package Control: Install Package
* Look for Markdown Preview and install it.

### Reference

* [Markdown Preview](https://github.com/revolunet/sublimetext-markdown-preview)

## 撤销恢复

* Ctrl+Z 撤销
* Ctrl+Y 恢复撤销

## 查找替换

* Ctrl+F 查找
* Ctrl+H 替换

## Ctrl+P

* Ctrl+P 快速跳转当前项目中的任意文件
* Ctrl+P 输入 @ 快速跳转 [等同于 Ctrl+r]
* Ctrl+P 输入 # 当前文件搜索 [等同于 Ctrl+f]
* Ctrl+P 输入 : 跳转到指定行

## 命令调用

* Ctrl+Shift+P
    * 命令调用，如之前 Markdown Preview 安装就调用了 Package Control: Install Package。

## Console

* Ctrl+`

