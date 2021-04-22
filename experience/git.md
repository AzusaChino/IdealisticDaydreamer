# Git Related

## ssh -T git@github.com timeout

add `config` to ~/.ssh folder

```properties
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

## windows 环境下换行符导致.sh 脚本无法执行

git windows 安装之后，可以配置下面两个参数：

- `git config --global core.autocrlf true`
  - true 表示检出是转换 CRLF, 提交时转换为 LF
  - input 表示检出是不转换，提交时转换为 LF
  - false 表示不做转换
- `git config --global core.safecrlf true`
  - true 表示不允许提交时包含不同换行符
  - warn 则只在有不同换行符时警告
  - false 则允许提价时有不同换行符存在

也可以通过修改～/.gitconfig

```properties
[core]
autocrlf = false ---> true
safecrlf = true
```
