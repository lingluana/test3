 

### 初始化配置

git config --global user.name "majikai"

git config --global user.email 1793218783@qq.com

git config --list

#### 区域

工作区

暂存区

版本库

#### 对象

Git对象

树对象

提交对象



### Git目录

- hooks   目录包含客户端或服务端的钩子脚本;
- info   包含一个全局性排除文件；
- logs   保存日志信息；
- **objects   目录存储所有数据内容;**
- **refs   目录存储指向数据(分支)的提交对象的指针；**
- config   文件包含项目特有的配置选项；
- description   用来显示对仓库的描述信息；
- **HEAD   文件指示目前被检出的分支；**
- **index   文件保存暂存区信息；**



### Git 底层概念（底层命令）

#### 基础的Linux命令

- **clear**： 清除屏幕
- **echo 'test content'**：往控制台输出信息 echo 'test content' > test.txt
- **ll**：将当前目录下的 子文件&子目录平铺在控制台
- **find 目录名**：将对应目录下的子孙文件&子孙目录平铺在控制台
- **find 目录名 -type f**：将对应目录下的文件平铺在控制台
- **rm 文件名**：删除文件
- **mv 源文件 重命名文件**：重命名
- **cat 文件的url**：查看对应文件的内容
- **vim 文件的url（在英文模式下）**
  - 按 i 进入插入模式  进行文件的编辑
  - 按 esc 键 & 按 : 键   进行命令的执行
  - q!     强制退出（不保存）
  - wq   保存退出
  - set nu  设置行号

#### 初始化新仓库

命令：git init



#### Git 对象

​		Git的核心部分是一个简单的键值对数据库。你可以向该数据库插入任意类型的内容，它会返回一个键值，通过该键值可以在任意时刻再次检索该内容

##### 向数据库写入内容 并返回对应的键值

​	命令：

```
echo 'test content' | git hash-object -w -stdin
```

​	**\-w** 选项指示 hash-object 命令存储数据对象；若不制定此选项，则该命令仅返回对应的键值

**--stdin（standard input）**选项则指示该命令从标准输入读取内容;若不指定此选项,贝须在命令尾部给出待存储文f件的路径

- **git hash-object -w 文件路径** 

  存文件

- **git hash-object 文件路径**

  返回对应文件的键值

  70460b4b4aece5915caf5c68d12f560a9fe3e4

  

  返回：该命令输出一个长度为40个字符的校验和 。这是一个 SHA-1 哈希值	

  



##### 查看 Git 是如何存储数据的

​		命令：

```
find .git/objects -type f
```

​		返回：

.git/object/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4

**查看对象是什么类型**

​		命令：

```
git cat-file -t 跟哈希值
```

**查看对象的内容**

​		命令：

`  git cat-file -p hash(哈希值)`

#### 树对象

​		树对象(tree object),它能解决文件名保存的问题，也允许我们将多个文件组织到一起。Git以一种类似于UNIX文件系统的方式存储内容。所有内容均以树对象和数据对象(git对象)的形式存储，其中树对象对应了UNIX中的目录项，数据对象(git对象)则大致上对应文件内容。一个树对象包含了一条或多条记录(每条记录含有一个指向git对象或者子树对象的SHA-1指针,以及相应的模式、类型、文件名信息)。一个树对象也可以包含另一个树对象。

####	**查看暂存区当前的样子**

```
git ls-files-s
```



​	**查看树对象：**

​	**命令：**

	  git cat-file -p master^{tree} (或者是树对象的hash(哈希值))

​	master^{tree} 语法表示 master 分支上最新的提交所指向的树对象。

#### 构建树对象

​		我们可以通过 update-index；write-tree；read-tree 等命令来构建树对象并塞入到暂存区。

​		**操作**

1. 利用 update-index 命令为 test.txt 文件的首个版本--创建一个暂存区。并通过 write-tree 命令生成树对像。

   命令：

   ```
   git update-index --add --cacheinfo 100644\83baae61804e65cc73a7201a7252750c76066a30 test.txt
   
   git write-tree
   ```

   文件模式为		100644，表示这是一个普通文件

   ​						    100755，表示一个可执行文件

   ​						    120000，表示一个符号链接。

   --add 选项：

   ​	因为此前该文件并不在暂存区中 首次需要 --add

   --cacheinfo选项：

   ​	因为将要添加的文件位于Git数据库中,而不是位于当前目录下所有需要 --cacheinfo

2. 新增 new.txt 将 new.txt 太和 test.txt 、文件的第二个个版本塞入暂存区。并通过 write tree 命令生成树对像。

   命令：

   ```
   echo 'new file' > new.txt
   
   git update-index-cacheinfo 100644\ 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
   
   git write-tree
   ```

3. 将第一个树对象加入第二个树对象，使其成为新的树对象

   命令：

   ```
   git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579 
   // read-tree命令，可以把树对象读入暂存区
   
   git write-tree
   
   ```
   

 **每次生成快照的时候最好添加注释**



项目的版本就是一个**提交对象**。

本质上项目的快照是一个**树对象**。

#### 提交对象

`echo 'first commit' | git commit-tree treehash  //生成一个提交对象`



### Git 本地操作（高层命令）

#### git操作最基本的流程

创建工作目录 对工作目录进行修改

`git add ./`

上面代码相当于执行

`git hash-object -w 文件名  //修改了多少个工作目录中的文件 此命令就要被执行多少次`

`git commit -m "注释内容"`

上面代码相当于执行

`git write-tree`

`git commit-tree`

#### git高层命令（CRUD)

` git init //初始化仓库`

` git status  //查看文件的状态`

` git diff  //查看哪些修改还没有暂存`

`git diff --staged   //查看哪些修改且被暂存了 但还没提交`

` git log --online  //查看提交的历史记录`

` git add ./    //将修改添加到暂存区`

` git rm 文件名  //删除工作目录中对应的文件  再将修改添加到暂存区`

`git mv 原文件名 新文件名  //将工作目录中的文件进行重命名 再将修改添加到暂存区`

`git commit  //需要进入vim编辑器`

` git commit -a   //可以跳过git add 步骤`

`git commit -m 注释 //将暂存区提交到版本库` 

`git commit -a -m` 

#### 记录每次更新到仓库

​	工作目录下面的所有文件都不外乎这两种状态:已跟踪或未跟踪

​		已跟踪的文件是指本来就被纳入版本控制管理的文件，在上次快照中有它们的记录，工作一段时间后，它们的状态可能是已提交，已修改或者已暂存

​		初次克隆某个仓库时，工作目录中的所有文件都属于已跟踪文件，且状态为已提交;在编辑过某些文件之后，Git将这些文件标为已修改。我们逐步把这些修改过的文件放到暂存区域,直到最后一次性提交所有这些暂存起来的文件。

#### 检测当前文件状态

​	命令：

` git status`

​	作用：确定文件当前处于什么状态

​	命令：

#### 基本操作

`git diff `

作用：查看修改的有没有被暂存

​	命令：

` git diff --cached 或者 git diff --staged（1.6.1以上）`

作用：查看哪些修改已经暂存起来还没提交

##### 查看历史记录

命令：

` git log`

` git reflog  //只要动HEAD就会记录`

` git log --pretty=oneline`

` git log --oneline `



### 配别名

`git config --global alias.自己起的名字 命令名字`

例如：

`git config --global alias.st status`

这样 git status 就可以用 git st代替



### Git 分支操作（杀手功能）

**分支的本质其实就是指向提交对象的可变指针 **

​	Git 的默认分支名字是master。

**HEAD: 是一个指针 它默认指向master分支 切换分支时其实就是让HEAD指向不同的分支 每次有新的提交时 HEAD都会带着当前指向的分支 一起向前移动**

​		命令：

​	`git branch  //显示查看分支列表`

​	` git branch 分支名  //在当前提交对象上创建新的分支`

​	**`git checkout -b 分支名  //创建并切换分支`**

​	`git branch 分支名 commithash  //在指定提交对象上创建新的分支`

​	` git checkout 分支名  //切换分支`

​	` git branch -d 分支名 //删除空的分支 删除已经被合并的分支`

​	` git branch -D 分支名  //强制删除分支` 

​	`git branch -v  //可以查看每一个分支的最后一次提交`

​	` git log --oneline --decorate --graph --all //查看整个项目的分支图`

​	**` git merge 分支名（需要合并的分支 ）  //合并分支 得先返回主分支`**

​		快速合并 -->不会产生冲突

​		典型合并 -->有可能产生冲突

​		解决冲突 -->打开冲突的文件 进行修改 add commit

​	`git branch -merged  //查看已经合并到当前分支的分支列表  一旦出现在这个列表就应该删除`

​	`git branch -no -merged  //查看没有合并到当前分支的分支列表 一旦出现在这个列表中 就应该观察一下是否需要合并`

​	**注意：**

​		在切换分支的时候 ，一定要保证当前分支是最干净的！！！

​	**允许切换分支：**

​		分支上所有的内容处于 已提交状态

​		（避免）分支上的内容是初始化创建 处于未跟踪状态

​		（避免）分支上的内容是初始化创建 第一次处于已暂存状态

​	**不允许切换分支：**

​			分支上的所有内容处于 已修改状态 或 第二次以后的已暂存状态

​		分支切换会改变你工作目录中的文件

​		切换分支时，一定要注意你工作目录里的文件会被改变，如果切换到一个比较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子，如果 Git 不能干净利落的完成这个任务，它将禁止切换分支

​		**每次切换分支前 提交一下当前分支**





### Git 存储

​		**git stash 命令**：

​	` git stash` 命令会将完成的修改保存到一个栈上，而你可以任何时候重新应用这些改动（` git stash apply`）

​	`git stash list  //查看存储`

​	` git stash apply stash@{2}  //如果不指定一个储藏，Git认为指定的是最近的储藏`

​	**`git stash drop  // 加上将要移除的储藏的名字来移除它`**

​	**` git stash pop  // 来应用储藏然后立即从栈上扔掉它`**

### 撤销&重置

#### 后悔药

工作区

​	如何撤回自己在工作目录中的修改：

​		` git checkout --filename`

暂存区

​	如何撤回自己自己的暂存：

​		`git reset HEAD  filename`

版本库

​	如何撤回自己的提交:

​	1.注释写错，重新给用户一次机会该注释

​		`git commit --amend` 

#### reset

​	三部曲：

  1. 第一部：`git reset --soft HEAD~  (相当于--amand)`

     本质上是撤销了上一次 git commit 命令

​	只动HEAD(带动分支一起移动)

​		2. 第二部：`git reset --mixed HEAD~`

​	动了暂存区

​		3. 第三部：`git reset --hard HEAD~ `

​	动HEAD (带着分支一起移动)

​	动了暂存区

​	动了工作目录

#### 路径reset

​	`git reset HEAD filename`  （reset 将会跳过第 1 步）

​		动了暂存区

​	 

#### checkout 

​	`git checkout commithash ` & `git reset --hard commithash`

​		区别：

	1. checkout只动HEAD        --hard动HEAD而且带着分支一起移动
 	2. checkout对工作目录是安全的     --hard是强制覆盖工作目录



​	`git checkout commithash`

` git checkout --filename` 这个相比于`git reset --hard commitHash --filename` 第一步和第二步都没有做只动了工作目录

​	`git checkout commithash <file>`

​	将会跳过第一步 更新暂存区 工更新作目录

### 数据恢复





### 打tag

#### 列出标签

`git tag`

`git tag -l 'v1.8.5*'`

#### 创建标签

Git 使用两种主要类型的标签：轻量标签 与 附注标签

​	**轻量标签**很像一个不会改变的分支 - 它只是一个特定提交的引用

​		`git tag v1.4`

​		`git tag v1.4 commithash`

#### 查看特定标签

​	`git show tagname`

#### 删除标签

​	`git tag -d tagname`

#### 检出标签

如果你想查看某个标签所指向的文件版本，可以使用git checkout命令：

​	`git checkout tagname`

​	虽然说这会使你的仓库处于“分离头指针(detacthed HEAD)”状态。在“分离头指针状态下，如果你做了某些更改然后提交它们，标签不会发生变化，但你的新提交将不属于任何分支，并且将无法访问,除非访问确切的提交哈希。因此，如果你需要进行更改一―比如说你正在修复旧版本的错误――这通常需要创建一个新分支:

​	`git checkout -b version2(相当创建个分支)`









### eslint

​	js代码的检查工具

​	下载： npm i eslint -D

​	使用：

​		生成配置文件 	`npx eslint --init`

​		检查js文件		`npx elint 目录名`

​	命中的规则：

​			

### eslint结合git

​	husky：哈士奇，给Git仓库配置钩子程序

​	使用

​			在仓库初始化完毕之后 再去安装哈士奇

​			在package.josn文件写配置





### 远程仓库 

#### 本地分支

​	正常的数据推送 和 拉取步骤

		1.	 确保本地分支已经跟踪了远程跟踪分支
 2. 拉取数据：git pull

 3. 上传数据：git push 

    一个本地分支怎么去跟踪一个远程跟踪分支

      1. 当克隆的时候 会自动生成一个master本地分支（已经跟踪了对应的远程跟踪分支）

      2. 在建立其他分支时，可以指定想要跟踪的远程跟踪分支

         ```
         	1. git checkout -b 本地分支名 远程跟踪分支名
         	2. git checkout -track 远程跟踪分支名将一个已经存在的本地分支 改成一个跟踪分支
         ```

         

         3. 将一个已经存在的本地分支 改成一个跟踪分支

         1. git branch -u 远程跟踪分支名

            





#### 团队协作

 1. 项目经理初始化远程仓库

    一定要初始化一个空的仓库：在github上操作

	2. 项目经理创建本地仓库

    `git remote  add 别名 仓库地址（https）` 

    ` git remote -v //显示远程仓库使用的 Git 别名与其对应的 URL`

    `git init`

    将源码复制进来

    修改用户名和邮箱

    `git add`

	`git commit`

    3. 项目经理去推送本地仓库到远程仓库

    清理windows凭据

	`git push 别名 分支` （输入用户名 密码；退完之后会附带生成远程跟踪分支）

    4. 项目经理邀请成员 & 成员接受邀请

	在github上操作

    5. 成员克隆远程仓库

    `git clone 仓库地址（在本地生成 .git 文件 默认为远程仓库配别名 orgin）`

	只有在克隆时候 本地分支 和远程跟踪分支是有同步关系的

    6. 成员做出贡献

    修改源码文件

    `git add`

    `git commit`

	`git push 别名 分支 （输入用户名 密码；推完之后会附带生成远程跟踪分支）`

    7. 项目经理更新修改

    `git fetch 别名 （将修改同步到远程分支上）`
    
    `git merge 远程跟踪分支`

#### 删除远程仓库

​	` git push origin -delete serverfix  //删除远程分支`

​	`git remote prune origin --dry -run  //列出仍在远程跟踪但是远程已经被删除的无用分支`

​	` git remote prune origin  //清除上面命令列出来的远程跟踪`   

#### pull request

​	让第三方人员参与到项目中 fork	





### 使用频率高的

git status

git add

gut commit

git push

git pull



### SSH

这是github它们自己的协议

​	`ssk keygen -trsa -C` 你的邮箱：生成 公司钥

​	`.ssh` 文件位置：