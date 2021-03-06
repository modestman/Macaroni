# Macaroni
Swift Dependency Injection Framework "Macaroni".

#### Main reason to exist

When I start my projects, I need some kind of DI. When [property wrappers](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md) were introduced, it was obvious that this feature can be used for DI framework. So here it is.

There is a serious limitation in current property wrappers. They can't access enclosing `self` in a wrapper type (this is noted as future direction in the proposal). This is why we need a parameter that shows to a wrapper what container to use for the injection. Other than that it is a very simple library.

# Simple example

First let's import Macaroni and prepare our protocol (We'll use it in all examples)

```swift
import Macaroni

protocol MyService {}
class MyServiceImplementation: MyService {}
```

Now we can create a container factory. Container is a box that holds all injected objects. Container can be created once (this is what SingletonContainer for), or you can implement your own logic.

During factory creation we create all the objects. Then register them in the container. 

```swift
class MyContainerFactory: SingletonContainerFactory {
    override func build() -> Container {
        let container = SimpleContainer()
        let myService = MyServiceImplementation()
        container.register { () -> MyService in myService }
        return container
    }
}
```

Now we have to create Scope. It is a container factory holder, that will create container if needed and use it to inject from it.

```swift
let myScope = Scope(factory: MyContainerFactory())
```

And, finally, we can inject things!

```swift
class MyController {
    @Injected(from: myScope)
    var myService: MyService
}
``` 

# More simple and tricky things

If you have a single container in your application, you can simplify this. There is the default place, where Macaroni looks for the Scope. It is called `Scope.default`. If you save your scope there, you can omit scope parameter in `@Injected` like this. 

```swift
Scope.default = myScope

class MyController {
    @Injected
    var myService: MyService
}
```

You can also create several scopes and use them where needed.

```swift
extension Scope {
    static let application = Scope(factory: ApplicationContainerFactory())
    static let flow = Scope(factory: FlowContainerFactory())
}

class ApplicationLevelObject {
    @Injected(from: .application)
    var myService: MyService
}

class FlowLevelObject {
    @Injected(from: .flow)
    var myService: MyService
}
```

Please note that all these properties are read-only.

If you need to use enclosing type information during object injection, you can use another `@Injected` option:

```swift
class MyController {
    @Injected({ createMyObjectFromContainer($0) })
    var myObject: MyObject
}
```

If you need to use enclosing instance there, only thing you can do is lazy initialization. It is called `@InjectedWithSetup`. And at some point in the future you have to call setup for the property.

```swift
class MyController {
    @InjectedWithSetup
    var myObject: MyObject

    init() {
        ...
        // we have to use underscore to call wrapper, not value itself
        _myObject.setup { container in
            createMyObjectFromContanerUsingSelf($0) 
        }
    }
}
``` 
