* 定义数组

```
my_array=(A B "C" D)
```

或者

```
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
```

* 读取数组

```
${array_name[index]}
```

* 获取数组中的所有元素

```
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

# 使用@ 或 * 可以获取数组中的所有元素
echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"
```

* 获取数组的长度

```
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

# 获取数组长度的方法与获取字符串长度的方法相同
echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"
```

* 数组遍历

```
for ((i = 0; i < ${#array[@]}; i++)); do
    #${#array[@]}获取数组长度用于循环
    echo ${array[i]}
done
```

或者

```
for element in ${array[@]}; do 
    #也可以写成for element in ${array[*]}
    echo $element
done
```

* 把字符串转换成数组

```
str="ONE,TWO,THREE,FOUR"
arr=($(echo $str | tr ',' ' '))
```