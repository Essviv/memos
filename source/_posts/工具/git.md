---
title: git
author: essviv
date: 2016-04-27 10:20:54+0800
---

#写在前面

这篇文档不是git命令的学习文档，git的基础命令及相应的语法和使用可以通过git help subcommand很方便地查到. 这里主要是将自己在学习git过程中遇到的一些问题记录下来，以备后续备忘使用. 

# 名词解释
* HEAD: git内部使用HEAD来指示当前所在分支的最新提交 

* master: 从远程仓库拉取代码时，git会默认在本地创建一个master分支，并用它来跟踪远程仓库中origin/master分支的内容

* origin: 从远程创建拉取代码时，git会默认将远程仓库的地址设置为origin，也可以通过git remote add手工加入其它的远程仓库地址

# Rebase

在git内部中使用分支是个很常见的做法，使用分支必然意味着需要将不同的分支内容进行合并. 在git中提供了两种合并的方法，一种是merge， 一种是rebase. merge的使用方法比较简单，这里不做阐述，这是主要是简单记录下rebase的使用方法. 查阅rebase的文档可以发现，它的使用方法有下面三种: 

	> rebase --onto newBase Upstream Branch

## rebase --onto A B C

这是最完整的命令形式，它的作用是切换到branch分支，并找到这个分支中**不存在**于upstream中的那些提交，将这些提交一一地在newBase中重放. 举个例子，可以看到，在进行rebase操作之前，分支b2是提交是在b1提交的基础上，在执行了rebase操作后，b2分支中的5b9686以及16f966两次被rebase到master分支的f71fe18的提交上, 并且HEAD指针指向b1分支. 

```shell
    > git lg
    * 5b9686c1fb2dbb1ef95fb36b47254bbf66225b83 (HEAD -> b2) 7
    * 16f9663f77c6c58447e2f7d7f2b4f129fc1f1b05 6
    * 9947edd2cbafc34cc24d0182a8118e900ee043d9 (b1) 5
    * 479e44cf3d5e0e20f82ecd84f597d73c47b69919 3
    | * f71fe18e70ea2e2b2ae04ec405ea2e57b03be496 (master) 4
    |/  
    * 88a3b40bea4b9a4c0971ba4e08efb92491c2c4a3 2
    * 95f16a83315b86a4b0228310626ddcab69aefa96 1

    > git rebase --onto master b1 b2
    > git lg
    * c87c8e21df58e064b5b6e23def4efc9ead57bec6 (HEAD -> b2) 7
    * aa84807e80971f8d51930a5c6d9dbf055bceff2a 6
    * f71fe18e70ea2e2b2ae04ec405ea2e57b03be496 (master) 4
    | * 9947edd2cbafc34cc24d0182a8118e900ee043d9 (b1) 5
    | * 479e44cf3d5e0e20f82ecd84f597d73c47b69919 3
    |/  
    * 88a3b40bea4b9a4c0971ba4e08efb92491c2c4a3 2
    * 95f16a83315b86a4b0228310626ddcab69aefa96 1
```
    
## rebase B C

这个命令省略了--onto参数，它默认将Upstream分支当作rebase分支, 从下面的例子可以看出，这个命令首先将分支切换到b1，并找到不在b2中的所有提交, 这里即20c108和f6a00b两次提交，然后把这两个提交在b2分支上进行重放. 最后得到一条线性的提交树

```shell
    > git rebase b2 b1
    * 20c108c57a31783bdd9b794d9ad9549bbdc9a5e3 (HEAD -> b1) 5
    * f6a00b0ee7905b937843747ee95ab73b891aef32 3
    * c87c8e21df58e064b5b6e23def4efc9ead57bec6 (b2) 7
    * aa84807e80971f8d51930a5c6d9dbf055bceff2a 6
    * f71fe18e70ea2e2b2ae04ec405ea2e57b03be496 (master) 4
    * 88a3b40bea4b9a4c0971ba4e08efb92491c2c4a3 2
    * 95f16a83315b86a4b0228310626ddcab69aefa96 1
```

## rebase B

这个命令格式省略了branch参数, 也就是说它在进行rebase操作之前，并不会进行分支切换，而是直接在当前分支进行操作. 最后的结果就是，计算出当前分支中没有在upstream分支中的提交对象，并将它们一一在upstream分支中回放.

## 使用rebase要注意的事项
使用rebase可以使提交历史更加清晰明了，但使用它有个前提，就是不能对已经推到远程仓库的提交信息进行rebase, 否则其它的合作者在重新拉取仓库时会遇到很难搞的一些问题.
    
# Git的内部存储机制

git除了对外提供简洁友好的用户命令之外，对内部它还有一套底层的接口来实现对象的存储. git内部存储的所有信息都放在.git目录下，目录的结构如下所示,
![git目录](https://github.com/Essviv/images/blob/master/git-dir.png?raw=true)

其中branches在高版本的git中已弃用，config中保存了项目相关的一些配置, description记录了gitWeb需要使用的一些信息，hooks记录了客户端以及服务端的一些hook配置，info中记录了全局配置的忽略跟踪的一些文件(.gitignore),除此之外，就剩下四个比较重要的文件和目录，分别是HEAD, index, objects和refs.

其中HEAD记录了当前分支的最新提交信息，index中记录了暂存区的信息，objects目录中记录了库中所有的对象，具体的存储形式在后续中会提到，而refs中又有heads, tags，以及remotes三个子目录，分别对应记录了分支，标签以及远程仓库的提交信息. 

## object

在git中,所有的对象都被存储为blob的形式，具体的存储过程如下：
1. 获取相应的内容(content)
2. 增加相应的头信息(header)
3. 获取内容和头信息的sha1(sha1)
4. 使用zlib压缩内容(zlib_content)
5. 存储

```ruby
#引用库
require 'digest/sha1'
require 'zlib'
require 'fileutils'

#blob内容
content = "hello world!"

#blob头信息
header = "blob #{content.length}\0"

#blob存储的消息
store = header + content

#blob消息的sha1
sha1 = Digest::SHA1.hexdigest(store)

#blob消息的zlib
zlib_content = Zlib::Deflate.deflate(store)

#写到文件中
path = '.git/objects/' + sha1[0, 2] + '/' + sha1[2, 38]

FileUtils.mkdir_p(File.dirname(path))

File.open(path, 'w'){|f|f.write zlib_content}

#校验
git cat-file -p bc7774a7b18deb1d7bd0212d34246a9b1260ae17
```
    
## hash-object

上面的ruby脚本简单地展示了git内部创建blob对象的过程，事实上，git也提供了hash-object命令用于很方便地创建git节点，在git内部也是使用这个命令创建各个对象的blob

```shell
    > echo "hello world!" >> test.txt
    > git hash-object -w test.txt
    a0423896973644771497bdc03eb99d5281615b51
    > git cat-file -p a0423896973644771497bdc03eb99d5281615b51
    hello world!
```
    
## cat-file

这个命令号称是git用来检测内部对象的“瑞士军刀”. 它可以用来查询对象的存储信息，具体的输出根据不同的类型会有不同的信息，当然它也可以用来查询存储对象的类型信息, 在git中存储的对象类型分为四种，分别是blob, tree, commit和tag.

```shell
    > git cat-file -p a0423896973644771497bdc03eb99d5281615b51
    hello world!
    
    > git cat-file -t a0423896973644771497bdc03eb99d5281615b51
    blob
```

## update-index

这个命令是将相应的文件增加到暂存区，它还可以更新暂存里的文件内容，在执行完这条命令后，文件的blob对象就已经存储到git的objects中了.

```shell
    > git update-index --add test.txt
```

## write-tree

在将文件加到git的暂存区后，就可以使用write-tree命令了。write-tree命令是将暂存区中存储的对象写成一个树对象，并返回相应的SHA1标识. 同样地，生成的tree对象也存储在.git的objects中

通过cat-file查询树对象可以发现，树对象中存储了相应对象的类型，地址信息，及其文件名和读写权限信息(100644)

```shell
    > git write-tree
    5d56cf9b9843c20d7b29bf6374501f4f48841210
    
    > git cat-file -p 5d56cf9b9843c20d7b29bf6374501f4f48841210
    100644 blob a0423896973644771497bdc03eb99d5281615b51	test.txt
```

## commit-tree

在使用write-tree生成相应的树对象之后，就可以使用commit-tree命令来生成相应的commit对象了.顾名思义，这里的commit对象就是使用git commit时会生成的对象. 在使用commit-tree时，必要指定要提交的树对象的标识，即由write-tree命令返回的sha1标识，另外，它还可以通过-p参数指定父提交对象，这样就形成了使用git log时显示的提交历史树.

通过cat-file来查询commit-tree对象，可以看到对象中存储了提交信息，作者信息，提交者信息以及相应的树对象地址.

```shell
    > echo 'first commit' | git commit-tree 5d56cf9b9843c20d7b29bf6374501f4f48841210
    4f69a44b7b55cd5dd4dcb872af5b0ac5e793de9b
    
    > git cat-file -p 4f69a44b7b55cd5dd4dcb872af5b0ac5e793de9b
    tree 5d56cf9b9843c20d7b29bf6374501f4f48841210
    author essviv <514912821@qq.com> 1461726403 +0800
    committer essviv <514912821@qq.com> 1461726403 +0800
    
    first commit
    
```

## update-ref

在学习git的过程，经常会提到分支是git有别于其它VCS的“杀手锏”，因为在git中创建分支是个非常简单快速的过程, 而update-ref就是用来完成这个功能的. 在.git/refs目录下可以看到两个子目录，分别是heads和tags, 其中heads目录记录了各个分支的头结点的信息, 而tags则记录了tag信息，以下的例子就通过update-ref很简单地创建了个test分支，可以看到，在git中创建个分支简单到只需要在refs/heads/中创建个相应的文件，并在文件中写入41个字符（40个sha1标识+1个换行符). 

```shell
    > echo '4f69a44b7b55cd5dd4dcb872af5b0ac5e793de9b' >> test
```

## symbolic-ref

除了分支信息外，git中还通过HEAD指针来指示当前所在分支的最新提交对象，这个信息就存储在.git/HEAD中，可以从HEAD文件的内容中看到，这个文件里存储的就是到refs/heads中某个文件的引用而已. 可以想像，在git中使用git checkout进行分支切换的时候，只需要将HEAD的中的内容指定相应的refs/heads中的分支结点，然后更新工作区的内容即可. 

```shell
    > cat HEAD
    ref: refs/heads/test
```

## All-in-one

这里通过上述的命令简单地构造git的提交信息，完成通过git用户命令就可以完成的内容，进一步加深对git内部存储的理解.

> 创建文件test，并产生一些文件内容
> 使用update-index --add将它加入到暂存区
> 使用write-tree获取相应的树对象
> 使用commit-tree获取相应的提交对象C1
> 修改文件test内容，并增加新的文件new，加上一些内容
> 使用update-index将它们加入到暂存区
> 使用write-tree获取相应的树对象
> 使用commit-tree获取相应的提交对象C2，并将它的父提交对象设置为C1
> 重复前面四个步骤，获取提交对象C3，并将它的父对象设置为C2
> 使用update-ref在refs/heads中创建master分支，并将它指向C1
> 使用symbolic-ref将HEAD对象指定master
> 使用git log来查看刚刚创建的提交记录树

```shell
> echo “first commit” >> test

> git update-index --add test

> git write-tree
bff0827a0da713487c9fca9235263772db9a3b0e

> echo "first commit" | git commit-tree bff0827a0da713487c9fca9235263772db9a3b0e
d6fac28b22fffdcd4d54126ec44ec86a712d6fde

> echo "second commit" >> test

> echo "new file" >> new

> git update-index test

> git update-index --add new

> git write-tree
bde4f04d1037685b47deeb515984cdb994888d07

> echo "second commit" | git commit-tree bde4f04d1037685b47deeb515984cdb994888d07 -p d6fac28b22fffdcd4d54126ec44ec86a712d6fde
0f33097c08b7827224083238dc9294f59aa7948a

> echo "third commit" >> test

> echo "third commit" >> new

> git update-index test

> git update-index new

> git write-tree
f4c0f72be323c47af0dd2bbddfbaf6f548b11926

> echo "third commit" | git commit-tree f4c0f72be323c47af0dd2bbddfbaf6f548b11926 -p 0f33097c08b7827224083238dc9294f59aa7948a
25eee6f70d508567abe72969d3524f557bc159d8

> git update-ref refs/heads/master 25eee6f70d508567abe72969d3524f557bc159d8

> git update-ref refs/heads/test 0f33097c08b7827224083238dc9294f59aa7948a

> git log --pretty=onelie --graph --decorate=short

> git symbolic-ref HEAD refs/heads/test

> git log --pretty=onelie --graph --decorate=short
```

## 其它

从上述的描述中可以看到，git内部维护了许多对象，包括blob, blob, tree以及tags信息，并且每次更新后重新提交又会创建新的blob对象，这样，随着时间的推移，git内部的对象将越来越多，如果不对这些对象进行一些处理，可以想像，这些对象占据的空间将逐步增大，因此，git内部使用了压缩的方法, 将一批文件压缩将形成pack文件，pack文件还有相应的idx信息，方便外部快速地对pack文件进行读取,这部分内容具体可以git官方文档.

# 参考文档

ProGit Book: [https://progit2.s3.amazonaws.com/en/2016-03-22-f3531/progit-en.1084.pdf](https://progit2.s3.amazonaws.com/en/2016-03-22-f3531/progit-en.1084.pdf)