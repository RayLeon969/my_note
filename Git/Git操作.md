在此记录笔记上传的操作

git安装自行下载安装：傻瓜式操作

# SSH密匙

安装好git后，首先去计算机登录用户的文件夹下，即：

![image-20231204104157973](assets/image-20231204104157973.png)

然后找到.ssh文件夹

![image-20231204104226470](assets/image-20231204104226470.png)

进去后打开Git Bash

```bash
设置用户名
git config --global user.name "Ellen"

设置邮箱
git config --global user.email "1140267690@qq.com"

生成rsa密匙和公匙
ssh-keygen -t rsa -C “1140267690@qq.com”
```

一直按回车即可

执行完毕后显示：

![image-20231204104645279](assets/image-20231204104645279.png)

然后去github添加SSH key

![image-20231204104758301](assets/image-20231204104758301.png)

![image-20231204105003607](assets/image-20231204105003607.png)

然后可以在Bash中输入以下内容检查是否成功

```bash
 ssh -T git@github.com
```

成功显示：

![image-20231204105117280](assets/image-20231204105117280.png)



# Pull操作

如果是新建文件夹	





1. 首先去需要pull的GitHub项目下，复制如下内容。

![image-20240212113611818](assets/image-20240212113611818.png)

2. 然后使用 git clone

```bash
git clone git@github.com:RayLeon969/my_note.git
```

3. 然后 cd my_note/
4. git checkout master 切换分支
5. 然后直接git pull



# Push操作

操作顺序是先add，然后commit，最后push.

首先来到项目文件夹下

## add

```bash
git add .
```



## commit

```bash
git commit -m "第几次提交or别的"
```



## push

```bash

git push -u origin master

覆盖式提交
git push -f origin master

标有origin的代表是远程分支 git push origin<远程分支>:<本地分支> 代表从本地分支推送到远程分支 如果没有远程分支就会创建这个远程分支
git push origin master:master
```





如果提示没有origin则可能是未建立remote的原因

## remote

```bash
查看有无remote
git remote -v

添加remote
git remote add origin git@github.com:RayLeon969/my_note.git
```

添加remote的地址来源于：

![image-20231204105558962](assets/image-20231204105558962.png)



# Reset操作

```bash
git reset --hard xxxxx
```

--hard会强制回退到xxxx的commit节点，不会保留你所做的更改，所以你应该将所做的更改保存u或者考虑用--sorf

xxxx可以用git log 查询

# Branch

```bash
查看分支
git branch

删除分支
git branch -d or -D xxxx
-D是强制删除，不会保留更改

查看远程分支
git branch -r

删除远程分支
git push origin --delete xxxx
```



# checkout操作

```bash
切换分支
git checkout master

基于当前分支创建新分支并切换到新分支
git checkout -b new_branch
```





# 记录端口不可用Bug

上传报错：

```bash
ssh: connect to host github.com port 22: Connection timed out
```

显示22端口连接超时 查阅相关资料后解决方法如下：

1. 进入当前用户的下的.ssh文件中

2. 使用vim config来编写config文件

3. 文件内容如下

   ```tex
   Host github.com
   User git
   Hostname ssh.github.com
   PreferredAuthentications publickey
   IdentityFile ~/.ssh/id_rsa
   Port 443
   ```

4. 保存后使用以下命令检查连通性

   ```bash
   ssh -T git@github.com
   ```

   

