## 1. 服务启动
### 1.1 init() 与main()的执行顺序，多个包的init()，同一个包的init()
init()–>main()
与import的顺序相关，相同包的init()执行顺序不做保障
## 2. 字符串原理 len函数
```go
func TestLen(t *testing.T) {
   str := "字节欢迎你"
   fmt.Println(str)
}
```

输出结果是多少：15<br>
```go
//string是8位字节的集合，通常但不一定代表UTF-8编码的文本。
//string可以为空，但不能为nil。string的值是不能改变的
type string string
存储结构：只读的字节切片
源码的编码方式：UTF-8 ，英文占1个字节，中文占3个字节
延伸问题：如何求字符串长度？
len([]rune(str))
type rune = int32
```
### 2.1 延展问题，Go语言中int占几个字节
与操作系统位数相关，32位操作系统 -> 4个字节，64位操作系统 -> 8个字节
## 3. 并发原理 for中协程创建
```go
func TestGoRoutine(t *testing.T) {
    for i := 0; i < 10; i++ {
       go func() {
          fmt.Println(i)
       }()
    }
}
```
随机数字
原因：**golang是值拷贝传递**。for循环很快，协程创建需要的时间大于for循环的时间。<br>
因为协程创建 需要进行 堆栈分配，上下文准备，以及与内核态的线程进行映射工作等。<br>
所以在协程创建好后，大家同时去访问tmp变量，这个时候 tmp 就被多个协程共享了，导致取到的值都一样了。<br>

如何解决？
```go
func TestGoRoutine(t *testing.T) {
    for i := 0; i < 10; i++ {
       tmp := i
       go func() {
          fmt.Println(tmp)
       }()
    }
}

func TestGoRoutine(t *testing.T) {
    for i := 0; i < 10; i++ {
       go func(tmp int) {
          fmt.Println(tmp)
       }(i)
    }
}
```
## 4. 基础问题：垃圾回收机制
[垃圾回收机制](https://learnku.com/articles/59021)<br>
 GC 什么时候会被触发呢？<br>
一是堆内存的分配达到控制器计算的触发堆大小，初始大小环境变量 GOGC，之后堆内存达到上一次垃圾收集的 2 倍时才会触发 GC。<br>
二是如果一定时间内没有触发，就会触发新的循环，该触发条件由 runtime.forcegcperiod 变量控制，默认为 2 分钟。<br>