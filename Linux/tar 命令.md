Linux
tar,详解
[TOC]

> &emsp;&emsp;在 Linux 中用 `tar` 来存储或者展开 `.tar`、`.tar.gz` 的存档文件，必须配合参数使用。

# 命令

## 独立参数

| 参数 | 作用 |
| --- | --- |
| -c | 建立压缩档案 |
| -x | 解压 |
| -t | 查看内容 |
| -r | 向压缩归档文件末尾追加文件 |
| -u | 更新原压缩包中的文件 |

&emsp;&emsp;这五个是独立的命令参数，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。

## 常规参数

&emsp;&emsp;下面的参数是根据需要在压缩或解压档案时可选的：

| 参数 | 作用 |
| --- | --- |
| -z | 有gzip属性的 【.tar.gz】 |
| -j | 有bz2属性的 【bz2】 |
| -J | 有xz属性的 【xz】 |
| -Z | 有compress属性的 |
| -v | 显示所有过程 |
| -O | 将文件解开到标准输出 |
| -C | 指定解压目录 |
| -f | 使用档案名字，`切记，这个参数是最后一个参数，后面只能接档案名。` |


## 特殊参数

| 参数 | 作用 |
| --- | --- |
| `--exclude` | 排除目录，文件 |

## 例

```
# 这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。
tar -cf all.tar *.jpg

# 这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。
tar -rf all.tar *.gif

# 这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。
tar -uf all.tar logo.gif

# 这条命令是列出all.tar包中所有文件，-t是列出文件的意思
tar -tf all.tar

# 这条命令是解出all.tar包中所有文件，-t是解开的意思
tar -xf all.tar

# 排除html下的 web 文件夹，不要在后面带 “/”，排除文件 Dockerfile，使用相对路径
tar -czvf html.tar.gz html --exclude=html/web --exclude=html/Dockerfile
```

* 压缩

```
# 将目录里所有jpg文件打包成tar.jpg
tar -cvf jpg.tar *.jpg

# 将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
tar -czf jpg.tar.gz *.jpg

# 将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
tar -cjf jpg.tar.bz2 *.jpg

# 将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
tar -cZf jpg.tar.Z *.jpg

# rar格式的压缩，需要先下载rar for linux
rar a jpg.rar *.jpg

# zip格式的压缩，需要先下载zip for linux
zip jpg.zip *.jpg
```

* 解压

```
# 解压 tar包
tar -xvf file.tar

# 解压tar.gz
tar -xzvf file.tar.gz

# 解压 tar.bz2
tar -xjvf file.tar.bz2

# 解压tar.Z
tar -xZvf file.tar.Z

# 解压tar.xz
tar -xJvf fle.tar.xz 

# 解压rar
unrar file.rar

# 解压zip
unzip file.zip
```
