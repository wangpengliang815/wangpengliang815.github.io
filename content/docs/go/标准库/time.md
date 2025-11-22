学习Go语言内置的 `time` 包的基本用法，`time` 包提供了时间的显示和计算用的函数。日历计算采用公历。

时间可分为时间点与时间段，Go 提供了以下两种基础类型
- 时间点 `Time`
- 时间段 `Duration`

除此之外也提供了以下类型，做一些特定的业务
- 时区 `Location`
- `Ticker`
- 定时器 `Timer`

# 时间点

一般使用的所有与时间相关的业务都是基于点而延伸的，两点组成一个时间段，大多数应用也都是围绕这些点与面去做逻辑处理。

## 初始化

go 针对不同的参数类型提供了以下初始化的方式

```go
  // func Now() Time
  fmt.Println(time.Now())

  // func Parse(layout, value string) (Time, error)
  time.Parse("2016-01-02 15:04:05", "2018-04-23 12:24:51")

  // func ParseInLocation(layout, value string, loc *Location) (Time, error) (layout已带时区时可直接用Parse)
  time.ParseInLocation("2006-01-02 15:04:05", "2017-05-11 14:06:06", time.Local)

  // func Unix(sec int64, nsec int64) Time
  time.Unix(1e9, 0)

  // func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time
  time.Date(2018, 1, 2, 15, 30, 10, 0, time.Local)

  // func (t Time) In(loc *Location) Time 当前时间对应指定时区的时间
  loc, _ := time.LoadLocation("America/Los_Angeles")
  fmt.Println(time.Now().In(loc))

  // func (t Time) Local() Time
```

获取到时间点之后为了满足业务和设计，需要转换成需要的格式，也就是所谓的时间格式化。

## 格式化

### to string

格式化为字符串我们需要使用 time.Format 方法来转换成我们想要的格式

      fmt.Println(time.Now().Format("2006-01-02 15:04:05"))  // 2018-04-24 10:11:20
      fmt.Println(time.Now().Format(time.UnixDate))         // Tue Apr 24 09:59:02 CST 2018

Format 函数中可以指定你想使用的格式，同时 time 包中也给了一些我们常用的格式

```
const (
    ANSIC       = "Mon Jan _2 15:04:05 2006"
    UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
    RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
    RFC822      = "02 Jan 06 15:04 MST"
    RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
    RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
    RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
    RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
    RFC3339     = "2006-01-02T15:04:05Z07:00"
    RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
    Kitchen     = "3:04PM"
    // Handy time stamps.
    Stamp      = "Jan _2 15:04:05"
    StampMilli = "Jan _2 15:04:05.000"
    StampMicro = "Jan _2 15:04:05.000000"
    StampNano  = "Jan _2 15:04:05.000000000"
)     
```

注意: galang 中指定的特定时间格式为 "2006-01-02 15:04:05 -0700 MST"， 为了记忆方便，按照美式时间格式 月日时分秒年 外加时区 排列起来依次是 01/02 03:04:05PM ‘06 -0700，刚开始使用时需要注意。

### to time stamp

​      func (t Time) Unix() int64
​      func (t Time) UnixNano() int64

      fmt.Println(time.Now().Unix())
    
      // 获取指定日期的时间戳
      dt, _ := time.Parse("2016-01-02 15:04:05", "2018-04-23 12:24:51")
      fmt.Println(dt.Unix())
    
      fmt.Println(time.Date(2018, 1,2,15,30,10,0, time.Local).Unix())
### 其他

time 包还提供了一些常用的方法，基本覆盖了大多数业务，从方法名就能知道代表的含义就不一一说明了。

      func (t Time) Date() (year int, month Month, day int)
      func (t Time) Clock() (hour, min, sec int)
      func (t Time) Year() int
      func (t Time) Month() Month
      func (t Time) Day() int
      func (t Time) Hour() int
      func (t Time) Minute() int
      func (t Time) Second() int
      func (t Time) Nanosecond() int
      func (t Time) YearDay() int
      func (t Time) Weekday() Weekday
      func (t Time) ISOWeek() (year, week int)
      func (t Time) IsZero() bool
      func (t Time) Local() Time
      func (t Time) Location() *Location
      func (t Time) Zone() (name string, offset int)
      func (t Time) Unix() int64
# 时间类型

`time.Time` 类型表示时间。可以通过 `time.Now()` 函数获取当前的时间对象，然后获取时间对象的年月日时分秒等信息。

示例代码如下：

```go
func getTimeNow() {
	now := time.Now() //获取当前时间
	fmt.Printf("current time:%v\n", now)

	year := now.Year()     //年
	month := now.Month()   //月
	day := now.Day()       //日
	hour := now.Hour()     //小时
	minute := now.Minute() //分钟
	second := now.Second() //秒
	fmt.Printf("%d-%02d-%02d %02d:%02d:%02d\n", year, month, day, hour, minute, second)
}
```

# 时间戳

时间戳是自 `1970年1月1日（08:00:00GMT）`至当前时间的总毫秒数。也被称为 `Unix时间戳`。

示例代码如下：

```go
// 获取时间戳
func getUnix() {
	// 获取时间戳
	timestamp1 := time.Now().Unix()     //时间戳
	timestamp2 := time.Now().UnixNano() //纳秒时间戳
	fmt.Printf("current timestamp1:%v\n", timestamp1)
	fmt.Printf("current timestamp2:%v\n", timestamp2)
}
```

使用 `time.Unix()` 函数可以将时间戳转为时间格式。

```go
func unixConverTime() {
	// 将时间戳转换为时间格式
	timestamp := time.Now().Unix() //时间戳
	fmt.Printf("current timestamp1:%v\n", timestamp)

	timeObj := time.Unix(timestamp, 0) //将时间戳转为时间格式
	fmt.Println(timeObj)
	year := timeObj.Year()     //年
	month := timeObj.Month()   //月
	day := timeObj.Day()       //日
	hour := timeObj.Hour()     //小时
	minute := timeObj.Minute() //分钟
	second := timeObj.Second() //秒
	fmt.Printf("%d-%02d-%02d %02d:%02d:%02d\n", year, month, day, hour, minute, second)
}
```

# 时间间隔

`time.Duration` 是 `time` 包定义的一个类型，它代表两个时间点之间经过的时间，以纳秒为单位。`time.Duration` 表示一段时间间隔，可表示的最长时间段大约290年。

`time` 包中定义的时间间隔类型的常量如下：

```go
const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)
```

例如：`time.Duration` 表示1纳秒，`time.Second` 表示1秒。

# 时间操作

## Add

求时间+n后的时间：

```go
func (t Time) Add(d Duration) Time
```
示例代码如下：
```go
func timeAdd() {
	now := time.Now()
	value := now.Add(time.Hour) // 当前时间加1小时后的时间
	fmt.Println(value)
}
```

## Sub

求两个时间之间的差值：

```go
func (t Time) Sub(u Time) Duration
```

示例代码如下：

```
func timeSub() {
	now := time.Now()
	value := now.Sub(now.Add(time.Hour))
	fmt.Println(value)
}
```

返回一个时间段 `t-u`。如果结果超出了 `Duration` 可以表示的最大值/最小值，将返回最大值/最小值。要获取时间点 `t-d`（d为Duration），可以使用 `t.Add(-d)`。

## Equal

```go
func (t Time) Equal(u Time) bool
```

判断两个时间是否相同，会考虑时区的影响，因此不同时区标准的时间也可以正确比较。本方法和用 `t==u` 不同，这种方法还会比较地点和时区信息。

## Before

```go
func (t Time) Before(u Time) bool
```

如果t代表的时间点在u之前，返回真；否则返回假。

## After

```go
func (t Time) After(u Time) bool
```

如果t代表的时间点在u之后，返回真；否则返回假。

定时器

使用`time.Tick(时间间隔)`来设置定时器，定时器的本质上是一个通道（channel）。

```go
func tickDemo() {
	ticker := time.Tick(time.Second) //定义一个1秒间隔的定时器
	for i := range ticker {
		fmt.Println(i)//每秒都会执行的任务
	}
}
```

# 时间格式化

时间类型有一个自带的方法`Format`进行格式化，需要注意的是Go语言中格式化时间模板不是常见的`Y-m-d H:M:S`而是使用Go的诞生时间2006年1月2号15点04分（记忆口诀为2006 1 2 3 4）。也许这就是技术人员的浪漫吧。

补充：如果想格式化为12小时方式，需指定`PM`。

```go
func formatDemo() {
	now := time.Now()
	// 格式化的模板为Go的出生时间2006年1月2号15点04分 Mon Jan
	// 24小时制
	fmt.Println(now.Format("2006-01-02 15:04:05.000 Mon Jan"))
	// 12小时制
	fmt.Println(now.Format("2006-01-02 03:04:05.000 PM Mon Jan"))
	fmt.Println(now.Format("2006/01/02 15:04"))
	fmt.Println(now.Format("15:04 2006/01/02"))
	fmt.Println(now.Format("2006/01/02"))
}
```

# 解析字符串格式的时间

```go
now := time.Now()
fmt.Println(now)
// 加载时区
loc, err := time.LoadLocation("Asia/Shanghai")
if err != nil {
	fmt.Println(err)
	return
}
// 按照指定时区和指定格式解析字符串时间
timeObj, err := time.ParseInLocation("2006/01/02 15:04:05", "2019/08/04 14:15:20", loc)
if err != nil {
	fmt.Println(err)
	return
}
fmt.Println(timeObj)
fmt.Println(timeObj.Sub(now))
```

练习题：