Go语言中提供映射关系的容器为 `map`，内部使用`散列表（hash）`实现。是一种无序的基于`key-value`的数据结构，`map` 是引用类型，必须初始化才能使用。

# map 定义

`map` 定义语法如下：

```go
map[KeyType]ValueType
```

- `KeyType`：表示键的类型
- `ValueType`：表示键对应的值的类型

`map` 类型的变量默认初始值为 `nil`，需要使用 `make()`函数来分配内存。语法为：

```go
make(map[KeyType]ValueType, [cap])
```

其中 `cap` 表示 `map` 的容量，该参数虽然不是必须的，但是应该在初始化时就为其指定一个合适的容量。

# map 基本使用

内置的 `make` 函数可以创建一个 `map`

```go
func createMap() {
	users := make(map[string]int, 8)
	users["小王"] = 18
	users["小李"] = 2
	fmt.Println(users)
	fmt.Println(users["小王"])
	fmt.Printf("type of a:%T\n", users)
}
```

也可以使用 `map` 字面值语法创建，支持在声明的时候填充元素，例如：

```go
func createMap2() {
	users := map[string]int{
		"小周": 22,
		"小张": 23,
	}
	fmt.Println(users)
	fmt.Println(users["小王"])
	fmt.Printf("type of a:%T\n", users)
}
```

创建空的 `map` 的表达式是

```go
map[string]int{}
```

# 判断某个键是否存在

格式如下:

```go
value,ok := map[key]
```

示例代码：

```go
func existMap() {
	users := map[string]int{
		"小周": 22,
		"小张": 23,
	}
	// 如果返回值是bool值通常使用ok接收，约定俗称
	v, ok := users["小张"]
	if ok {
		fmt.Println(v)
	} else {
		fmt.Println("不存在")
	}
}
```

# map 遍历

Go语言中使用 `for range` 遍历 `map`。

```go
	users := map[string]int{
		"小周": 22,
		"小张": 23,
	}
	for k, v := range users {
		fmt.Println(k, v)
	}		
```

只想遍历 `key` 时可以这么写：

```go
	for k := range users {
		fmt.Println(k)
	}
```

**注意：** 遍历 `map` 时的元素顺序与添加键值对的顺序无关。

# 删除键值对

使用 `delete()` 内建函数从 `map` 中删除一组键值对，函数格式如下：

```go
delete(map, key)
```

- `map` ：表示要删除键值对的`map`
- `key` ：表示要删除的键值对的键

示例代码如下：

```go
func delMap() {
	users := map[string]int{
		"小周": 21,
		"小张": 22,
	}
	for k, v := range users {
		fmt.Println(k, v)
	}
	delete(users, "小周")
	for k, v := range users {
		fmt.Println(k, v)
	}
}
```

注意：这些操作是安全的，即使元素不在map中也没有关系；如果找失败将返回 `value` 类型对应的零值。

# 按照指定顺序遍历

`map` 的迭代顺序是不确定的，并且不同的哈希函数实现可能导致不同的遍历顺序。实践中遍历的顺序是随机的，每一次遍历的顺序都不相同。这是故意的，每次都使用随机的遍历顺序可以强制要求程序不会依赖具体的哈希函数实现。如果需要按顺序遍历 `key/value` 对，必须显式地对 `key` 进行排序

```go
func orderTraversalMap() {
	users := map[string]int{
		"1": 22,
		"2": 23,
		"3": 23,
		"4": 23,
		"5": 23,
		"6": 23,
		"7": 23,
		"8": 23,
	}
    
	// 这里看到map的迭代顺序是不确定随机的，并且不同的哈希函数实现可能导致不同的遍历顺序
	for k, v := range users {
		fmt.Println(k, v)
	}
	fmt.Println("************排序后：***********************")
	// 如果需要按顺序遍历 key/value，必须显式地对key进行排序
	// 1.取出map中的所有key存入切片keys
	var keys = make([]string, 0, 200)
	for key := range users {
		keys = append(keys, key)
	}
	// 2.对切片进行排序
	sort.Strings(keys)
	// 3.按照排序后的key遍历map
	for _, key := range keys {
		fmt.Println(key, users[key])
	}
}
```

# 元素为 map 类型的切片

下面的代码演示了切片中的元素为map类型时的操作：

```go
func sliceValueMap() {
	var mapSlice = make([]map[string]string, 3)
	for index, value := range mapSlice {
		fmt.Printf("index:%d value:%v\n", index, value)
	}
	fmt.Println("after init")
	// 对切片中的map元素进行初始化
	mapSlice[0] = make(map[string]string, 10)
	mapSlice[0]["name"] = "wangpengliang"
	mapSlice[0]["password"] = "123456"
	mapSlice[0]["address"] = "北京"
	for index, value := range mapSlice {
		fmt.Printf("index:%d value:%v\n", index, value)
	}
}
```

# 值为切片类型的 map

下面的代码演示了map中值为切片类型的操作：

```go
func mapValueslice() {
	var sliceMap = make(map[string][]string, 3)
	fmt.Println(sliceMap)
	fmt.Println("after init")
	key := "中国"
	value, ok := sliceMap[key]
	if !ok {
		value = make([]string, 0, 2)
	}
	value = append(value, "北京", "上海")
	sliceMap[key] = value
	fmt.Println(sliceMap)
}
```