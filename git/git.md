# git相关
## git仓库地址变更，本地如何修改
```
git remote rm apptrace-docs
git remote add apptrace-docs git@172.16.30.10:william/apptrace-docs.git
```
## 代码提交流程
```
git pull apptrace-docs master
git status
git add .
git status
git commit -m "提交记录"
git status
git push apptrace-docs willian.he
```
## git强制覆盖
```
git checkout willian.he
git fetch --all
git reset --hard origin/master
git push -f
```

## 合并某个分支的一个commit到另一个分支
例如要将A分支的一个commit合并到B分支：
首先切换到A分支
```
git checkout A
git log
```
找出要合并的commit ID :
例如
`0128660c08e325d410cb845616af355c0c19c6fe`
然后切换到B分支上
```
git checkout B
git cherry-pick  0128660c08e325d410cb845616af355c0c19c6fe
```
然后就将A分支的某个commit合并到了B分支了

## 存储密码
执行pull、 push等操作；可能需要输入密码，如果总是提示需要输入密码，可以使用命令：
git config --global credential.helper store
来长期储存密码，如果电脑只使用一个git账号，可以直接使用

