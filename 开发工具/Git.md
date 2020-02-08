**使用http协议：用户认证需可以去 控制面板中 用户下查找**



**长用**

```
git remote add origin http://192.168.4.92/CoderLG/tmp.git  //将origin 和远程地址 连接
git remote -v	//查看远程分支

权限
develop权限不能向master分支上提交，但是其他的分支上是可以的
```



**大文件上传**

```
下载安装 git large file storage

1.首先需要在git版本库所在目录下对lfs进行初始化。 执行后，在根目录下会生成“.gitattributes”文件。
	git lfs install

2.添加track规则，下面以后缀为".dat"的二进制文件为例。
	git lfs track "*.dat"

3.执行后，会发现.gitattributes文件多出一行
	*.dat filter=lfs diff=lfs merge=lfs -text
    说明已经生效。
    
4.将.gitattributes加入到版本控制中。
    git add .gitattributes
    git commit -m "add .gitattributes"

```



**年度统计**

```
1. 统计git提交次数: 所有人的所有提交次数，会展示所有的提交人 提交次数详情。
git log | grep "^Author: " | awk '{print $2}' | sort | uniq -c | sort -k1,1nr


2. 统计时间内提交次数。
git log --author="lig" --since="2019-01-01" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }'


3. 统计提交行数：根据1展示出详情，可以填入username。将展示该用户增加行数，删减行数，剩余行数。
git log --author=lig  --since="2019-01-01" --no-merges | grep -e 'commit [a-zA-Z0-9]*' | wc -l

placename
    master
    20
    2534 4531
    
    release
    29
    4887	3020

webmanager
    master
    247
    913313 759969

    release
    1
    293	0



tiltphoto
    master
    24
    2995 2707

    release
    28
    3358	2894


```



**Gerrit**

```
1.git commit一次后再次修改了文件，应如何操作
	1）命令git status    查看修改
	2）命令 git add .    提交所有修改的文件
	3）命令 git commit --amend 对前面的提交进行修订
	
2.可以通过git log|HEAD 查看当前的提交
	git log|head
	
3.已经commit多次应如何操作？
    git rebase -i
    s
    "备注"
    git status
    git review -R -v

4.review之后被驳回，怎么办？
    git add . 
    git commit --amend 
    git review -R -v 
    
    
 分支
 git checkout -b develop origin/develop  //新建分支develop从origin/develop复制  必须这样建立关系
 提交到此分支
 git add .
 git commit -m ''
 git review -R -v dev  //dev分支名
```



**李昕哲 Gerrit** 

```
git操作流程：

\1. 拉取分支到本地

git clone ssh://lixz@192.168.4.172:29418/iCenter/datamanager

2.拉取release_1.1.2到本地

git pull origin release_1.1.2:release_1.1.2

3.基于release_1.1.2建立本地分支

git checkout -b  whj

4.在本地分支上进行修改完成并进行合并。

git checkout release_1.1.2

git merge -s subtree    --no-commit   --no-ff      --allow-unrelated-histories  whj

(也可以使用idea的图形界面进行操作)

5.使用命令行

git log

找到第一个changed Id为空的地方，比如 commitid 为 3e70ec83f4b23d54670ae925db152f5c9681c82b

执行:

git reset --soft 3e70ec83f4b23d54670ae925db152f5c9681c82b

git add .

git commit -s -a -m "datamanagerUpload"

git review -R -v  release_1.1.2

代码合并完成后。将代码拉到本地release_1.1.2，并合并本地fix分支。

git pull origin release_1.1.2:release_1.1.2

git checkout whj

git merge release_1.1.2


```



**配置文件**

```
git config --local	user.name ""
git config --local	user.email ""

//非常不推荐
git config --global user.name "lig"
git config --global user.email "lig@geovis.com.cn"

--------------------------------

git config  user.name lig 
git config  user.email lig@geovis.com.cn
git config --global gitreview.user lig
git config --global gitreview.email lig@geovis.com.cn
git config --global gitreview.remote origin

```



**分支**

```
push
---------------------------
直接push到一个远程没有的分支，必须远程上是一个非空的项目才行

push到一个远程已经有的分支
git fetch origin develop	//拉去远程分支
git checkout -b develop origin/develop  //新建分支develop从origin/develop复制  必须这样建立关系

----------------------


git branch -a					//查看所有分支
git fetch origin develop		 //拉去远程分支
git checkout -b develop master    //新建分支develop 从本地复制
git checkout -b develop 		 //新建分支develop 从所在分支上复制
git checkout master //切换到主分支

//新建远程分支
直接push到一个远程没有的分支，必须远程上是一个非空的项目才行
git push origin dev:dev  //本地：远程
//删除远程分支
 git push origin  :lg  		//(空：远程分支)
 
//删除本地已经提交的文件
 git rm -rf --cache .idea 
```

**log**

```
git log 		空格翻页 b上一页 q退出
git reflog

git reset --hard 对应的id

git diff 
```









### git教程

<https://www.liaoxuefeng.com/> 
<https://blog.csdn.net/kong_21/article/details/72571854> 

### 一台电脑同时用github和gitlab

<http://www.arccode.net/config-multi-git-account-and-workspaces.html> 

### 中文乱码

<https://blog.csdn.net/MLQ8087/article/details/52174834> 
<https://gist.github.com/nightire/5069597> 

### 如何忽略这些文件和文件夹的变更信息？

在git管理的项目文件夹中（严格的讲，应该叫做git的本地repository），创建一个文件名为“.gitignore”的纯文本文件 ,设置需要过滤的文件
内容详解：https://gist.github.com/octocat/9257657 

### 明明在gitignore中忽略了.idea文件夹,但是提交时仍旧会出现.idea内文件变动的情况 

.idea已经被git跟踪，之后再加入.gitignore后是没有作用的 ， 用 git rm -r --cached .idea 清除.idea的git缓存，并在.gitignore中添加.idea 

### Commit但还没有push，返回commit之前

git  reset --soft commit_id(可以git log查看) 15678b6cb19cba21366144864311f27458553637

### 主从分支

<https://blog.csdn.net/carfge/article/details/79691360> 

### 从远程仓库拉取并合并

<https://blog.csdn.net/weixin_44380128/article/details/87695222> 





