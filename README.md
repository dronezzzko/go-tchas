# Go-tchas
List of interesting gotchas in Go.

* [Dark Corners](#dark-corners)
    * [Pointer of composite literals](#pointer-of-composite-literals)
    * [Loop types](#loop-types)
    * [Copy slice without using copy() function](#copy-slice-without-using-copy-function)
    * [Comparable slices](#comparable-slices)
    * [NaN != NaN, Inf == Inf](#nan--nan-inf--inf)
    * [bytes.Equal and reflect.DeepEqual give different results](#bytesequal-and-reflectdeepequal-give-different-results)
* [Performance](#performance)
    * [for-range loop](#for-range-loop)

## Dark Corners
### Pointer of composite literals

There is a way to create a pointer of composite literal without a helper function.

Won't work:
```go
var pb = &true 
var pi = &123 
var pb = &"abc"
```

One-line "hack" that actually works:
```go
var pb = &(&[1]bool{true})[0] 
var pi = &(&[1]int{9})[0]  
var ps = &(&[1]string{"Go"})[0]
```
Or:
```go
var pb2 = &([]bool{true})[0]  
var pi2 = &([]int{9})[0]
var ps2 = &([]string{"Go"})[0]
```

### Loop types
Looks wierd but valid:
```go
type S []S
type M map[int]M
type F func(F) F
type Ch chan Ch
type P *P
```
```go
package main

func main() {
	type P *P
	var pp = new(P)
	*pp = pp
	_ = ************pp
}
```
```go
package main

type F func() F

func f() F {
	return f
}

func main() {
	f()()()()()()()()()
}
```

### Copy slice without using ``copy()`` function
Since Go 1.17 a slice can be copied in this way:
```go
package main

const N = 128
var x = []int{N-1: 789}

func main() {
	var y = make([]int, N)
	*(*[N]int)(y) = *(*[N]int)(x) // the same as: copy(y, x) 
	println(y[N-1]) // 789
}
```

### Comparable slices
As we know in Go, slices are incomparable. Well... Since Go 1.17, if the elements of two slices are comparable and the lengths of the two slices are equal and known at coding time, then we could this:
```go
package main

func main() {
	var x = []int{1, 2, 3, 4, 5}
	var y = []int{1, 2, 3, 4, 5}
	var z = []int{1, 2, 3, 4, 9}

	println(*(*[5]int)(x) == *(*[5]int)(y)) // true
	println(*(*[5]int)(x) == *(*[5]int)(z)) // false
}
```

### NaN != NaN, Inf == Inf
```go
package main

var a = 0.0
var x = 1 / a // +Inf
var y = x * a // NaN

func main() {
	println(x, y)   // +Inf NaN
	println(x == x) // true
	println(y == y) // false
}
```
### bytes.Equal and reflect.DeepEqual give different results
```go
package main

import (
	"bytes"
	"reflect"
)

func main() {
	var x = []byte{}
	var y []byte
	println(bytes.Equal(x, y)) // true
	println(reflect.DeepEqual(x, y)) // false
}
```
## Performance
### for-range loop
With each iteration of a slice, the address of a local variable is the same. Only value of the local variable is changing.

Several examples below show the costs of copying some large values:
```go
package copybench

import (
	"testing"
)

type (
	bigStruct struct {
		h     uint64
		cache [500]byte
		body  []byte
	}
)

func BenchmarkRangeValueCopy(b *testing.B) {
	var sum uint64 = 0
	var bigSlice = make([]bigStruct, 2000)
	b.ResetTimer()

	b.Run("range_value_copy", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			for _, v := range bigSlice {
				sum += v.h
			}
		}
	})

	b.Run("range_value_index", func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			for ii := range bigSlice {
				sum += bigSlice[ii].h
			}
		}
	})

	_ = sum
}

```
Output:

```terminal
BenchmarkRangeValueCopy/range_value_copy-8                 26079             43284 ns/op               0 B/op          0 allocs/op
BenchmarkRangeValueCopy/range_value_index-8               488025              2453 ns/op               0 B/op          0 allocs/op
```
From the results, we could learn that the performance of the benchmark ``range_value_copy`` is much lower than the ``range_value_index``. The reason is every element is copied to the iteration variable ``v`` in the benchmark ``range_value_copy``, and the copy cost is not small. The other benchmark avoid the copies.
