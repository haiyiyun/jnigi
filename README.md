# JNIGI
Java Native Interface Go Interface.

A package to access Java from Go code. Can be used from a Go executable or shared library.
This allows for Go to initiate the JVM or Java to start a Go runtime respectively.

Docs: https://pkg.go.dev/github.com/haiyiyun/jnigi

## Pull Requests
**Please make sure**: `go test` works, add any doc comments for function signature changes.

## v1
As of 2021-12-05 the master branch will be version 2. Packages that used JNIGI before this should update their go.mod to set v1 as the
version. Or update their code to be compatible with version 2.

## Compile
The `CGO_CFLAGS` needs to be set to add the JNI C header files. The `compilevars.sh` script will do
this.
```
# put this in your build script
source <gopath>/src/github.com/haiyiyun/jnigi/compilevars.sh <root path of jdk>
```

On Windows you can use `compilevars.bat` in the same way (but you don't need `source` at the begining).


## Finding JVM at Runtime
Use the `LoadJVMLib(jvmLibPath string) error` function to load the shared library at run time.
There is a function `AttemptToFindJVMLibPath() string` to help to find the library path.

## Status
* Has been used in Go (many versions since 1.6) executable multi threaded applications on Linux / Windows.
* Tests for main functions are present.

## Example

```` go
package main

import (
    "fmt"
    "github.com/haiyiyun/jnigi"
    "log"
    "runtime"
)

func main() {
    if err := jnigi.LoadJVMLib(jnigi.AttemptToFindJVMLibPath()); err != nil {
        log.Fatal(err)
    }

    runtime.LockOSThread()
    jvm, env, err := jnigi.CreateJVM(jnigi.NewJVMInitArgs(false, true, jnigi.DEFAULT_VERSION, []string{"-Xcheck:jni"}))
    if err != nil {
        log.Fatal(err)
    }

    hello, err := env.NewObject("java/lang/String", []byte("Hello "))
    if err != nil {
        log.Fatal(err)
    }

    world, err := env.NewObject("java/lang/String", []byte("World!"))
    if err != nil {
        log.Fatal(err)
    }

    greeting := jnigi.NewObjectRef("java/lang/String")
    err = hello.CallMethod(env, "concat", greeting, world)
    if err != nil {
        log.Fatal(err)
    }

    var goGreeting []byte
    err = greeting.CallMethod(env, "getBytes", &goGreeting)
    if err != nil {
        log.Fatal(err)
    }

    // Prints "Hello World!"
    fmt.Printf("%s\n", goGreeting)

    if err := jvm.Destroy(); err != nil {
        log.Fatal(err)
    }
}
````
