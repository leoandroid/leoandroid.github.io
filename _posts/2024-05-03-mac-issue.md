---
show_subscribe: false	

title: Mac 疑难杂症
tags: 
    - Mac
---

## 1. Ruby 安装使用过程中遇到的问题:


1. Ruby/Gem 下载慢的问题:
```shell
# 添加镜像源并移除默认源
gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
# 列出已有源，应该只有镜像源一个
gem sources -l
# 使用以下命令替换 bundler 默认源
bundle config mirror.https://rubygems.org https://mirrors.tuna.tsinghua.edu.cn/rubygems
```

2. Mac 自带的 `Ruby` 和通过 `HomeBrew` 下载的`Runby` 冲突问题。
通过 `HomeBrew` 下载完最新的 `ruby` 后，在使用 `which ruby` 或 `where ruby` 或 `ruby -v` 返回的都是Mac自带的 `ruby` 而不是 `HomeBrew` 下载的。在 `~/.zshrc` 中添加下面的代码:
```shell
if [ -d "/opt/homebrew/opt/ruby/bin" ]; then
 export PATH=/opt/homebrew/opt/ruby/bin:$PATH
 export PATH=`gem environment gemdir`/bin:$PATH
fi 
```
    







