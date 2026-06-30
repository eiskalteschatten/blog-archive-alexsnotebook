<figure><img loading="lazy" decoding="async" src="Gemini_Generated_Image_unxjd1unxjd1unxj.png" alt="AI-generated image of a man presenting &quot;Web App Using SwiftNIO&quot; to an audience"><figcaption>AI-generated image of a man presenting “Web App Using SwiftNIO” to an audience</figcaption></figure>

Recently, I’ve been experimenting a lot more with Swift as a language for web applications. I’ve primarily been focusing my efforts on the web framework [Vapor](https://www.vapor.codes) which is the most complete and popular web framework for Swift and is used by the likes of some very large projects such as the [Swift Package Index](https://swiftpackageindex.com).

The framework is based on [SwiftNIO](https://github.com/apple/swift-nio) which is Apple’s open-source “cross-platform asynchronous event-driven network application framework for rapid development of maintainable high performance protocol servers & clients.” I also recently read somewhere that Apple itself uses SwiftNIO to power its newer web services as it is low-level and low-latency — both critical aspects considering the sheer enormity of the number of requests that Apple’s web services have to process. Whether or not that is true is something I can’t confirm, but it sounds plausible considering it is an Apple-developed framework that does exactly what a high-performance web service would need.

So, naturally, I asked myself what it would take to make my own basic web app using SwiftNIO without any other dependencies beyond the three others that SwiftNIO itself relies on. As it turns out, it doesn’t actually take as much as I thought. Since I wanted to keep this experiment simple, I focused on basic functionality rather than implementing a fully featured web framework. If I wanted that, I would just go with Vapor.

Before we start, I want to mention that I pushed the entire experiment to [GitHub](https://github.com/eiskalteschatten/swift-webapp-frameworkless) with thoroughly commented code so that you can see what I did and/or play around with it yourself. There will be several links to various parts of the code on GitHub through the post.

**Setting Up the Server**
-------------------------

The first, and arguably most critical, feature any web application needs is a server. In this case, one that uses SwiftNIO for network communication. In order to accomplish that, I created two classes: [HTTPServer](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/55c5c2cff25867d5610b187556beb48410a42f06/Sources/HTTPServer.swift#L166) and [HTTPHandler](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/55c5c2cff25867d5610b187556beb48410a42f06/Sources/HTTPServer.swift#L28). As the names might suggest, the first sets up the server and network infrastructure while the latter handles each individual connection.

The `HTTPServer` class includes a method called `start()` which uses SwiftNIO’s [ServerBootstrap](https://swiftpackageindex.com/apple/swift-nio/main/documentation/nioposix/serverbootstrap) to bind a TCP port and configure how connections are handled:

```swift
public func start() async throws {
    let router = self.router

    let bootstrap = ServerBootstrap(group: group)
        // How many pending connections the OS kernel should queue before the
        // application has a chance to accept them.
        .serverChannelOption(ChannelOptions.backlog, value: 256)
        // Allow the port to be reused immediately after the process restarts,
        // without waiting for the OS TIME_WAIT period to expire.
        .serverChannelOption(ChannelOptions.socketOption(.so_reuseaddr), value: 1)
        // For each accepted connection, configure the NIO pipeline:
        //   1. Built-in HTTP/1.1 encoder + decoder
        //   2. Our custom HTTPHandler for routing
        .childChannelInitializer { channel in
            channel.pipeline.configureHTTPServerPipeline().flatMap {
                channel.pipeline.addHandler(HTTPHandler(router: router))
            }
        }

    let channel = try await bootstrap.bind(host: host, port: port).get()
    print("Server running on http://\(host):\(port)")

    // Suspend here until the channel is closed. In normal operation this never
    // returns — the server runs until the process is killed.
    try await channel.closeFuture.get()
}
```

That’s all that the `HTTPServer` class actually does. The rest happens in the `HTTPHandler` class which, as you may have noticed is added as a handler to the channel pipeline.

For each incoming request, a new `HTTPHandler` instance is created and is responsible for four different tasks:

1.  Receiving request parts
2.  Bridging to Swift concurrency via Task
3.  Writing the response
4.  Closing the connection

This is the lifecycle of a single request:

```swift
Client connects
        ↓
ServerBootstrap accepts → creates HTTPHandler
        ↓
SwiftNIO fires channelRead(.head)  → store head
SwiftNIO fires channelRead(.body)  → accumulate bytes  (0 or more times)
SwiftNIO fires channelRead(.end)   → dispatch to router via Task
        ↓
Router matches route → calls handler → returns HTTPResponse
        ↓
eventLoop.execute → write response head + body + end
        ↓
Connection closed
```

### Receiving Request Parts

The `HTTPHandler` class implements SwiftNIO’s `ChannelInboundHandler` protocol which expects, among other things, the function `channelRead()`. This function is called multiple times per request and a value identifying which part of the request is currently being handled is passed via one of three different enums: `.head`, `.body` and `.end`. With these, we can determine our course of action and how to handle the data being sent.

One gotcha, however, is that SwiftNIO breaks down large bodies into chunks which means there is the potential for multiple `.body` parts. I go into more detail about that in the section below where I describe how I implemented the RESTful methods.

### Bridging to Swift Concurrency via Task

SwiftNIO operates with event loops which, like when working with Node.js, should never be blocked. As such, we need to ensure that route handlers are executed asynchronously. I implemented that by stuffing them inside a Swift `Task` when SwiftNIO passes `.end`:

```swift
Task { [head = head] in
    let response = await router.handle(head: head, body: body)
    // ↑ runs on Swift's cooperative thread pool, not the NIO thread
    
    eventLoop.execute {
        self.write(response: response, context: context)
        // ↑ hop back onto the NIO event loop to write the response
    }
}
```

### Writing The Response

The `HTTPHandler` class is also responsible for serializing the `HTTPResponse` object returned by the routes (more on that below) into SwiftNIO’s three-part format which includes:

-   HTTPResponseHead (status & headers)
-   ByteBuffer (body bytes)
-   End marker (signals that the response is complete)

This logic takes place in the `write()` function ([see on GitHub](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/55c5c2cff25867d5610b187556beb48410a42f06/Sources/HTTPServer.swift#L127)):

```swift
private func write(response: HTTPResponse, context: ChannelHandlerContext) {
    // Allocate a NIO `ByteBuffer` sized to the response body and copy the bytes in.
    var buffer = context.channel.allocator.buffer(capacity: response.bodyData.count)
    buffer.writeBytes(response.bodyData)

    var headers = HTTPHeaders()
    headers.add(name: "Content-Type", value: response.contentType)
    // Telling the client the exact byte count avoids chunked transfer encoding
    // and lets browsers display progress correctly.
    headers.add(name: "Content-Length", value: "\(buffer.readableBytes)")
    headers.add(name: "Connection", value: "close")

    let responseHead = HTTPResponseHead(version: .http1_1, status: response.status, headers: headers)

    // Write all three parts. The first two use `write` (buffered); the last uses
    // `writeAndFlush` to push everything to the network in one syscall.
    context.write(wrapOutboundOut(.head(responseHead)), promise: nil)
    context.write(wrapOutboundOut(.body(.byteBuffer(buffer))), promise: nil)

    let channel = context.channel
    context.writeAndFlush(wrapOutboundOut(.end(nil))).whenComplete { _ in
        // Close the TCP connection once all bytes have been sent.
        channel.close(promise: nil)
    }
}
```

`HTTPResponse` is a `Sendable` struct I created that contains the response HTTP status, the content type and the body data as a Swift `Data` object. You can view it on [GitHub](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/55c5c2cff25867d5610b187556beb48410a42f06/Sources/Router.swift#L16).

### Closing The Connection

The `write()` function above also contains the code that closes the connection after every response. This is the bit of code responsible for doing that:

```swift
let channel = context.channel
context.writeAndFlush(wrapOutboundOut(.end(nil))).whenComplete { _ in
    // Close the TCP connection once all bytes have been sent.
    channel.close(promise: nil)
}
```

### Starting the Server

Now that we have everything in place, all that is left is to instantiate the `HTTPServer` class and call the `start()` function at the end of the [main.swift](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/main/Sources/main.swift) file to start the server:

```swift
let server = HTTPServer(host: "127.0.0.1", port: 8080, router: router)
try await server.start()
```

Now that we’ve looked at how the custom server works, let’s see what else the application can do using the server.

What This Basic Web App Can Do
------------------------------

I’ve mentioned several times already that the web app I made is basic and simple. It does, however, contain everything necessary for a basic API application or even a web application with light templating needs. Here is an overview:

-   RESTful methods
-   Routing
-   Basic templating with XSS safety
-   Static content
-   HTTP/1.1 support

### RESTful Methods

First off, the application includes support for routes with all RESTful methods (GET, POST, PUT, PATCH, DELETE) including body and header parsing. Initially, I wasn’t going to include support for anything but GET, but SwiftNIO made it surprisingly trivial to include all of them.

Supporting body parsing was a bit trickier, but also wasn’t all that difficult once I figured out how SwiftNIO actually handles requests. It doesn’t actually ever give you a complete HTTP request in one go. Instead, it streams it piece by piece. These can be identified via enums like this: `.head  →  .body  →  .body  →  ...  →  .end`. As you can see though, the `.body` case can arrive multiple times for large payloads since SwiftNIO breaks it down into chunks. That made it a bit trickier to solve, but also not terrible. I just had to accumulate the chunks and then put them together at the end of the request:

```swift
private var bodyBuffer: ByteBuffer?

// ...

case .body(var buffer):
    // Accumulate body chunks. For small bodies this will be a single call;
    // for larger bodies NIO may deliver multiple chunks.
    if bodyBuffer == nil {
        bodyBuffer = buffer
    } else {
        // Append the new chunk to the existing buffer.
        bodyBuffer!.writeBuffer(&buffer)
    }

case .end:
    if var buf = bodyBuffer, let bytes = buf.readBytes(length: buf.readableBytes) {
        body = Data(bytes)   // ByteBuffer → [UInt8] → Data
    } else {
        body = Data()        // no body (GET, HEAD, etc.)
    }
```

The code above is heavily cherry-picked. You can see the whole context on [GitHub](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/55c5c2cff25867d5610b187556beb48410a42f06/Sources/HTTPServer.swift#L71).

### Routing

Of course, no web application would be worth anything if there wasn’t some form of routing. To accomplish this, I created a `Router` class that is responsible for registering all routes with their respective methods and handler functions, parsing the incoming URLs and calling said handler functions. The latter always return an `HTTPResponse` object as described above. I figured the best way to do this would be the regex-route (pun intended) which can be seen in the `add()` function of the `Router` class:

```swift
public func add(_ method: HTTPMethod, _ path: String, _ handler: @escaping Handler) {
    var paramNames: [String] = []

    // Replace each `:paramName` segment with a regex capture group `([^/]+)`,
    // and record the parameter name so we can map captures back to names later.
    let regexString = path.replacing(/:([a-zA-Z0-9_]+)/) { match in
        paramNames.append(String(match.output.1))
        return "([^/]+)"
    }

    guard let regex = try? Regex("^" + regexString + "$") else { return }
    lock.withLock { routes.append(Route(method: method, regex: regex, paramNames: paramNames, handler: handler)) }
} 
```

Then all you have to do is add your routes and handlers in the [main.swift](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/main/Sources/main.swift) file:

```swift
router.add(.GET, "/") { head, _, _, _ in
    let pageHtml = layoutView(title: "Homepage", content: homeView())
    return HTTPResponse(status: .ok, contentType: "text/html", body: pageHtml.description)
}

router.add(.GET, "/hello/:name") { head, params, urlComponents, _ in
    let name = params["name"] ?? "unknown"

    // Extract the optional `?german=true` query parameter.
    let germanParam = urlComponents.queryItems?.first(where: { $0.name == "german" })?.value
    let isGerman = germanParam == "true"

    let title = isGerman ? "Hallo \(name)!" : "Hello \(name)!"
    let pageHtml = layoutView(title: title, content: helloView(name: name, isGerman: isGerman), name: name)

    return HTTPResponse(status: .ok, contentType: "text/html", body: pageHtml.description)
}
```

You might have noticed in the example that parameterization of URLs is also supported. This includes both inline parameters (i.e. `/hello/:name`) as well as query parameters (i.e. `/hello/:name?german=true`). Inline parameters are parsed using the regex above and query parameters are extracted from the `urlComponents` variable injected into the handler function.

You can find the entire `Router` class on [GitHub](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/main/Sources/Router.swift).

### Basic Templating with XSS Safety

There were a number of approaches I considered when it came to adding templating to the project. My first instinct was, of course, to reach for one of the off-the-shelf solutions such as [Vapor’s Leaf](https://docs.vapor.codes/leaf/overview/), but I wanted to avoid extra dependencies and keep it as simple as possible. As such, I just decided on simple string interpolation where each template is a standard Swift function that returns a string of HTML. That may not be the best approach for a large web application with a complex frontend, but works perfectly well for this experiment.

Fortunately, that also means it’s fairly straightforward. There is a layout view that provides the structure for the site as well as a specific view for each route that should return HTML.

The layout view:

```swift
func layoutView(title: String, content: HTML, name: String = "") -> HTML {
    return """
    <!DOCTYPE html>
    <html>
        <head><title>\(title)</title></head>
        <link rel="stylesheet" href="/css/main.css">
        <script src="/js/scripts.js"></script>
        <body>
            <nav>
                <a href="/">Home</a>
            </nav>
            <main>\(content)</main>
            <p>Input your name: <input type="text" id="name" value="\(name)"></p>
            <div class="buttons">
                <button type="button" id="sayHelloButton">Say Hello!</button>
                <button type="button" id="sayHelloGermanButton">Say Hello in German!</button>
            </div>
        </body>
    </html>
    """
}
```

The `/hello/:name` route’s view:

```swift
func helloView(name: String, isGerman: Bool = false) -> HTML {
    let greeting = isGerman ? "Hallo \(name)!" : "Hello \(name)!"
    return "<h1>\(greeting)</h1>"
}
```

The layout and route-specific views get put together in each route’s handler. If you look back at the previous examples of adding routes, you’ll see the `layoutView()` function takes a content parameter. This is where the route-specific view is injected into the layout.

You might have also noticed the mysterious `HTML` type. This doesn’t come from SwiftNIO, but instead is a custom interpolation struct that provides XSS (cross-site scripting) safety. While a little excessive for this particular basic project, I happened to find out while making it that Swift supports such a thing and decided to give it a shot. I won’t go into detail about it here, but you can view the commented struct on [GitHub](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/main/Sources/HTML.swift).

### Static Content

If you have templates, you have to have static content as well. What modern website doesn’t have CSS and JavaScript? Even my basic example does, but more as a proof of concept than anything else.

Static files are located in the `Public` folder at the root of the project. The application will automatically figure out the path to it relative to the executable and then return the requested file via a wildcard GET route added to the router after all of the user-defined routes have been added. If no file is found, a 404 “not found” error is returned.

This was surprisingly one of the trickier parts of the whole experiment to figure out, but once I had it, it wasn’t terrible to implement. Essentially, I had to read the static file as a Swift `Data` object and pass it along with the `mimeType` to the `HTTPResponse` to be handled in the same way it handles a standard string response:

```swift
guard let data = try? Data(contentsOf: URL(fileURLWithPath: fullPath)) else {
    return HTTPResponse(status: .notFound, body: "Not Found")
}

let ext = (fullPath as NSString).pathExtension
return HTTPResponse(status: .ok, contentType: mimeType(for: ext), data: data)
```

There is a little bit of magic that also happens in there in the form of path sanitization so that a user can’t use standard Unix/Linux paths like “/..” to reach other files in the OS:

```swift
let sanitized = filePath
    .components(separatedBy: "/")
    .filter { !$0.isEmpty && $0 != ".." }
    .joined(separator: "/")

guard !sanitized.isEmpty else {
    return HTTPResponse(status: .notFound, body: "Not Found")
}
```

You can see the wildcard route and how the files are read on [GitHub](https://github.com/eiskalteschatten/swift-webapp-frameworkless/blob/55c5c2cff25867d5610b187556beb48410a42f06/Sources/Router.swift#L121).

### Only HTTP/1.1 is Supported

One notable limitation of the web application in its current form is that it only supports HTTP/1.1. SwiftNIO also has additional Apple-developed libraries that extend it to support HTTP/2 ([swift-nio-http2](https://github.com/apple/swift-nio-http2)) and even an experimental library to allow it to support HTTP/3 ([swift-nio-http3](https://github.com/apple/swift-nio-http3)), but I figured sticking with HTTP/1.1 would be good enough for this small experiment.

What Else It Could Have Done
----------------------------

I enjoyed creating this project enough that it was hard to keep it simple. I could have just kept tacking on feature after feature until it did everything a standard web application can do, but I was (fairly) disciplined and kept it basic. These are some of the ways I thought it might be nice to expand it and, perhaps, I will fire up a new repo one day to do so.

As mentioned above, the server currently only supports HTTP/1.1. It would probably be fairly trivial to add at least support for HTTP/2 with [swift-nio-http2](https://github.com/apple/swift-nio-http2), but I haven’t attempted that yet.

Also, a proper templating engine might be useful for a real application. I’ve worked with [Vapor’s Leaf](https://docs.vapor.codes/leaf/overview/), but have also dabbled with [Stencil](https://github.com/stencilproject/Stencil) which is another template language for Swift. Either one would be a good choice.

Then, of course, rare is the web app that doesn’t have a database behind it. The easiest solution would be to statically link to the operating system’s C SQLite library and call it using raw queries directly in your Swift code. For other databases such as MySQL and PostgreSQL, the makers of Vapor also provide NIO-libraries: [MySQLNIO](https://github.com/vapor/mysql-nio) and [PostgresNIO](https://github.com/vapor/postgres-nio). If you would prefer a full-fledged ORM, then [Vapor’s Fluent](https://docs.vapor.codes/fluent/overview/) is probably the way to go.

However, if you’ve already got Leaf and Fluent, then you might as well save yourself the trouble of everything else and just use [Vapor](https://www.vapor.codes) itself which, as mentioned above, is also based on SwiftNIO.

Conclusion
----------

So, would I ever use this on an actual production system? Honestly, probably not. For the vast majority of projects, a complete web framework like Vapor makes a lot more sense because it makes development so much faster and easier with much less code to maintain yourself. The performance penalty is so negligible that you probably wouldn’t be able to measure the difference in anything but nanoseconds.

However, if you are serving requests at the same scale as Apple, then every nanosecond counts and you might want to consider going with raw SwiftNIO. The margin for error is wide, but the ability to finely tweak performance is vast. Since no project I have ever worked on — professionally or personally — has ever even come close to receiving the amount of requests an Apple service does, I honestly would always go with a framework like Vapor.

That said, I did enjoy the experiment and it was interesting to piece together how it works behind the scenes. I could have easily just kept bolting on more features until I had my own mini framework, but I decided to just keep it simple and focus on the core of the application. After all, the point of the experiment was to figure out how it worked and if I could even make it work at all. Fortunately, I was successful in both endeavors.

And just as a small aside: The README file in the [GitHub repository](https://github.com/eiskalteschatten/swift-webapp-frameworkless) has a few more technical details than I’ve included here. There are also instructions on how to get it to run and build on Mac (with and without Xcode) as well as Linux. It will only work on Windows with WSL since SwiftNIO relies on POSIX calls for its thread-pooling.

Links
-----

-   [GitHub repository with my experiment](https://github.com/eiskalteschatten/swift-webapp-frameworkless)
-   [SwiftNIO](https://github.com/apple/swift-nio)
-   [Vapor](https://www.vapor.codes)