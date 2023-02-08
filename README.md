# Go-tchas
List of interesting gotchas in Go.

1. [Undocumented Behavior](#undocumented-behavior)

## Undocumented Behavior
### Pointer of composite literals

There is a way to create a pointer of composite literal without a helper function.

Won't work:
```go
var pb = &true 
var pi = &123 
var pb = &"abc"
```

One-line "hacks" that actually work:
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

func main() { type P *P
  var pp = new(P) *pp = pp
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
