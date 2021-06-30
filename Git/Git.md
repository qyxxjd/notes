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
git remote add target xxx.git
git fetch target 
git checkout dev
git merge target/dev
```

全局配置
```git
// 用户信息
git config --global user.name Classic
git config --global user.email pgliubin@gmail.com
// 配置大小写敏感
git config --global core.ignorecase false
// 提交文件最大值
git config --global http.postBuffer 524288000
// HTTP代理
git config --global http.proxy "http://127.0.0.1:8080"
git config --global https.proxy "http://127.0.0.1:8080"
// socks5代理
git config --global http.proxy "socks5://127.0.0.1:1080"
git config --global https.proxy "socks5://127.0.0.1:1080"
// 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```
