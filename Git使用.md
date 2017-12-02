# Git使用

### 创建仓库

1. 首先在github上创建一个仓库 new repository
2. 进入name目录下，git init 初始化此目录，会出现 .git 文件夹
3. git config user.email ,git config user.name 配置此库的用户email和name
4. vim README.md创建一个初始的文件
5. git add README.md（git add .）添加到git. git commit -m""提交到git
6. git remote rm origin 添加远程主机前做一下删除
7. git remote add origin git@github.com:dongfangqiaoen/variety_shop.git（第一步创建的仓库的地址）
8. git push -u origin master 将当前分支推送到origin主机，并默认origin主机（-u）
9. 如果做完测试想删掉此库，可在此库中的setting中的最下面选择delete this repository
