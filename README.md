# goRecycleBuffer

## What is this?
`goRecycleBuffer` is for effective buffer usage in golang and thread-safe.
It is wrote with very short and simple code.

## Installation
Use following command in your terminal.
`go get github.com/DiyLecko/goRecycleBuffer`

## How to use?
1. Import `import "github.com/DiyLecko/goRecycleBuffer"`
2. Init with `goRecycleBuffer.Init(8192) // 1th param is bufferSize.`
3. Use `buf := <-goRecycleBuffer.Get` to get buffer
4. Use `goRecycleBuffer.Give<- buf` to free buffer.

here is an example.
```
// goRecycleBuffer_test.go
package goRecycleBuffer

import (
	"math/rand"
	"runtime"
	"testing"
	"time"
)

func init() {
	runtime.GOMAXPROCS(runtime.NumCPU())
}

func TestRecycleBuffer(t *testing.T) {
	Init(8192)

	buf := <-Get
	Give <- buf
}

var doneTest2 chan bool = make(chan bool)
var endCountTest2 int = 100
var testCountTest2 int = 10
var countText2 int = 0

func testRecycleBuffer2() {
	for i := 0; i < testCountTest2; i++ {
		time.Sleep(time.Millisecond * time.Duration(rand.Int63()%500))
		buf := <-Get
		go func() {
			time.Sleep(time.Millisecond * time.Duration(rand.Int63()%1000))
			Give <- buf
			countText2++

			if countText2 == endCountTest2*testCountTest2 {
				doneTest2 <- true
			}
		}()
	}
}
func TestRecycleBuffer2(t *testing.T) {
	Init(8192)

	for i := 0; i < endCountTest2; i++ {
		go testRecycleBuffer2()
	}

	<-doneTest2

	t.Logf("buffer count : %d\n", GetBufferCount())
}
```

result is
```
=== RUN   TestRecycleBuffer
--- PASS: TestRecycleBuffer (0.00s)
=== RUN   TestRecycleBuffer2
--- PASS: TestRecycleBuffer2 (4.58s)
	goRecycleBuffer_test.go:50: buffer count : 206
PASS
ok  	goRecycleBuffer	4.588s
```

In above example, buffer is made 1000 times. But in result, buffer is made only 206 times.
