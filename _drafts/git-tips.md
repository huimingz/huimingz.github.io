
### 按行add/reset代码

```
# 部分提交
git add -p file

# 部分重置
git reset -p file
```

### rebase 与 merge

```
# rebaseの場合のコマンド例

git checkout 分岐先のブランチ1
git rebase master
git checkout 分岐先のブランチ2
git rebase master
```

```
# merge の場合のコマンド例

git checkout master
git merge 分岐先のブランチ1
git merge 分岐先のブランチ2
```

注意事项:

1. 更新当前分支代码的时候一定要使用 `git pull origin xxx --rebase`
2. 合并代码的时候按照最新分支优先合并为原则
3. 要经常从上游分支更新代码，如果长时间不更新上游分支代码容易出现大量冲突

### Cherry Pick

cherry-pick就是从不同的分支中捡出一个单独的commit，并把它和你当前的分支合并。如果你以并行方式在处理两个或以上分支，你可能会发现一个在全部分支中都有的bug。如果你在一个分支中解决了它，>你可以使用cherry-pick命令把它commit到其它分支上去，而不会弄乱其他的文件或commit。



### 查找历史代码

搜索提交历史中的内存：

```
git grep <regexp> $(git rev-list --all)
```

## 参考

* [How to grep (search) committed code in the Git history](https://stackoverflow.com/questions/2928584/how-to-grep-search-committed-code-in-the-git-history)
* [3分で理解できる！git-rebaseとmergeとの違いまとめ](https://blog.codecamp.jp/git_rebase)

