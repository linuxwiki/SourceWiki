世界上有三中程序员，一种是用 Vim 的，一种是用 Emacs 的，另外一种是用其它编辑器的。本文介绍 Emacs 的基本使用。

# 一、Emacs24 的安装

    wget https://github.com/mirrors/emacs/archive/emacs-24.3.92.zip
    unzip emacs-24.3.92.zip
    cd emacs-24.3.93
    ./autogen.sh
    ./configure --without-makeinfo
    make -j2
    sudo make install

# 二、基本快捷键

这里列出在没有打开任何 mode 的情况下，原生 Emacs 所支持的快捷键(快捷键映射关系: C->Ctrl, M->Alt/ESC)。

`C-g` : 撤销命令(没事多摁几下，我都是没事摁着玩的)
`C-z` : 挂起 Emacs
`C-x C-c`: 关闭 Emacs

## 2.1 光标移动

+ `C-n` : 下一行
+ `C-p` : 上一行
+ `C-f` : 前进一个字符
+ `C-b` : 后退一个字符
+ `M-f` : 前进一个单词
+ `M-b` : 后退一个单词
+ `C-v` : 向下翻页
+ `M-v` : 向上翻页
+ `C-l` : 光标所在行移动到屏幕中央
+ `C-a` : 行首
+ `C-e` : 行尾
+ `M-a` : 句首
+ `M-e` : 句尾
+ `M-<` : 文件首
+ `M->` : 文件尾

## 2.2 文档编辑

+ `<Backspace>` : 向前删除一个字符
+ `C-d` `<DEL>` : 向后删除一个字符，我一般用 `C-d`
+ `M-d` `M-<DEL>` : 向后删除一个单词
+ `C-k` : 删除光标到行尾，删除一行用 `C-a, C-k` (Vim 中的 dd)。
+ `M-k` : 删除光标到句尾
+ `C-@` : 标记(Mark set), 标记之后配合光标移动快捷键，可以选中一个区域
+ `M-w` : 复制
+ `C-w` : 剪切
+ `C-y` : 粘贴
+ `C-/` : 撤销(UNDO)

大小写转换:

+ `M-l`: Convert following word to lower case (downcase-word).
+ `M-u`: Convert following word to upper case (upcase-word).
+ `M-c`: Capitalize the following word (capital-word).
+ `C-x C-l`: Convert region to lower case (downcase-region).
+ `C-x C-u`: Convert region to upper case (upcase-region).

块操作:

+ `C-x r k`: Kill the text of the region-rectangle, saving its contents as the “last killed rectangle” (kill-rectangle).
+ `C-x r M-w`: Save the text of the region-rectangle as the “last killed rectangle” (copy-rectangle-as-kill).
+ `C-x r d`: Delete the text of the region-rectangle (delete-rectangle).
+ `C-x r y`: Yank the last killed rectangle with its upper left corner at point (yank-rectangle).
+ `C-x r o`: Insert blank space to fill the space of the region-rectangle (open-rectangle). This pushes the previous contents of the region-rectangle to the right.
+ `C-x r N`: Insert line numbers along the left edge of the region-rectangle (rectangle-number-lines). This pushes the previous contents of the region-rectangle to the right.
+ `C-x r c`: Clear the region-rectangle by replacing all of its contents with spaces (clear-rectangle).
+ `M-x delete-whitespace-rectangle`: Delete whitespace in each of the lines on the specified rectangle, starting from the left edge column of the rectangle.
+ `C-x r t string <RET>`: Replace rectangle contents with string on each line (string-rectangle).
+ `M-x string-insert-rectangle <RET> string <RET>`: Insert string on each line of the rectangle.

## 2.3 搜索

+ `C-s` : 向前搜索
+ `C-r` : 向后操作
+ `C-g` : 回到搜索开始前的位置
+ `M-%` : 询问并替换

## 2.4 文件操作(缓冲区)

Emacs 可以打开很多文件，一个文件可以理解成一个 buffer，你可以在多个文件中来回切换。

+ `C-x f` : 打开一个文件
+ `C-x s` : 保存
+ `C-x C-w` : 另存为
+ `C-x C-v` : 关闭当前缓冲区，并打开新文件
+ `C-x C-b` : 打开缓冲区列表
+ `C-x b` : 切回最近的 buffer
+ `C-x k` : 关闭当前缓冲区

## 2.5 窗口(WINDOWS)操作

+ `C-x 1` : 只保留当前窗口
+ `C-x 2` : 水平切分
+ `C-x 3` : 垂直切分
+ `C-x o` : 切换到另外一个窗口
+ `C-x 0` : 关闭当前窗口(并不是删除buffer)

# 三、通用定制

默认的 Emacs 的快捷键有些不好用，比如 `C-x C-f` 时，没有任何提示。再如窗口切换使用 `C-x o`，当 Buffer 多时，非常不方便，这一节介绍 Emacs 通用的定制(不针对某种开发/编辑环境)。

Emacs 的配置文件为 `~/.emacs`，如果没有就 `touch` 一个，插件位置一般在 `~/.emacs.d`，如果没有就 `mkdir` 一个。Emacs 启动时会加载这些配置(也因此会减慢Emacs的启动速度，建议不要给Emacs装太多的插件，看着花哨但不实用)。

用 `y/n` 代替 `yes/no`:

    (fset 'yes-or-no-p 'y-or-n-p)

设置编码为 UTF-8

    (setq locale-coding-system 'utf-8)
    (set-terminal-coding-system 'utf-8)
    (set-keyboard-coding-system 'utf-8)
    (set-selection-coding-system 'utf-8)
    (prefer-coding-system 'utf-8)

`C-c e` 打开 eshell, `C-c l`清空 eshell:

    (defun clear-eshell-buffer ()
      (interactive)
      (let ((inhibit-read-only t))
        (delete-region (point-min) (point-max))))
    (global-set-key (kbd "C-c l") 'clear-eshell-buffer)
    (global-set-key (kbd "C-c e") 'eshell)

tab -> 空格:

    (setq-default indent-tabs-mode nil)

括号匹配、对齐、补全：

    (show-paren-mode t)
    (require 'electric)
    (electric-indent-mode t)
    (electric-pair-mode t)
    (electric-layout-mode t)
    

保存时删除多余的空白字符:
    
    (add-hook 'before-save-hook 'delete-trailing-whitespace)
    (setq show-trailing-whitespace t)

去掉自动保存和备份:

    (setq auto-save-default nil)
    (setq make-backup-files nil)


行号:

    (setq linum-format "%3d|")
    (global-linum-mode 1)

自动换行：

    (global-visual-line-mode 1)
    (blink-cursor-mode -1)

使用 M-(1,2,3...9)窗口切换(依赖于 windows-numbering 插件):

    (add-to-list 'load-path "~/.emacs.d/lisp/window-numbering.el")
    (require 'window-numbering)
    (setq window-numbering-assign-func
          (lambda () (when (equal (buffer-name) "*Calculator*") 9)))
    (window-numbering-mode 1)

Mini Buffer 优化(依赖 smex 插件，ido 是 Emacs 自带的):

    ;; C-x f/b
    (require 'ido)
    (ido-mode t)
    ;; M-x
    (add-to-list 'load-path "~/.emacs.d/lisp/smex")
    (require 'smex)
    (smex-initialize)
    (global-set-key (kbd "M-x") 'smex)
    (global-set-key (kbd "M-X") 'smex-major-mode-commands)

Tip:  `M-x eval-buffer` 可以使配置文件立即生效，调试非常方便。

# 四、高级定制

这一节针对不同的应用，介绍一些插件。

## 4.1 代码参考线: [fill-column-indicator](https://github.com/alpaker/Fill-Column-Indicator)

    (require 'fill-column-indicator)
    (setq fci-rule-color "#333") # 参考线颜色，我的配色是暗色的, 所以 #333 看着舒服一点
    (setq fci-rule-column 80)    # 宽度设置为 80 
    (define-globalized-minor-mode
      global-fci-mode fci-mode (lambda () (fci-mode 1)))
    (global-fci-mode 1)

## 4.2 代码自动补全: [auto-complete](https://github.com/auto-complete/auto-complete)

对于用管IDE的朋友,我要解释一下,这里的自动补全不是类成员提示, =_=!

    (require 'popup)
    (require 'auto-complete-config)
    (add-to-list 'ac-dictionary-directories
                 "~/.emacs.d/auto-complete/dict")
    (ac-config-default)
    (setq ac-use-quick-help nil)
    (setq ac-ignore-case t)
    (setq ac-menu-height 8)
    
    (setq ac-use-menu-map t)
    (define-key ac-menu-map "\C-n" 'ac-next)
    (define-key ac-menu-map "\C-p" 'ac-previous)
    (global-set-key "\M-/" 'auto-complete)

auto-complete 和 [yasnippet](https://github.com/capitaomorte/yasnippet) 是一对好基友，yasnippet 支持很多语言的语法补全，感兴趣可以尝试一下(我不太喜欢用，感觉会影响加载速度)。

## 4.3 相同符号高亮: [highlight-symbol](https://github.com/nschum/highlight-symbol.el)

用这个做查找也挺不错的!

    (require 'highlight-symbol)
    (global-set-key (kbd "M--") 'highlight-symbol-at-point)
    (global-set-key (kbd "M-n") 'highlight-symbol-next)
    (global-set-key (kbd "M-p") 'highlight-symbol-prev)

## 4.4 Markdown: [markdown-mode](http://jblevins.org/projects/markdown-mode/)

    (autoload 'markdown-mode "~/.emacs.d/lisp/markdown-mode/markdown-mode.el"
      "Major mode for editing Markdown files" t)
    (setq auto-mode-alist
          (cons '("\\.md" . markdown-mode) auto-mode-alist))
    (setq auto-mode-alist
          (cons '("\\.markdown" . markdown-mode) auto-mode-alist))
    (setq auto-mode-alist
          (cons '("\\.txt" . markdown-mode) auto-mode-alist))

## 4.5 Python: [python.el](https://github.com/fgallina/python.el)

推荐使用 python.el 而不是 [python-mode](https://github.com/klen/python-mode)。

    (require 'python)

## 4.6 Google Protobuf Buffer: [protobuf-mode](http://code.google.com/p/protobuf/source/browse/trunk/editors/protobuf-mode.el?r=227)

    (require 'protobuf-mode)
    (add-to-list 'auto-mode-alist '("\\.proto$" . protobuf-mode))

## 4.7 C++开发定制: xcscope + etags + c++-mode

    ;; 编译与调试
    (global-set-key [(f5)] 'compile)
    
    (require 'xcscope)
    (cscope-setup)
    
    (global-set-key [(f9)] 'ff-find-other-file)
    
    (setq tab-stop-list ())
    (loop for x downfrom 40 to 1 do
          (setq tab-stop-list (cons (* x 4) tab-stop-list)))
    
    (defconst my-c-style
      '(
        (c-tab-always-indent        . t)
        (c-hanging-braces-alist     . ((substatement-open after)
                                       (brace-list-open)))
        (c-hanging-colons-alist     . ((member-init-intro before)
                                       (inher-intro)'
                                       (label after)
                                       (acecss-label after)))
        (c-cleanup-list             . (scope-operator
                                       empty-defun-braces
                                       defun-close-semi))
        (c-offsets-alist            . ((arglist-close . c-lineup-arglist)
                                       (case-label . 4)
                                       (substatement-open . 0)
                                       (block-open        . 0)
                                       (knr-argdecl-intro . -)
                                       (innamespace . -)
                                       (inline-open . 0)
                                       (inher-cont . c-lineup-multi-inher)
                                       (arglist-cont-nonempty . +)
                                       (template-args-cont . + )))
        (c-echo-syntactic-information-p . t)
        )
      "My C Programming Style")
    
    ;; offset customizations not in my-c-style
    (setq c-offsets-alist '((member-init-intro . ++)))
    
    ;; Customizations for all modes in CC Mode.
    (defun my-c-mode-common-hook ()
      ;; add my personal style and set it for the current buffer
      (c-add-style "PERSONAL" my-c-style t)
      ;; other customizations
      (setq tab-width 4
            indent-tabs-mode nil)
      ;; we like auto-newline and hungry-delete
      ;; (c-toggle-auto-hungry-state 1)
      ;; key bindings for all supported languages.  We can put these in
      ;; c-mode-base-map because c-mode-map, c++-mode-map, objc-mode-map,
      ;; java-mode-map, idl-mode-map, and pike-mode-map inherit from it.
      )
    (add-hook 'c-mode-common-hook 'my-c-mode-common-hook)
    (add-hook 'c-mode-hook 'hs-minor-mode)
    
    (add-to-list 'auto-mode-alist '("\\.h\\'" . c++-mode))
    
    ;; http://stackoverflow.com/questions/14668744/emacs-indent-for-c-class-method
    (defun vlad-cc-style()
      (c-set-offset 'inline-open '0)
      )
    (add-hook 'c++-mode-hook 'vlad-cc-style)

[xcscope.el](https://github.com/dkogan/xcscope.el) 使用:

+ cscope -r : 在根目录下递归生成数据库
+ C-c s a : Set initial directory;
+ C-c s A : Unset initial directory;
+ C-c c I : create list of files to index;
+ C-c s s : Find symbol;
+ C-c s d : Find global definition;
+ C-c s c : Find functions calling a function;
+ C-c s C : Find called functions (list functions called from a function);
+ C-c s t : Find text string;
+ C-c s e : Find egrep pattern;
+ C-c s f : Find a file;
+ C-c s i : Find files #including a file;
+ C-c s b : Display cscope buffer;

etags 使用:

搭配 `find` 来生成 tags:

    find . -name '*.c' -o -name '*.cpp' -o -name '*.h' -print | etags -

+ `M-x visit-tags-table <RET> FILE <RET>` : 选择tags文件
+ `M-. [TAG] <RET>` : 访问标签
+ `M-*` : 返回
+ `C-u M-.` : 寻找标签的下一个定义

其它：

+ `F9` : 在头文件和对应源文件之间跳转: `(global-set-key [(f9)] 'ff-find-other-file)`

## 4.8 [谷歌翻译](https://github.com/atykhonov/google-translate.git)


    (add-to-list 'load-path "~/.emacs.d/lisp/google-translate")
    (require 'google-translate)
    (require 'google-translate-smooth-ui)
    (global-set-key "\C-ct" 'google-translate-smooth-translate)
    (setq google-translate-translation-directions-alist
          '(("en" . "zh-CN") ("zh-CN" . "en") ))

`C-c t` 打开翻译，我指定了英->中，中->英两种翻译模式。

## 4.9 Emacs主题

把 主题 放到最后，是想告诉大家，使用 Emacs(或者其它任何工具) 时，不要花时间在这些炫酷的东西上面，还是要聚焦于实用和高效。Emacs24自带了几款主题(Emacs23没有的哦)，使用 `M-x customize-theme` 回车可查看配色效果。确定喜欢的一款注意后在配置文件中添加一行代码就可以啦。

我一般使用 wombat ，即 `(load-theme 'wombat)`。

# 五、学习资源 

## 5.1 视频

+ [Emacs as a C/C++ Editor/IDE (Part 3): cedet mode for true intellisense](http://www.youtube.com/watch?v=Ib914gNr0ys&feature=share)

## 5.2 网站(博客)

+ [MELPA](http://melpa.milkbox.net/) : N多插件等你选，同时也可以感受一下 Emacs 的强大。
+ [GNU Emacs Manuals Online](http://www.gnu.org/software/emacs/manual/)
+ [Emacs Redux](http://emacsredux.com/)
+ [Emacs Markdown Mode](http://jblevins.org/projects/markdown-mode/)

## 5.3 参考/扩展资料

+ [一年成为Emacs高手(像神一样使用编辑器)](https://github.com/redguardtoo/mastering-emacs-in-one-year-guide/blob/master/guide-zh.org)
+ [Emacs as a Python IDE](http://www.jesshamrick.com/2012/09/18/emacs-as-a-python-ide/)
+ [Emacs快速参考](http://jianlee.ylinux.org/Computer/Emacs/emacsBFE99F8FE883.html)
+ [Case Conversion Commands](http://www.gnu.org/software/emacs/manual/html_node/emacs/Case.html)
+ [Rectangles](http://www.gnu.org/software/emacs/manual/html_node/emacs/Rectangles.html)
+ [曹乐: 在Emacs下用C/C++编程](http://www.caole.net/diary/emacs_write_cpp.html)
