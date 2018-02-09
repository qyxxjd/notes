## 标签

查看`tag`列表
```git
git tag
```

创建
```git
git tag xxx 
git tag -a v2.0.0 -m '备注信息' 
```

删除
```git
git tag -d xxx
```

Fork项目和源项目代码同步
```git
// 添加一个别名为 target 的地址, 指向目标项目
git remote add target git@gitlab.ops.xkeshi.so:android_cashier/mvn-repo.git
git fetch target 
git checkout dev
git merge target/dev
```
