* 定义一个空 Map

```
declare -A map=（）
```

* 定义时初始化 Map

```
declare -A map=(["a"]="1" ["b"]="2")
```

* 输出所有 key

```
echo ${!map[@]}
```

* 输出所有 value

```
echo ${map[@]}
```

* 添加值

```
map["c"]="3"
```

* 输出 key 对应的值

```
echo ${map["a"]}
```

* 遍历 Map

```
for key in ${!map[@]}
do
    echo ${map[$key]}
done
```

* 找到就删除，没找到就新增

```
if [ ! -n "${map[$key]}" ]
then
	map[$key]=$value
else
	echo "find value"
	unset map[$key]
fi
```
