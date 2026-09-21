A few days ago, [the first beta for Vapor 5 was announced](https://blog.vapor.codes/posts/vapor-5-beta/). If you don’t know what Vapor is, it’s probably the most popular web framework for Swift. I’ve [posted about it](https://blog.alexseifert.com/tag/vapor/) before.

Vapor 5 is the first new major version in six and a half years and promises a lot of new features, particularly when it comes to concurrency and other performance enhancements.

This is the list of highlights from their announcement blog post:

> -   No more `EventLoop` or `EventLoopFuture`! Vapor 5 is fully async/await with real structured concurrency support. We’ve even managed to almost completely remove NIO from the public APIs, with a couple of `ByteBuffer`s to remove from Multipart and some NIO work around compression and TLS to figure out.
> -   A brand new HTTP server. Vapor 5 builds upon [Swift HTTP Server](https://github.com/swift-server/swift-http-server), which is a new, modern HTTP server with support for bidirectional streaming, HTTP/3, and much more. Vapor can now concentrate on being a great web framework and hand off the HTTP handling to the new server.
> -   Streaming by default. Vapor 5 request and response bodies are now streaming-first, making it super easy to work with large files, backpressure, with great memory efficiency. We still have lots of syntactic sugar on top, so if you’re dealing with `Data` or `JSON`, you can easily handle those. Even better, the streaming closures use `Span`, making it as performant as possible.
> -   Service Lifecycle support. Vapor 5’s `Application` is just a `Service`, making it super easy to integrate with bigger applications. We also allow you to pass other services to your `Application` and let Vapor handle the lifecycle of those services as well.
> -   Swift Configuration support. We now use Swift Configuration to manage configuration of Vapor, making it easier to integrate with the rest of the Swift ecosystem.
> -   Swift HTTP Types. Our request and response types now expose `HTTPTypes` such as `Status` to better integrate with the ecosystem and make the HTTP server (and NIO) an implementation detail.
> -   A brand new, rewritten, Vapor Testing library that unifies the API with one way to test your application.
> -   Streaming support in the `Client` and `VaporTesting`.
> -   *All* of the SwiftPM flags! We have enabled almost every flag we could, including `InternalImportsByDefault`, `NonisolatedNonsendingByDefault` and `.strictMemorySafety()`. Not only does this ensure we are as safe as possible, but it sets us up for the future to handle Swift versions.
> -   New traits that make it easy to turn off certain features to shrink down the amount of Vapor you need to compile, use and distribute.
> -   A new, experimental macro library. We’ve built a new experimental macro library to help you define your routes and controllers. This brings type-safe routing and parameters, compiler checked authentication, and simpler route definitions! This will be a separate post and is still moving quickly.
> 
> [Vapor Blog](https://blog.vapor.codes/posts/vapor-5-beta/)

While I don’t have any active projects using Vapor, I’ve experimented with version 4 quite a bit and thoroughly enjoyed it. I’ll definitely be keeping an eye on version 5 and am excited for the stable release.

See the [official blog post](https://blog.vapor.codes/posts/vapor-5-beta/) announcing version 5 for more details about it.