在写go代码时有时会遇到这样的需求，调用服务或方法得到一个切片，然后这个切片在进一步处理时（例：返回给上层调用）有些值是不需要的，此时就需要找到这些特定的值，然后从切片中删掉。

我们第一反应是遍历找到，然后删除；遍历找到并无问题，但是要在找到后删除就变得不那么简单了，因为slice不是map，没法直接删除一个对应的元素。鉴于此，我们会有两种方案浮现出来：

1. 使用map把所有元素便利存储，然后使用delete直接删除，最后在遍历map重新赋值到slice里面得到想要的结果
2. 在遍历时如果遇到要查询的元素就跳过，否则就把元素append到一个新的slice里面

方案1要遍历slice和map，引入多余的map空间，同时会破坏数据的原有顺序。方案2只需要分配一个新的slice就可以了，并且数据的顺序不会改变。

那么还有没有更好的方式呢？

下面介绍一种方式，只需要遍历一次，且不需要开辟多余的内存空间。

方案3：
根本目的是我们要把找到的元素进行删除，那么我们用一个count来表示最后需要保留的元素的数量，
遍历时，遇到不需要删除的元素count就加1，遇到需要删除的元素时就直接continue，此时count对应的下标就是需要删除的元素。
在执行count加1前我们就可以把下标i和下标count的元素进行替换。即删除了目标元素又保留了原有顺序
代码展示一下

```golang
package main
import "fmt"
func main () {
  s1 := []int{1, 2, 3, 4, 5, 6}
  s2 := []int{1, 2, 5, 5, 5, 6}
  s3 := []int{1, 2, 3, 4}
  s4 := []int{5, 5, 5, 5, 5, 5}
  fmt.Println(delete(s1,5))
  fmt.Println(delete(s2,5))
  fmt.Println(delete(s3,5))
  fmt.Println(delete(s4,5))
}

func delete(s []int, item int) []int{
    length := len(s)
    count := 0
    for i := 0; i < length; i++ {
        if s[i] == item {
            continue
        }
        
        if count < i {
            s[count], s[i] = s[i], s[count]
        }
        
        count++
    }
    
  return s[0:count]
}
```

如上述代码所示，我们只需要用一个count保留元素的数量，不需要多余的空间，时间复杂度也降低了，还能保证原有顺序。

进一步思考一下，我们的业务数据不可能只会是int类型，也有可能是其他的类型，那么当多种类型都需要这种删除操作时我们要对每一种类型都写一个这样的方法吗？

下面我们封装一个通用方法SliceDeleteItem，通过func来判断是否需要删除，这样不管什么类型我们都可以用这个方法来处理，代码如下：

```golang
package main
import "fmt"
import "reflect"

type user struct {
    name string
    age int
}

func main () {
    s1 := []int{1, 2, 3, 4, 5, 6}
    s2 := []string{"one","two","three","four"}
    s3 := []user{{name:"zhangsan",age:33},{name:"lisi",age:34},{name:"laowu",age:23}}
    fmt.Println(SliceDeleteItem(s1,func(index int) bool{
        return s1[index] == 5
    }))

    fmt.Println(SliceDeleteItem(s2,func(index int) bool{
        return s2[index] == "three"
    }))

    fmt.Println(SliceDeleteItem(s3,func(index int) bool{
        return s3[index].name == "lisi"
    }))
}

func SliceDeleteItem(slice interface{}, isDelete func(index int) bool) interface{} {
    sliceValue := reflect.ValueOf(slice)
    if sliceValue.Kind() != reflect.Slice {
        panic("需要传入的类型必须是slice")
    }
    length := sliceValue.Len()
    count := 0

    for i := 0; i < length; i++ {
        if isDelete(i) {
            continue
        }
        if count < i {
            reflect.Swapper(slice)(count, i)
        }

        count++
    }

    return sliceValue.Slice(0, count).Interface()
}
```
