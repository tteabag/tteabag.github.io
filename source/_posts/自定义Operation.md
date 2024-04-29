---
title: 自定义 Operation
date: 2022-05-19 14:52:13
tags:
---

# 自定义Operation

operation分两种，同步和异步

## 1.同步
 同步的定义，只需要重写main()方法，main方法内部不能调用其他异步函数，因为main()方法执行完成，operation即执行结束。
 
<!-- more -->
 
```
class SyncOperation: Operation {   
    override func main() {
        print("执行同步任务")
    }
}
```

## 2.异步
异步方法需要重写如下方法：

* var isAsynchronous: Bool { get }
* override var isExecuting: Bool { get }
* override var isFinished: Bool { get }

OperationQueue通过监听上面方法确定任务的状态，然后在start()方法中执行任务（可以调用异步方法），根据任务结果更新状态。

可重写cancel方法，cancel方法中一定要更新isFinished=true，不然operation不会从OperationQueue的operations中删除，operation会一直不会结束
> Tip：异步方法isAsynchronous需要返回true

demo如下：

```
private class BaseOperation: Operation {
    enum State: String {
        case ready = "Ready"
        case executing = "Executing"
        case finished = "Finished"
        fileprivate var keyPath: String { return "is" + self.rawValue }
    }
    
    override var isAsynchronous: Bool {
        return true
    }
    override var isExecuting: Bool {
        return state == .executing
    }
    override var isFinished: Bool {
        return state == .finished
    }
    private var state = State.ready {
        willSet {
            willChangeValue(forKey: state.keyPath)
            willChangeValue(forKey: newValue.keyPath)
        }
        didSet {
            didChangeValue(forKey: state.keyPath)
            didChangeValue(forKey: oldValue.keyPath)
        }
    }
    
    override func start() {
        if isCancelled {
            state = .finished
        } else {
            state = .ready
            main()
        }
    }
    
    override func main() {
        state = self.isCancelled ? .finished : .executing
    }
    
    func finish(error: Error?) {
        state = .finished
    }
}
```

```
private class UploadOperation: BaseOperation {
    override func start() {
        super.start()
        /// 执行任务，更新状态  
    }
    override func cancel() {
        super.cancel()
        /// 这里一定要更新状态
        finish(error: nil)
    }
}
```
