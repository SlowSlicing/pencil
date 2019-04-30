* 打开 `Terminal`，Windows 下用 `Git Bash`。
* 进入到本地仓库目录。
* 把想要同步的仓库（就是被你 `Fork` 的仓库）关联到本地 upstream。
```
$ git remote add upstream https://github.com/*******.git（或者使用 ssh 方式）
```
* 查看远程仓库关联状态
```
$ remote -v
```
* 拉取远程仓库所有分支
```
$ git fetch upstream 
```
* 把被 Fork 仓库的代码合并到本地 master 分支
```
$ git checkout master
$ git rebase upstream/master
```
* 现在你本地已经是最新的了，只需要 `push` 到你的远程分支即可。
