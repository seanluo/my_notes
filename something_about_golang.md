##golang
* `for`循环中使用闭包时，一定要显示向闭包函数传递某个循环变量，否则闭包只会使用第一次的值
* 使用WaitGroup

    import (
    	"sync"
    )
    
    type WaitGroupWrapper struct {
    	sync.WaitGroup
    }
    
    func (w *WaitGroupWrapper) Wrap(cb func()) {
    	w.Add(1)
    	go func() {
    		cb()
    		w.Done()
    	}()
    }

这种方法将`WaitGroup`包装起来，其他的struct中需要等待一系列的goroutine返回时，先生成一个`WaitGroupWrapper`对象，再用该对象的`Wrap`方法包裹要并行且等待的函数，一般是匿名函数的形式。最后调用者通过该对象的`Wait()`方法等待所有函数返回。