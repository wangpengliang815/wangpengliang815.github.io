
Go语言内置的`flag`包实现了命令行参数的解析，使得开发命令行工具更为简单。

# os.Args

如果只是简单的想要获取命令行参数，可以像下面的代码示例一样使用 `os.Args` 来获取命令行参数。

```go
package main

import (
	"fmt"
	"os"
)

// 最简单的获取命令行参数
func getOsArgs() {
	// os.Args是一个[]string
	if len(os.Args) > 0 {
		for index, arg := range os.Args {
			fmt.Printf("args[%d]=%v\n", index, arg)
		}
	}
}
```

```bash
PS D:\GoProject\src> go run .\flagTest.go "a" "s" "1"
args[0]=C:\Users\ADMINI~1\AppData\Local\Temp\go-build752076497\b001\exe\flagTest.exe
args[1]=a
args[2]=s
args[3]=1
```

`os.Args` 是一个存储命令行参数的字符串切片，第一个元素是执行文件的名称。

# flag包使用

介绍 `flag` 包的常用函数和基本用法，更详细的内容请查看[官方文档](https://studygolang.com/pkgdoc)。

# 导入flag包

```go
import flag
```

# flag参数类型

`flag`  包支持的命令行参数类型有`bool`、`int`、`int64`、`uint`、`uint64`、`float` `float64`、`string`、`duration`。

|   flag参数   |                            有效值                            |
| :----------: | :----------------------------------------------------------: |
|  字符串flag  |                          合法字符串                          |
|   整数flag   |           1234、0664、0x1234等类型，也可以是负数。           |
|  浮点数flag  |                          合法浮点数                          |
| bool类型flag |  1, 0, t, f, T, F, true, false, TRUE, FALSE, True, False。   |
|  时间段flag  | 任何合法的时间段字符串。如”300ms”、”-1.5h”。 合法的单位有”ns”、”us” /“µs”、”ms”、”s”、”m”、”h”。 |

# 定义命令行flag参数

有以下两种常用的定义命令行 `flag` 参数的方法。

### flag.Type()

基本格式：`flag.Type(flag名, 默认值, 帮助信息)*Type` 

 例如：要定义姓名、年龄、婚否三个命令行参数可以按如下方式定义：

```go
	name := flag.String("name", "张三", "姓名")
	age := flag.Int("age", 18, "年龄")
	married := flag.Bool("m", false, "婚否")
	delay := flag.Duration("d", 0, "时间间隔")
```

需要注意：此时`name`、`age`、`married`、`delay`均为对应类型的指针。

### flag.TypeVar()

基本格式： `flag.TypeVar(Type指针, flag名, 默认值, 帮助信息)` 

例如要定义姓名、年龄、婚否三个命令行参数，可以按如下方式定义：

```go
	var name string
	var age int
	var married bool
	var delay time.Duration
	flag.StringVar(&name, "name", "张三", "姓名")
	flag.IntVar(&age, "age", 18, "年龄")
	flag.BoolVar(&married, "m", false, "婚否")
	flag.DurationVar(&delay, "d", 0, "时间间隔")
```

# 解析命令行参数

通过以上两种方法定义好命令行flag参数后，需要通过调用`flag.Parse()`来对命令行参数进行解析	

支持的命令行参数格式有以下几种：

- `-flag xxx` （使用空格，一个`-`符号）
- `--flag xxx` （使用空格，两个`-`符号）
- `-flag=xxx` （使用等号，一个`-`符号）
- `--flag=xxx` （使用等号，两个`-`符号）

其中，布尔类型的参数必须使用等号的方式指定。

Flag 解析在第一个非flag参数（单个”-“不是flag参数）之前停止，或者在终止符”–“之后停止。

# flag其他函数

```go
flag.Args()  // 返回命令行参数后的其他参数，以[]string类型
flag.NArg()  // 返回命令行参数后的其他参数个数
flag.NFlag() // 返回使用的命令行参数个数
```

# 使用示例

命令行参数使用提示：

```bash
PS D:\GoProject\src> go run .\flagTest.go -help 
Usage of C:\Users\ADMINI~1\AppData\Local\Temp\go-build3280475971\b001\exe\flagTest.exe:
  -age int
        年龄 (default 18)
  -d duration
        时间间隔
  -m    婚否
  -name string
        姓名 (default "张三")
```

正常使用命令行flag参数：

```bash
PS D:\GoProject\src> go run .\flagTest.go -name=wpl -age=18 -m=false --d=1h
wpl 18 false 1h0m0s
[]
0 
4 
```

使用非flag命令行参数：

```bash
PS D:\GoProject\src> go run .\flagTest.go  a b c                           
张三 18 false 0s
[a b c]
3      
0     
```