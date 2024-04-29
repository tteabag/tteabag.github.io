---
title: Swift Concurrency
date: 2022-05-17 11:31:35
tags:
---

Swift Concurrency

# Async/await

## 一.怎样调用第一个async函数

情况1：

```
func processWeather() async {
    // Do async work here
}

@main
struct MainApp {
    static func main() async {
        await processWeather()
    }
}
```

<!-- more -->

情况2：SwiftUI相关modifiers，如refreshable() and task()

```
struct ContentView: View {
    @State private var sourceCode = ""

    var body: some View {
        ScrollView {
            Text(sourceCode)
        }
        .task {
            await fetchSource()
        }
    }

    func fetchSource() async {
        do {
            let url = URL(string: "https://apple.com")!

            let (data, _) = try await URLSession.shared.data(from: url)
            sourceCode = String(decoding: data, as: UTF8.self).trimmingCharacters(in: .whitespacesAndNewlines)
        } catch {
            sourceCode = "Failed to fetch apple.com"
        }
    }
}
```

情况三：使用Task api

```
Button("Go") {
    Task {
        await fetchSource()
    }
}
```

## 2.性能消耗

* 只要使用await，就会产生一个潜在的暂停点，swift无法判断在暂停点是否一定会暂停，固在await出一定会产生"calling convention"消耗；
* 暂停点如果没有暂停，则和synchronous方式一样，没得额外消耗，如果暂停，那一定是赚了

## 3.怎样创建使用异步属性（async properties）

异步属性只能用于**只读计算属性**

```
struct RemoteFile<T: Decodable> {
    let url: URL
    let type: T.Type

    // 定义
    var contents: T {
        get async throws {
            let (data, _) = try await URLSession.noCacheSession.data(from: url)
            return try JSONDecoder().decode(T.self, from: data)
        }
    }
}

// 使用
Task {
    do {
        messages = try await source.contents
    } catch {
        print("Message update failed.")
    }
}
```

## 4.async let的使用

async let，相当于DispathGroup，各个任务互相独立，同时执行

```
func loadData() async {
    async let (userData, _) = URLSession.shared.data(from: URL(string: "https://hws.dev/user-24601.json")!)
    async let (messageData, _) = URLSession.shared.data(from: URL(string: "https://hws.dev/user-messages.json")!)

    do {
        let decoder = JSONDecoder()
        let user = try await decoder.decode(User.self, from: userData)
        let messages = try await decoder.decode([Message].self, from: messageData)
        print("User \(user.name) has \(messages.count) message(s).")
    } catch {
        print("Sorry, there was a network problem.")
    }
}
```

## 5.将block转换成async

涉及函数`withCheckedContinuation `、`withUnsafeContinuation `、`withUnsafeThrowingContinuation `、`withCheckedThrowingContinuation `，
check函数会检查正确性，当多次resume，或者没有resume，都会导致crash

```
func fetchMessages() async -> [Message] {
    await withCheckedContinuation { continuation in
        fetchMessages { messages in
            continuation.resume(returning: messages)
        }
    }
}

func fetchMessages() async -> [Message] {
    do {
        return try await withCheckedThrowingContinuation { continuation in
            fetchMessages { messages in
                if messages.isEmpty {
                    continuation.resume(throwing: FetchError.noMessages)
                } else {
                    continuation.resume(returning: messages)
                }
            }
        }
    } catch {
        return [
            Message(id: 1, from: "Tom", message: "Welcome to MySpace! I'm your new friend.")
        ]
    }
}
```

# 二.Sequences and streams

```
/// for in
func fetchUsers() async throws {
    let url = URL(string: "https://hws.dev/users.csv")!

    for try await line in url.lines {
        print("Received user: \(line)")
    }
}
/// Iterator
func printUsers() async throws {
    let url = URL(string: "https://hws.dev/users.csv")!
    var iterator = url.lines.makeAsyncIterator()

    if let line = try await iterator.next() {
        print("The first user is \(line)")
    }
}
/// map\filiter\prefix...
func shoutQuotes() async throws {
    let url = URL(string: "https://hws.dev/quotes.txt")!
    let uppercaseLines = url.lines.map(\.localizedUppercase)

    for try await line in uppercaseLines {
        print(line)
    }
}
/// 返回不透明类型opaque return type
func getQuotes() async -> some AsyncSequence {
    let url = URL(string: "https://hws.dev/quotes.txt")!
    let anonymousQuotes = url.lines.filter { $0.contains("Anonymous") }
    let topAnonymousQuotes = anonymousQuotes.prefix(5)
    let shoutingTopAnonymousQuotes = topAnonymousQuotes.map(\.localizedUppercase)
    return shoutingTopAnonymousQuotes
}
/// convert an AsyncSequence into a Sequence
extension AsyncSequence {
    func collect() async rethrows -> [Element] {
        try await reduce(into: [Element]()) { $0.append($1) }
    }
}
```

# 三、Task\TaskGroup

## 1.创建Task

```
func fetchUpdates() async {
    let newsTask = Task { () -> [NewsItem] in
        let url = URL(string: "https://hws.dev/headlines.json")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode([NewsItem].self, from: data)
    }

    do {
        let highScores = try await highScoreTask.value
        print("Latest news loaded with \(news.count) items.")
        if let topScore = highScores.first {
        }
    } catch {
    }
}
```

## 2.Task.init和Task.detached的区别

## 3.获取Task的结果result

```
func fetchQuotes() async {
    let downloadTask = Task { () -> String in
        let url = URL(string: "https://hws.dev/quotes.txt")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return String(decoding: data, as: UTF8.self)
    }
    let result = await downloadTask.result
    do {
        let string = try result.get()
    } catch {
    }
}
```

## 4.取消任务

检查任务：`Task.checkCancellation()`和`Task.isCancelled`

取消任务：`task.cancel()`

## 5.Task睡眠

task sleep不会导致thread block
`try await Task.sleep(nanoseconds: 3_000_000_000)`

## 6.自愿暂停任务

自愿暂停任务，可以让其他任务执行机会，但不一定会暂停（优先级最高）
`await Task.yield()`

## 7.创建TaskGroup

两个方法`withTaskGroup `和`withThrowingTaskGroup `，另外TaskGroup遵循`AsyncSequence `协议，TaskGroup只能返回同一Type数据

cancel TaskGroup分三种情况：

* 父Task被取消
* 显示的调用cancelAll()
* 其中一个子任务throws未捕获的错误

```
func loadStories() async {
    do {
        stories = try await withThrowingTaskGroup(of: [NewsStory].self) { group -> [NewsStory] in
            for i in 1...5 {
                group.addTask {
                    let url = URL(string: "https://hws.dev/news-\(i).json")!
                    let (data, _) = try await URLSession.shared.data(from: url)
                    return try JSONDecoder().decode([NewsStory].self, from: data)
                }
            }
            let allStories = try await group.reduce(into: [NewsStory]()) { $0 += $1 }
            return allStories.sorted { $0.id > $1.id }
        }
    } catch {
    }
}
```

## 8.处理TaskGroup返回不同数据类型

两种方式：

* 使用`async let`
* 使用enum+关联类型

## 9.TaskLocal

```
enum User {
    @TaskLocal static var id = "Anonymous"
}

@main
struct App {
    static func main() async throws {
        Task {
            try await User.$id.withValue("Piper") {
                print("Start of task: \(User.id)")
                try await Task.sleep(nanoseconds: 1_000_000)
                print("End of task: \(User.id)")
            }
        }

        Task {
            try await User.$id.withValue("Alex") {
                print("Start of task: \(User.id)")
                try await Task.sleep(nanoseconds: 1_000_000)
                print("End of task: \(User.id)")
            }
        }

        print("Outside of tasks: \(User.id)")
    }
}
```

## 10.task modifier

> task modifier 在onAppear调用，在onDisappear时cancel，为了避免反复调用，使用task(id:)

# 四.Actor

actior特点如下：

* actor关键词同struct、class同级
* actor和class一致，都是引用类型，但actor不能继承，也就不能使用final、override关键词。
* actor自动继承Actor协议

> actor是线程安全的，外部访问actor的mutable state需要使用await

> 使用isolated关键词，不会要求使用await

```
actor DataStore {
    var username = "Anonymous"
}

func debugLog(dataStore: isolated DataStore) {
    print("Username: \(dataStore.username)")
}
```

> * actor 内部使用`nonisolated `，方法和计算属性可以使用
> * 访问使用`nonisolated `的方法属性不需要加`await`
> * nonisolated方法内部只能访问nonisolated方法不需要加await
> * Codable等协议方法需要使用上`nonisolated `关键词

```
actor User {
    let username: String
    let password: String
    var isOnline = false

    init(username: String, password: String) {
        self.username = username
        self.password = password
    }

    nonisolated func passwordHash() -> String {
        return password
    }
}

let user = User(username: "twostraws", password: "s3kr1t")
print(user.passwordHash())
```

## 1.MainActor

MainActor是一个全局的actor，保证所以的work在主线程执行。
被@MainActor标记的方法、类型可以安全的操作UI
MainActor.run可以将任务放在主线程执行

demo：

```
@MainActor
class AccountViewModel: ObservableObject {
    @Published var username = "Anonymous"
    @Published var isAuthenticated = false
}

await MainActor.run {
    print("This is on the main actor.")
}

Task {
    await MainActor.run {
        print("This is on the main actor.")
    }
}

Task { @MainActor in
    print("This is on the main actor.")
}
```
