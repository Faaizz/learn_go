# Learn Go

## Content
- [Learn Go](#learn-go)
  - [Content](#content)
  - [Useful Commands](#useful-commands)
  - [A Tour of Go](#a-tour-of-go)
    - [Basic Types](#basic-types)
      - [Casting](#casting)
      - [Constants](#constants)
    - [Looping](#looping)
    - [Conditionals](#conditionals)
      - [`if`](#if)
      - [`switch`](#switch)
    - [`defer`](#defer)
    - [`struct`](#struct)
    - [Arrays](#arrays)
    - [Slices](#slices)
      - [Creating a slice with `make`](#creating-a-slice-with-make)
      - [Appending to slices](#appending-to-slices)
    - [Range](#range)
    - [Maps](#maps)
      - [Mutating Maps](#mutating-maps)
    - [Functions](#functions)
      - [Closures](#closures)
      - [Methods](#methods)
    - [Interfaces](#interfaces)
      - [Interface values with nil underlying values](#interface-values-with-nil-underlying-values)
      - [Empty Interfaces and Type Assertions](#empty-interfaces-and-type-assertions)
      - [The `Stringer` Interface](#the-stringer-interface)
      - [The `error` Interface](#the-error-interface)
      - [The `io.Reader` Interface](#the-ioreader-interface)
    - [Generics](#generics)
      - [Generic Functions](#generic-functions)
      - [Generic Types](#generic-types)
    - [Goroutines](#goroutines)
    - [Channels](#channels)
      - [Buffered Channels](#buffered-channels)
      - [`range` and `close`](#range-and-close)
      - [`select`](#select)
      - [Equivalent Binary Trees Example](#equivalent-binary-trees-example)
    - [`sync.Mutex`](#syncmutex)
  - [References](#references)

## Useful Commands
```shell
# Organize dependencies
go mod tidy
# Remove downloaded modules
go clean -modcache
# Print install directory
go list -f '{{.Target}}'
# Add install directory to PATH
export PATH=$PATH:$(dirname $(go list -f '{{.Target}}'))
# Change install directory
go env -w GOBIN=/path/to/install/bin
# Or GOPATH (Go checks the 'bin' directory of this path)
go env -w GOPATH=/path/to/install
# Unset ENV
go env -u GOPATH
```

## A Tour of Go
### Basic Types
```go
bool
string
int int8 int16 int32 int64
uint uint8 uint16 uint64 uintptr
byte // alias for uint8
rune // alias for int32 (a Unicode code point)
float32 float64
complex64 complex128
```
Print a variable's type using:
```go
fmt.Println("varName has type: %T", varName)
```
#### Casting
Go requires explicit casting **always**.
#### Constants
Are declared like variables but cannot use the simple assignment syntax (`:=`).
```go
// Typed constant
const Pi int = 3.14
// Untyped numeric constant: can functions as an `int` or `float` depending on the context where it's used
const Rho = 0.002
```

### Looping
Only `for` loop construct is available. Unlike other popular languages, no parenthesis are required around loop components and curly braces are **always** required around the code:
```go
for idx:=0; idx<10; idx++{
    fmt.Println(idx)
}
```
Like other popular languages, the initialization and increment(post) statements are optional:
```go
sum := 10
for ; sum<100; {
    sum*=2
    fmt.Println(sum)
}
```
But again, unlike other popular languages, the semicolons can be dropped such that `for` seems like a `while`. Ommiting the loop condition implies a `true` and loops forever:
```go
cond:= true
idx:= 5
for cond {
    fmt.Println(cond)
    if idx<10{
        cond= false
    }
    idx++
}
```
To get a forever loop, you can omit all three statements all-together
```go
for {
    if status := upload.Status(); status == true {
        break
    }
}
```

### Conditionals
#### `if`
`if` like `for`doesn't require parenthesis around the condition. 
Further, it allows a short statement to execute before evaluating the condition:
```go
if cond:=true; cond{
    fmt.Println("Always true!")
}
```
Variables declared in an `if`'s short statement are only available in the `if` block and in blocks of associated `else`s.

#### `switch`
In go, `switch`statements also allow the short statement as in `if` and `for`.
`switch` statements **only** run the first `case` whose value matches the condition. Hence, `break` statements are not need. Also, the value of `case`statements need not be constants nor integers, they can be any valid expression that provides a value (e.g. a function call):
```go
switch os:=runtime.GOOS; os{
    case "darwin":
        fmt.Println("Running on a mac!")
    case printLinux():
        fmt.Println("Running on a linux machine")
    default:
        fmt.Println("Maybe a windows PC?")
}
```
Further, a `switch` with no condition implies a `true`:
```go
switch {
    case 1>2:
        fmt.Println("Never!")
    case 1==6:
        fmt.Println("Nooooo")
    default:
        fmt.Println("Finally...")
}
```

### `defer`
A `defer`statement delays execution of a function till the surrounding function returns.
However, evaluation of expression(s) that serve as arguments to the function (if any) is not delayed.
```go
func greetMe(){
    defer fmt.Println("You")

    fmt.Println("Hey")
}
```
Deferred function calls are pushed onto a stack (if there's multiple) and are executed in a LIFO order after the function returns.

### `struct`
A `struct`is a collection of fields.
```go
// Defining a struct
type Quad struct{
    x, y int
}

var(
    square= Quad{2,2}
    invalid= Quad{x: 10} // y is implied to be default value (0 for int)
    another_invalid= Quad{} // x,y implied to be 0
    rectPtr= &Quad{2,4} // Returns pointer to the struct
)

func main(){
    square_len:= square.x
    // Struct pointers can be used to access fields
    rect_len:= rectPtr.x
}
```

### Arrays
`[n]T` is an `n`-element array of type `T`.
```go
// Array of 10 integers
var intArr= [10]int

func main(){
    // Assignment
    intArr[0]= 10
    // Short assignment
    primes:= [6]int{2,3,5,7,11,13}
    // Slice the array with index 0 included and 3 excluded
    fmt.Println(primes[0:3])
}
```

### Slices
Are dynamically sized view into an array. `[]T` is a slice with elements of type `T`.
```go

var primes= [6]int{2,3,5,7,11,13}
// Slice the array with index 0 included and 3 excluded
var aSlice= primes[0:3]
```
Note that changing the elements of a slice modifies the element in the underlying array.
Slice literals create the underlying array, then build a slice that references it:
```go
var aSLiceLit= []string{"Elements", "of", "underlying", "array."}
```
Slice indexing is similar to indexing in numpy:
```go
var numArr [10]int

// All reference the entire array
numArr[0:10]
numArr[0:]
numArr[:10]
numArr[:]
```
A slice has a lenght and a capacity. Its length is the number of elements it contains, while its capacity is the number of elements of the underlying array counting from the first element referenced by the slice.
The length and capacity of a slice can be changed by re-slicing:
```go
func main(){
    // Underlying array
    s:= [6]int{1,2,3,4,5,6}

    // Slice with length 0, capacity 6
    s= s[:0]
    fmt.Println("Length and cap ", len(s), cap(s))

    // Re-slicing
    // Slice with length 2, capacity 6
    s= s[:2]
    fmt.Println("Length and cap ", len(s), cap(s))

    // Re-slicing
    // Slice with length 2, capacity 4
    s= s[2:]
    fmt.Println("Length and cap ", len(s), cap(s))
}
```
The zero value of a slice is `nil`. I.e. no underlying array and zero length, zero capacity.

#### Creating a slice with `make`
Second argument is the length of the slice, third is its capacity (optional).
```go
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```

#### Appending to slices
```go
s:= []int{0,9,8}
s= append(s, 1,2,4)
```
If the underlying array is too small, a new array is allocated and referenced by the slice.

### Range
This form of `for` loop iterates over a slice or a map:
```go
aSlice:= []int{1,2,3,4,5,6}

for idx,val := range(aSlice){
    fmt.Println(idx,val)
}
```
Either the index or the value can be ignored by assigning to `_`.
Note that the value is a **copy** of the actual value (i.e. passed by value, not by reference), thus changing this value doesn't affect the underlying array.

### Maps
Map keys to values and have a zero value of `nil`. 
`make` returns a map of a given type, initialized and ready for use.
```go

type Quad struct{
    length, height float64
}

var squares= map[string]Quad

func main(){
    rectangles:= make(map[string]Quad)

    rectangles["a"]= Quad{12.4,17.6}
    fmt.Println(squares["a"])
}

```
Maps can also be initialized using literals. They work just like `struct` literals but the keys must always be present:
```go
litMap:= map[string]string{
    "viel": "plenty",
    "wenig": "little",
}
```
#### Mutating Maps
```go
// Insert/update
litMap[key]
// Retrieve
elem= litMap[key]
// Delete
delete(litMap, key)
// Check that key is present
// ok is true if yes, false otherwise
elem, ok:= litMap[key]
if !ok{
    fmt.Println("Key is not present")
}
```

### Functions
Go has 1st calss functions, i.e.., functions are values also.
They can be passed around as function arguments, returned from a function, and can be assigned to variables.
```go
func execFunc(x, y float64, fn func(float64, float64) error) bool {
    err := fn(x, y)
    if err == nil {
        return true
    }
    return false
}

func main() {
    myFunc := func (x, y float64) error {
        if x>y {
            return errors.New("big error")
        }
        return nil
    }

    fmt.Printf("Exec result: %v", execFunc(3, 4, myFunc))
}
```
#### Closures
Closures are functions that reference variables outside the function body.
Since Go only "passes by value", a closure "encloses" a copy of the variables it references and keeps a local copy which is only visible withing the scope if the closure.

#### Methods
Go does not have classes, but methods can be defined on types.
A method is a function with a special receiver parameter which appers in the method's parameter list between the `func` keyword and the method's name, 
the variable on which the method is called is passed as the receiver argument.
```go
type PytDouble struct{
    a,b float64
}

func (pytD PytDouble) third() float64{
    return math.Sqrt(pytD.a*pytD.a + pytD.b*pytD.b)
}

func main(){
    aPytDouble:= PytDouble{3.0,4.0}
    fmt.Println(aPytDouble.third())
}
```
You can define methods with pointer recievers to allow mutation of the receiver argument.
In this case, a pointer to the variable on which the method is called is passed as the receiver argument:
```go
type Account struct{
    name string
    balance float64
}

func (balPtr *Account) deduct(amount float64){
    balPtr.balance-=amount
}

func main(){
    myAcc:= Account{"Faizudeen", 12700.0}
    myAcc.deduct(5000)
    fmt.Println(myAcc.balance)
}
```

### Interfaces
Are defined as a set of method signatures. 
A variable of an interface type can hold any value that implements those methods.
A type implements an interface my implementing its methods, 
there's no explicit declaration such as "implements" in Java.
```go
type Shape interface{
    area() float64
    perimeter() float64
}

type RightTriangle struct{
    base, height float64
}

func (tri RightTriangle) area() float64{
    return 0.5*tri.base*tri.height
}
func (tri RightTriangle) perimeter() float64{
    hyp:= math.Sqrt(tri.base*tri.base + tri.height*tri.height)
    return hyp + tri.base + tri.height
}

type Square struct{
    length float64
}
func (square *Square) area() float64{
    return square.length*square.length
}
func (square *Square) perimeter() float64{
    return 4*square.length
}

func main() {
	var aShape Shape
	aSq:= Square{4}
    /* "aShape= aSq" gives an error because Square (the value) does NOT
     * implement area() and perimeter(), *Square (the pointer type) does
     */
    aShape= &aSq
	fmt.Println(aShape.area())

    aTri:= RightTiangle{3,4}
    aShape= aTri // Works because RightTriangle (the value) implements the required functions
    fmt.Println(aShape.perimeter())

    // To get the concrete type behind an interface, use the %T print format option
    fmt.Printf("Concrete type: %T", aShape)
}
```
#### Interface values with nil underlying values
In Go, it is perfectly okay for an interface to have a `nil` underlying concrete value.
However, note that the interface itself is **not** `nil`.
And methods can be constructed such that they can gracefully handle being invoked on a `nil` receiver.
```go

type Comer interface {
    Come() bool
}

type Overcomer struct {
    CanCome bool
}

func (o *Overcomer) Come() bool {
    if o == nil {
        return false
    }
    return o.CanCome
}

func main() {
    var c Comer
    var o *Overcomer
    c = o
    fmt.Printf("nil Overcomer: %v is non-nil Comer: %v\n", o, c)
    fmt.Println(c.Come())
}
```
#### Empty Interfaces and Type Assertions
An empty interface is an interface that specifies no methods.
An empty interface can hold concrete values of any type.
A type assertion returns a tuple of the interface value's underlying value and a boolean indicating whether or not it is of the specified type:
```go

func main(){
    // Empty interface
    var i interface{} 
    i = "hello"
    // Type assertion
    val, ok:= i.(string)
    // ok is true

    // Type assertion
    val, ok:= i.(uint8)
    // ok is false
}
```
An empty interface is often used in parameter definition of functions that take unknown values as arguments.
```go
// Type switch example
func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	do(21)
	do("hello")
	do(true)
}
```

#### The `Stringer` Interface
Is very popular. It is a type that can describe itself as a string. 
The `fmt` package and other packages look for this interface to print values.
The interface is defined as:
```go
type Stringer interface {
    String() string
}
```
A sample implementation is:
```go
type Person struct{
    name, gender string
    age int
}

func (p Person) String() string{
    return fmt.Sprintf("%v is %v and %v years old.", p.name, p.gender, p.age)
}
```

#### The `error` Interface
The `error`type is a built-in interface defined as:
```go
type error interface {
    Error() string
}
```
`fmt`and other packages often check for the `error` interface when printing values.
A `fmt.Println(err)` calls the `Error()` method of the underlying value of `err`. E.g.:
```go
i, err := strconv.Atoi("42")
if err != nil {
    fmt.Printf("couldn't convert number: %v\n", err)
    return
}
fmt.Println("Converted integer:", i)
```

#### The `io.Reader` Interface
The `io.Reader` interface represents the read end of a data stream.
The Go standard library containes many implementations of this interface.
`Read`populates a given byte slice with data and returns the number of bytes populated, and an error value. When the stream ends, `io.EOF` error is returned.

### Generics
#### Generic Functions
Type parameters of a function appear in square brackets, before the function's parameters.
E.g.:
```go
func Index[T comparable](s []T, t T) int {
    ...
}
```
`comparable` used here is a built-in constraint that type `T` (in this example) must fulfil.
`comparable` makes it possible to use `==` and `!=` on a type.
Other constraints may be used in its place as required.

#### Generic Types
Go also supports generic types.
Type parameters could be added to a type declaration as shown below:
```go
type List[T any] struct {
    next *List[T]
    val T
}
```

### Goroutines
A *goroutine* is a lightweight thread managed by the Go runtime
```go
go f(x,y,x)
```
Runs the function `f` in a new goroutine.
`f`, `x`,`y`, and `z` are evealuated in the current goorutine, but the execution of `f` happens in the new goroutine. Note **evaluation** differs from **execution**.


### Channels
A channel is a typed conduit through which values can be sent and recieved. 
By default, sending and receiving is blocked until the other side is ready, providing built-in synchronization.
Lime Maps and Slices, Channels must be created before use
```go
# Create an int channel
ch_int:= make(chan int)

# Send value over channel
ch_int <- 200

# Get value from channel
res:= <- ch_int
```

#### Buffered Channels
```go
buff_len:= 10
# Make buffered channel
buff_ch_int:= make(chan int, buff_len)
```
Senders block when the buffer is full, while receivers block when the buffer is empty.

#### `range` and `close`
A channel can be closed by a sender to indicate receiers that no more values would be sent over the channel. 
This is not a requirement as with files.
The `range` loop pulls values from a channel until the channel is closed.
```go
func producer(chann chan int){
    for idx:=0;idx<10;idx++{
        chann <- idx*2
    }

    close(chann)
}

func main(){
    chann:= make(chan int)

    go producer(chann)

    for val := range(chann){
        fmt.Println(val/2)
    }
}
```

We can also explicitly check if a channel is closed by assigning a second variable in the receiver expression, which is `false` if the channel is closed.
```go
val, ok:= <- chann
// ok is false if channel is closed
```

#### `select`
A `select`statements lets a goroutine block until one of its cases can run.
If multiple cases can run at the same time, it selects one at random.
```go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```
A `default` case can also specified, this is run if no other case is ready.

#### Equivalent Binary Trees Example
```go
package main

import (
	"golang.org/x/tour/tree"
	"fmt"
)

// Compare Slices
func testEq(a, b []int) bool {

    // If one is nil, the other must also be nil.
    if (a == nil) != (b == nil) { 
        return false; 
    }

    if len(a) != len(b) {
        return false
    }

    for i := range a {
        if a[i] != b[i] {
            return false
        }
    }

    return true
}


// Walk walks the tree t sending all values
// from the tree to the channel ch.
func Walk(t *tree.Tree, ch chan int){
	// Check left branch
	if t.Left != nil{
		Walk(t.Left, ch)
	}
	
	// Send value to channel
	ch <- t.Value
	
	// Check right branch
	if t.Right != nil{
		Walk(t.Right, ch)
	}
	
	// End
	if t.Value==10{
		close(ch)
	}
}

// Same determines whether the trees
// t1 and t2 contain the same values.
func Same(t1, t2 *tree.Tree) bool{

	chan1:= make(chan int)
	go Walk(t1, chan1)
	//seq1:= make([]int)
	seq1:= []int{}
	for val:=range(chan1){
		seq1= append(seq1, val)
	}
	
	chan2:= make(chan int)
	go Walk(t2, chan2)
	//seq2:= make([]int)
	seq2:= []int{}
	for val:=range(chan2){
		seq2= append(seq2, val)
	}
	
	// Compare
	return testEq(seq1, seq2)
}

func main() {
	myTree:= tree.New(1)
	treeChan:= make(chan int)
	go Walk(myTree, treeChan)
	seq:= []int{}
	for val:= range(treeChan){
		seq= append(seq,val)
	}
	fmt.Println(seq)
	
	compared:= Same(tree.New(1), tree.New(1))
	fmt.Println(compared)
}
```

### `sync.Mutex`
Mutual Exclusion is the concept of preventing conflicts by ensuring that only one routine/function/method can mutate a variable at a point in time.
*Mutex* is the conventional name for the data structure that provides this concept.
Go's standard library provides mutual exlusion with `sync.Mutex`.
```go
// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	mu sync.Mutex
	v  map[string]int
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mu.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mu.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mu.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```
We can use `defer`to ensure that the mutex is unlocked as shown above in `Value` function.


## References
- [A Tour of Go](https://tour.golang.org)
- [Checking equality of two slices- Stack Overflow](https://stackoverflow.com/questions/15311969/checking-the-equality-of-two-slices)
