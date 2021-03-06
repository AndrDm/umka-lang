![](logo.png)

# Welcome to Umka
Umka is a statically typed embeddable scripting language. It combines the simplicity and flexibility needed for scripting with a compile-time protection against type errors. Its aim is to follow the Python Zen principle _Explicit is better than implicit_ more consistently than dynamically typed languages generally do.

## Features
* Clean syntax inspired by Go
* Cross-platform bytecode compiler and virtual machine
* Garbage collection
* Polymorphism via interfaces
* Multitasking based on fibers
* Type inference
* Distribution as a dynamic library with a simple C API
* C99 source

## Getting Started
* Download [release 0.1](https://github.com/vtereshkov/umka-lang/releases) for Windows and Linux (_Note:_ both versions use `/` as path separator)
* Take a tour of Umka (see below)
* Explore the [raytracer example](https://github.com/vtereshkov/umka-lang/blob/master/examples/raytracer.um) that demonstrates many language features like fibers, interfaces and dynamic arrays

![](examples/scene.png)

## A Tour of Umka
### Declarations
```
// Constants
const a = 3
const b* = 2.38  // Exported identifier
const (
    c = sin(b) / 5
    d = "Hello" + " World"
)

// Types
type IntPtr = ^uint16
type Arr = [a]real
type (
    Quat = struct {
        q: [4]real
        normalized: bool
    }
    Printable = interface {
        print(): int
    }
    ErrFn = fn(code: int)
)        

// Variables
var e: int
var f: str = d + "!"
var (
    g: Arr = [3]real{2.3, -4.1 / 2, b}
    h: [var][5]int  // Dynamic arrays require calling make()
)
q := Quat{q: [4]real{1, 0, 0, 0}, normalized: true}

// Functions
fn tan(x: real): real {return sin(x) / cos(x)}

/*
Built-in functions:
    printf fprintf sprintf scanf fscanf sscanf
    round trunc fabs sqrt sin cos atan exp log
    new make append len sizeof
    fiberspawn fiberfree fibercall fiberalive
*/

// Methods
fn (a: ^Arr) print(): int {
    printf("Arr: {8.3lf} {8.3lf} {8.3lf}\n", a[0], a[1], a[2])
    return 0
}
```
### Statements
```
// Assignment
h = make([var][5]int, 3)

// Declaration via assignment (with type inference)
sum := 0.0

// Function call
y := tan(30 * std.pi / 180)
h = append(h, [5]int{10, 20, 30, 40, 50})

// Method call
g.print()

// Conditional execution
if err := getError(); !err {
    std.println("Done")
}

// Switch
switch a {
    case 1, 3, 5, 7: std.println(std.itoa(a) + " is odd")
    case 2, 4, 6, 8: std.println(std.itoa(a) + " is even")
    default:         std.println("I don't know")
}

// Loop
for i := 0; i < len(g); i++ {
    if fabs(g[i]) > 1e12 {break}
    if g[i] < 0 {continue}
    sum += g[i]
}
```
### Multitasking
```
fn childFunc(parent: std.Fiber, buf: ^int) {
    for i := 0; i < 5; i++ {
        std.println("Child : i=" + std.itoa(i) + " buf=" + std.itoa(buf^))
        buf^ = i * 3
        fibercall(parent)
    }
}

fn parentFunc() {
    a := 0
    child := fiberspawn(childFunc, &a)    
    for i := 0; i < 10; i++ {
        std.println("Parent: i=" + std.itoa(i) + " buf=" + std.itoa(a))
        a = i * 7
        if fiberalive(child) {
            fibercall(child)
        }
    }    
    fiberfree(child)
}
```
