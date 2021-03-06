---
layout: post
title: "The MacroOperation: How to Chain Operations"
date: 2018-05-20 14:27:00 -0700
categories: swift
---

When I'm developing new features for an application, I like to organize my use cases into [Operations](https://developer.apple.com/documentation/foundation/operation) (or `NSOperations`). I'll often divide one use case into a series of small steps and create a separate `Operation` for each unit of work. Creating use cases in this way helps with readability, both by splitting up work into focused reusable objects and by [screaming](https://8thlight.com/blog/uncle-bob/2011/09/30/Screaming-Architecture.html) the use cases of an application.

A recent problem I've been working on is how to cleanly chain operations together in order to fulfill a larger application use case. You may already know that [OperationQueue](https://developer.apple.com/documentation/foundation/operationqueue), out of the box, has functionality to order operations using dependencies. In other words, the `OperationQueue` will schedule as many concurrent operations as it can (as long as its below [maxConcurrentOperationsCount](https://developer.apple.com/documentation/foundation/operationqueue/1414982-maxconcurrentoperationcount)), but if `BarOp` depends on `FooOp` then `BarOp` will not begin until `FooOp` finishes execution.



Operation dependencies are perfect when you simply need to schedule one operation after another, but falls short when an operation depends on output generated by its depending operation. For example, imagine we have a use case for uploading an image to a server. We would have two operations: `ChoosePhoto` and `UploadPhoto`. `ChoosePhoto` presents UI for the user to select a photo on disk. `UploadPhoto` takes in a `URL` to a photo on disk and uploads it to a server. In this scenario, `UploadPhoto` runs after `ChoosePhoto` and depends on the output of `ChoosePhoto` (a `URL`) in order to execute. What are our options for passing data from one operation to another?

In some cases, the input and output of an operation are known at instantiation. If we are chaining a `DownloadFile` operation to an `OpenFile` operation, you can create the destination of the downloaded file first and then inject the location in the constructors for `DownloadFile` and `OpenFile`.

```swift
let downloadSrc = ...
let downloadDst = ...
let downloadOp = DownloadOperation(src: downloadSrc, dst: downloadDst)
let openOp = OpenOperation(src: downloadDst)
```

However, what if the output of an operation is not known at compile time? Consider again our use case for uploading a photo to a server. Since the location of the photo is not known at compile time, we are forced to instantiate the operations without specifying all operation dependencies. In other words, `ChoosePhoto's` output and `UploadPhoto's` input are both `nil` at compile time.

So how do we do this? I've personally used the following tactics for dynamically passing data from one operation to its dependent operation.

### Completion Block

```swift
let choosePhoto = ChoosePhotoOp()
choosePhoto.completionBlock = {

  let uploadPhoto = UploadPhotoOp(location: choosePhoto.location)
  queue.add(uploadPhoto)
}

queue.add(choosePhoto)
```

The first approach I took to chain operations is to create and queue dependent operations inside the [completionBlock](https://developer.apple.com/documentation/foundation/operation/1408085-completionblock). One benefit this provides is that you can instantiate operations with the output of the previous operation. However, I find this approach to a be a little on the hacky side because it ignores the operation dependencies infrastructure built into `OperationQueue` to serialize operations.

My biggest issue with using dependent operations and `OperationQueue` is a great solution to fix the "callback hell" problem. However, if you create and queue operations within other operations completion blocks, then we are reintroducing that problem all over again. Two operations may be fine, but as soon as the number of dependent operations climb above that, code readability starts to degrade.

### Adapter Operation

```swift
let choosePhoto = ChoosePhotoOp()
let uploadPhoto = UploadPhoto()
let adapterOp = BlockOperation { [weak uploadPhoto, weak choosePhoto] in
  uploadPhoto.location = choosePhoto.location
}

adapterOp.addDependency(choosePhoto)
uploadPhoto.addDependency(adapterOp)

queue.addOperations([choosePhoto, adapterOp, uploadPhoto], ...)
```

So if not completion blocks then what? One approach I found to be a lot cleaner was chaining operations using `BlockOperations` as adapters. I discovered this idea from [a post by iOS Coach Frank](http://ioscoachfrank.com/2017/03/18/chaining-nsoperations.html). The idea behind these operation adapters is create a block that transfers the output of one operation to the input of another operation. By scheduling these adapter operations to execute between dependent operations you can ensure that the data will be set before the next operation begins executing.

Adapter operations are a huge step above using completion blocks. You may need to refactor your operations a bit however since you have to account for your input and outputs being optional values. This doesn't cause much of an issue however, since you can just have a `guard` at the beginning of your `main` or `start` methods and return early if the values don't exist. Optional values better reflect the reality of runtime dependent operations anyways. If an error occurs in a previous operation, the output probably be `nil`?

### Adapter Closures

```swift
let choosePhoto = ChoosePhotoOp()
let uploadPhoto = UploadPhoto()
choosePhoto.onOutputSet = { [weak uploadPhoto] location in
  uploadPhoto.location = location
}

uploadPhoto.addDependency(choosePhoto)

queue.addOperations([choosePhoto, uploadPhoto], ...)
```

Along the same line as creating adapter operations is to set closures on operations to pass the data. One thing I did not like about adapter operations is that they cluttered the queue. What I mean is that instead of `UploadPhoto` being dependent on `ChoosePhoto`, `UploadPhoto ` is now dependent on an adapter operation, which is dependent on `ChoosePhoto`. It's a small issue, but I still thought it could be improved.

What I came up with may be called the "Swifty" approach -- using closures to pass data. Rather than schedule a `BlockOperation`, we can have operations call a closure when the output is set. Be careful not to confuse this closure with the completion block previously mentioned. The completion block of one operation can be called after its dependent operation has already begun which would cause the input data to be `nil`. Calling a closure when the output is set, guarantees the data is always passed before the dependent operation begins.

### The MacroOperation
A common design pattern in [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612) is the [Command Pattern](https://en.wikipedia.org/wiki/Command_pattern) which I think `Operation` is a prefect example. The book also mentions a "MacroCommand" pattern -- this idea that you can sequentially call multiple commands one after the other to fulfill one large Command. Sound familiar? By chaining operations together, we are essentially creating macro operations.

After creating a few macro operations using the adapter closure strategy, I started to notice a trend with each one.

1. Create operation
2. Create dependent operation
2. Set Closure
3. Set dependency
4. Create operation
5. Create dependent operation
6. Set Closure
7. Set dependency
8. ...

Anytime I see repeated code, I have to think there's a better solution and with Swift's use of generics, we are able to push the boilerplate code surrounding operation chaining into one `MacroOperation` that can handle all our use cases.

```
let uploadUserPhoto = MacroOperation(choosePhoto).then(uploadPhoto)
```

The key to the `MacroOperation` is to return an instance of `MacroOperation` with each call to `then`. If you've ever used [Promises](https://javascript.info/promise-chaining) in Javascript or [PromiseKit](https://github.com/mxcl/PromiseKit) or [FutureKit](https://github.com/FutureKit/FutureKit) in Swift, this may look familiar.

`MacroOperation` also deals with what I'll call an `IOOperation`, which is a subclass of `Operation` used for tasks that take in input and produce output. `IOOperation's` input and output types are both generic which means the compiler to enforces whether or not two `IOOperations` can be linked. `IOOperation` is also a perfect place to add the adapter closure and calling it whenever `output` is set.

### Conclusion
I find `MacroOperation` to be the most succinct solution because it removes the noisy setup code and focuses the reader on what is important -- the chained operations. Having said that, `MacroOperation` has still got room to grow and issues to sort out. There are a few issues that I have run into when using it for complicated macro operations.

With its current implementation, the MacroOperation can only chain dependent operations that are run in the same order as their dependencies. Imagine we have three operations: the first operation downloads some data, the second operation does some work on that data, and the third operation cleans up the downloaded data. Now both the second and third operation's inputs are dependent on the output of the first operation. However, the third operation cannot be run until the second operation is completed.

Another issue I see involves the idea of using [Progress](https://developer.apple.com/documentation/foundation/progress) inside operations -- something I really want to experiment with. I won't go into much detail, but as you've probably already guessed, `Progress` represents the current progress of an operation. One very powerful feature of `Progress` is creating child `Progress` objects. By default each takes up an equal portion of the total parent progress, but can also we weighed differently to say have one progress take of 50% of its parent and the other two to be 25%. If `MacroOperation` were to use `Progress`, then it could also pass its progress down to its child operations. However, currently there would be no way to specify that one child operation takes more time and thus should take up a larger portion of the `MacroOperation's` progress.

You can find the code for `MacroOperation` on my [GitHub](https://github.com/duffneubauer/MacroOperation). If you've got a solution to the one of the issues I mentioned I'd love to hear it! Feel free to comment below or create a pull request.
