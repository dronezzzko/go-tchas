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
