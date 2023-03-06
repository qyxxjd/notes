#### 命令帮助

``` 
brew --help 
```

#### 查看已安装的软件列表

``` 
brew list 
```

#### 安装、卸载

``` 
brew install ***
brew uninstall ***
```

#### 更新

``` 
brew update      // 查看已安装软件版本更新信息
brew outdated    // 查看需要更新的软件
brew upgrade *** // 更新某个软件
```

#### 搜索

``` 
brew search *** 
```

#### 查看软件信息

``` 
brew info *** 
```

#### 清理软件的旧版本

``` 
brew cleanup     // 清理所有旧版本
brew cleanup *** // 清理某个软件的旧版本
```


#### 使用国内镜像加速访问
替换清华大学镜像
```git
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
brew update
```
还原GitHub镜像
```git
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask.git
brew update
```
打开 `.zshrc` 配置文件，加入变量
```shell
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles
```
