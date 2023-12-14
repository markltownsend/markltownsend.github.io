---
layout: post
title: Migrating to the @Observable macro from ObservableObject
date: 2023-12-14
category: swift
author: Mark Townsend
tags: [swift, swiftui]
summary: 
---

Starting with iOS 17, we were introduced to the `@Observable` macro which is meant to replace the `ObservableObject` protocol.

The advantage is that the `@Observable` macro removes a lot of the boiler plate of adding `@Published` to properties you may be interested in observing changes for in your SwiftUI View. Instead, by default all properties are observable in a SwiftUI View using the `@Observable` macro.

I recently migrated over to this with one of my side projects. This is one of the projects that I use to be able to keep up with the latest and greatest when it comes to the Apple platform, in general.

### Migrating
The basics are migrating from this:

```swift
import Foundation

public final class MyDataViewModel: ObservableObject {
	@Published var data: [String] = []
}
```

to `@Observable`

```swift
import Foundation
import Observation

@Observable
public final class MyDataViewModel {
	var data: [String] = []
}
```

The `data` property automatically becomes observable in your SwiftUI views. If you don't want it to be observable add the `@ObservationIgnored` macro.

```swift
@ObservationIgnored var data: [String] = []
```

The name of the macro is pretty straight forward. This allows the flexibility of opting out instead of opting in.

`@Observable` is a pretty powerful macro that does a lot of things under the hood. It's a great opportunity to make use of the `Expand Macro` menu to see exactly what is happening.

This is what it looks like using the short example code above.

![ObservationMacroExpanded](/assets/images/ObservableMacroExpansion.png)

As you can see, it adds quite a lot of code. It's a good idea to understand what is happening here.

**Between lines 12-13**: a property wrapper is added `@ObservationTracked` that tells the system that this property should notify clients when it's changed.

Starting from the `@ObservationIgnored` line is where all the magic of this macro begins. First it registers a private property of the struct `Observation.ObservationRegistrar`. This is a struct defined in the `Observation` module that "Provides storage for tracking and access to data changes". The documentation mentions that it's not necessary to create one of these directly if using the `@Observable` macro.  We can see that here.

The next two _internal_ methods are used for reading and writing each property by registering it with the `ObservationRegistrar` instance.  This is the gist of the boiler plate code that the macro provides for the developer so you don't have to implement it every time. A perfect use of macros.

### Observing properties outside of SwiftUI

Sometimes you may have to either have a support object that is a dependent of the `@Observable` instance or in the same class itself where it is necessary to respond to changes to the property outside of a SwiftUI View.  This is where Combine worked great. But because we are no longer using Combine's `@Publisher` property wrapper, Combine can't be used.  Instead, the `Observation` framework provides us with a function

```swift
func withObservationTracking<T>(_ apply: () -> T, onChange: @autoclosure () -> () -> Void) -> T
```

The documentation isn't super clear on the best way to use this function.  There have been a couple of blog posts in the last few weeks that have provided different approaches. See below for references to this different methods. By default this function will only be called once, so it is necessary to somehow keep calling it if you want to continue to monitor changes.

The simple implementation can be the following:

```swift
func startObserving() {
	withObservationTracking {
		Task { await doSomething(with data: self.data) }
	} onChange: {
		self.startObserving()
	}
}
```
I'm sure over time developers will figure out creative ways to make good use of this function. 

### Conclusion
There are lots of good improvements to how SwiftUI Views can monitor changes in properties in supporting objects with a simpler API. Check it out and let's see how we can continue to make our code bases simpler and easier to understand.

#### References
* [Migrating from the Observable Object protocol to the Observable macro](https://developer.apple.com/documentation/swiftui/migrating-from-the-observable-object-protocol-to-the-observable-macro) - Apple
* [Using Observation framework outside of SwiftUI
](https://nilcoalescing.com/blog/ObservationFrameworkOutsideOfSwiftUI/) - Natalia Panferova
* [Create an AsyncStream from withObservationTracking() function](https://nilcoalescing.com/blog/AsyncStreamFromWithObservationTrackingFunc/) - Matthaus Woolard