# 上传git仓库

## 1.删除git仓库

```
rm -rf .git
```

## 2.新建git仓库

```
git init
```

## 3.关联本地与远程

```
git remote add origin 远程仓库地址 //关联远程仓库
```

## 4.查看文件提交状态

```
git status
```

## 5.创建忽略文件

```
touch .gitignore

匹配规则
*.a # 忽略所有目录下的.a结尾的文件
!lib.a # 但lib.a除外
/TODO # 仅仅忽略项目根目录下的TODO文件，不包括subdir/TODO
build/ # 忽略build/目录下的所有文件
doc/*.txt # 忽略 doc/notes.txt 但不包括doc/server/arch.txt
doc/**/*.txt # 会忽略doc/目录及其子目录下的所有以.txt结尾的文件
```

## 6.添加

```
git add . #.表示所有
```

## 7.提交

```
git commit-m ''
```

## 8.拉取与推送

```
git pull
git push
```

