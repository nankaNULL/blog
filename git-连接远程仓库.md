Git的一些命令，小记一下

参考博客：

<https://blog.csdn.net/u012145252/article/details/80628451>

<https://www.cnblogs.com/zyjzz/p/7653378.html>

新建本地仓库关联到远程仓库完成过程（出bug的原因不想写了，参考上面博客吧）

1. 初始化本地仓库

   ```
   git init
   ```

2. 本地仓库代码提交存储

   ```
   git add .
   git commit -m " ... "
   ```

3. 本地仓库连接到远程仓库

   ```
   // origin 是远程仓库名 可以自定义
   git remote add origin git@github.com:nankaNULL/nodejs-egg.git
   
   // 查看远程仓库信息
   git remote -v 
   ```

4. 本地分支连接到远程分支

   ```
   git branch --set-upstream-to=origin/远程分支的名字  本地分支的名字
   // 或者 新建
   // new为新建分支名
   git branch –set–upstream new origin/new 
   
   // 查看本地分支和远程分支的关联关系
   git branch -vv 
   ```

5. 合并两个独立启动仓库的历史

   ```
   git pull origin master –-allow-unrelated-histories
   ```

6. 然后就可以了

merge合并

1. 在当前开发的分支上执行，在merge后跟上需要合并的分支名

   ```
   git merge dev_1.4.0
   ```

2. 然后看下信息，没什么问题的话就push上去