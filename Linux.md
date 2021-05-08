# Linux

查询所有自启动的服务

`systemctl list-unit-files | grep enable`



vi

```shell
# vi
# 命令模式下
ctrl + f 向上移动一页 
ctrl + b 向下移动一页
G 移动到文件最后一个行
gg 移动到文件第一行

x delete
dd 删除当前行
yy 复制当前行
p 粘贴
u 撤销

```



`git init`

```shell
echo "# learn" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/DaxiaoYang/learn.git
git push -u origin main
```

