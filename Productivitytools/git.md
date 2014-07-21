# 分布式管理系统 Git


## 知识点

基本命令让你快速的上手使用Git，知识点能让你更好的理解Git。

### 快照和差异

详细可看：[Pro Git: Git基础](http://iissnan.com/progit/html/zh/ch1_3.html)中有讲到 *直接记录快照，而非差异比较*，这里只讲我个人的理解。

Git 关心的是文件数据整体的变化，其他版本管理系统(以svn为例)关心的某个具体文件的*差异*。这个差异是好理解的，也就是两个版本具体文件的不同点，比如某一行的某个字符发生了改变。

Git 不保存文件提交前后的差异，不变的文件不会发生任何改变，对于变化的文件，前后两次提交则保存两个文件。举个例子：

SVN:

1. 新建3个文件a, b, c，做第一次提交 ->  `version1 : file_a file_b file_c`
2. 修改文件 b， 做第二次提交(真正提交的是 修改后的文件 b 和修改前的 `file_b` 的 diff) -> `version2: diff_b_2_1`
3. 当我要 checkout version2 的时候，实际上得到的是 `file_a file_b+diff_b_2_1 file_c`

Git:

1. 新建3个文件a, b, c，做第一次提交 ->  `version1 : file_a file_b file_c`
2. 修改文件 b (得到`file_b1`), 做第二次提交 -> `version2: file_a file_b1 file_c` 
3. 当我要用 version2 的时候，实际上得到的是 `file_a file_b1 file_c` 

上面的 `file_a file_b1 file_c` 就是 version2 的 *快照*。

### Git数据结构

Git的核心数是很简单的，就是一个链表(或者一棵树更准确一些？无所谓了)，一旦你理解了它的基本数据结构，再去看Git，相信你有不同的感受。继续用上面的例子(所有的物理文件都对应一个 SHA-1 的值)

当我们做第一次提交时，数据结构是这样的:


    sha1_2_file_map:
        28415f07ca9281d0ed86cdc766629fb4ea35ea38 => file_a
        ed5cfa40b80da97b56698466d03ab126c5eec5a9 => file_b
        1b5ca12a6cf11a9b89dbeee2e5431a1a98ea5e39 => file_c
    
    commit_26b985d269d3a617af4064489199c3e0d4791bb5:
        base_info:
            Auther: "JerryZhang(chinajiezhang@gmail.com)"
            Date: "Tue Jul 15 19:19:22 2014 +0800"
            commit_content: "第一次提交"
        file_list:
            [1]: 28415f07ca9281d0ed86cdc766629fb4ea35ea38
            [2]: ed5cfa40b80da97b56698466d03ab126c5eec5a9
            [3]: 1b5ca12a6cf11a9b89dbeee2e5431a1a98ea5e39
        pre_commit: null
        next_commit: null

当修改了 `file_b`, 再提交一次时，数据结构应该是这样的:

    sha1_2_file_map:
        28415f07ca9281d0ed86cdc766629fb4ea35ea38 => file_a
        ed5cfa40b80da97b56698466d03ab126c5eec5a9 => file_b
        1b5ca12a6cf11a9b89dbeee2e5431a1a98ea5e39 => file_c
        39015ba6f80eb9e7fdad3602ef2b1af0521eba89 => file_b1
    
    commit_26b985d269d3a617af4064489199c3e0d4791bb5:
        base_info:
            Auther: "JerryZhang(chinajiezhang@gmail.com)"
            Date: "Tue Jul 15 19:19:22 2014 +0800"
            commit_content: "第一次提交"
        file_list:
            [1]: 28415f07ca9281d0ed86cdc766629fb4ea35ea38
            [2]: ed5cfa40b80da97b56698466d03ab126c5eec5a9
            [3]: 1b5ca12a6cf11a9b89dbeee2e5431a1a98ea5e39
    pre_commit: commit_a08a57561b5c30b9c0bf33829349e14fad1f5cff
    next_commit: null
    
    commit_a08a57561b5c30b9c0bf33829349e14fad1f5cff:
        base_info:
            Auther: "JerryZhang(chinajiezhang@gmail.com)"
            Date: "Tue Jul 15 22:19:22 2014 +0800"
            commit_content: "更新文件b"
        file_list:
            [1]: 28415f07ca9281d0ed86cdc766629fb4ea35ea38
            [2]: 39015ba6f80eb9e7fdad3602ef2b1af0521eba89
            [3]: 1b5ca12a6cf11a9b89dbeee2e5431a1a98ea5e39
    pre_commit: null
    next_commit: commit_26b985d269d3a617af4064489199c3e0d4791bb5

当提交完第二次的时候，执行 `git log`，实际上就是从 `commit_a08a57561b5c30b9c0bf33829349e14fad1f5cff` 开始遍历然后打印 `base_info` 而已。

实际的 git 实际肯定要比上面的结构((的信息)的)要复杂的多，但是它的核心思想应该是就是，每一次提交就是一个新的结点。通过这个结点，我可以找到所有的快照文件。再思考一下，什么是分支？什么是 Tags，其实他们可能只是某次提交的引用而已(一个 `tag_head_node` 指向了某一次提交的node)。再思考怎么回退一个版本呢？指针偏移！依次类推，上面的基本命令都可以得到一个合理的解释。

(tip: *不要问我我怎么知道这么多，其实我也是猜的 ...*)
