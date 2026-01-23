---
tags:
  - docs
  - reference
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Golang

Created: 2019年7月16日 上午11:51
Last Edited Time: 2019年7月16日 下午2:56
Created By: Aaron Kuo
Last Edited By: Aaron Kuo

### Collection Functions

example

[https://gobyexample.com/collection-functions](https://gobyexample.com/collection-functions)

### Error Handling

[https://blog.wu-boy.com/2017/03/error-handler-in-golang/](https://blog.wu-boy.com/2017/03/error-handler-in-golang/)

### Regular Expression

[https://golang.org/pkg/regexp/#example_Match](https://golang.org/pkg/regexp/#example_Match)

### Sorting

[https://golang.org/pkg/sort/](https://golang.org/pkg/sort/)

### JSON

[https://yourbasic.org/golang/json-example/](https://yourbasic.org/golang/json-example/)

### Date / Time

[https://tecadmin.net/get-current-date-time-golang/](https://tecadmin.net/get-current-date-time-golang/)

### AWS Lambda

[https://djhworld.github.io/post/2018/01/27/running-go-aws-lambda-functions-locally/](https://djhworld.github.io/post/2018/01/27/running-go-aws-lambda-functions-locally/)

## Slice - Loop / Iteration / Array

[https://blog.golang.org/slices](https://blog.golang.org/slices)

[https://github.com/golang/go/wiki/SliceTricks](https://github.com/golang/go/wiki/SliceTricks)

[https://medium.com/rungo/the-anatomy-of-slices-in-go-6450e3bb2b94](https://medium.com/rungo/the-anatomy-of-slices-in-go-6450e3bb2b94)

[https://yourbasic.org/golang/for-loop-range-array-slice-map-channel/](https://yourbasic.org/golang/for-loop-range-array-slice-map-channel/)

### Slice

Built on fixed-size arrays to give a flexible, extensible data structure.

Slice is a data structure describing a contiguous section of an array.

*A slice describes a piece of an array*

Note: Issue of writing code of removing AMI

[https://yourbasic.org/golang/gotcha-change-value-range/](https://yourbasic.org/golang/gotcha-change-value-range/)

```markdown
The range loop copies the values from the slice to a local variable n; updating n will not affect the slice.
```

### 'Append' slice

```go
groups := []string{"A","B","C"}
groups = append(groups, "X", "Y", "Z")
```

Understand slice header, length, capacity, copy & make

### Gotcha

[https://medium.com/@Jarema./golang-slice-append-gotcha-e9020ff37374](https://medium.com/@Jarema./golang-slice-append-gotcha-e9020ff37374)

print pointer

```go
value := "foobar"
vp = &value
fmt.Printf("pointer is %p", vp)
```

## Covertion

### int, int64 and string

[https://yourbasic.org/golang/convert-int-to-string/](https://yourbasic.org/golang/convert-int-to-string/)

Find Type

```go
import (
	"reflect"
	"fmt"
)

func main(){
	t1 := "string"
	fmt.Println(reflect.TypeOf(t1))
}
```