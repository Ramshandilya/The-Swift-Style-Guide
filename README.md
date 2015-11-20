# The-Swift-Style-Guide
A guide to Swift coing style and guidelines.
## Objective
Software (a.k.a. source code) has a number of interesting characteristics -
* It is written once but read many times.
* It is harder to read than it is to write.
* The main consumer is not the compiler but people.
* Well written software not only satisfies the compiler (i.e. no errors or warnings) and correctly implements the desired feature set, but it’s also easy to consume in a relatively short period by a software engineer with the proper background.

The objective of the this guide is to encourage patterns that accomplish the following goals:
* Increase rigor, and decreased likelihood of programmer error.
* Increase clarity of intent.
* Reduce verbosity.
* Maintainability. Robustness.

Where the guide is silent, default to [Apple's coding guidelines for Cocoa](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html).

## Table of Contents
* [Naming](#naming)
* [Code Organization](#code-crganization)
* [Comments](#comments)
* [Mutability-Immutability](#mutability-immutability)
* [Types](#types)
* [Optionals](#optionals)
* [Closures](#closures)
* [Classes and Structures](#classes-and-structures)
* [Singletons](#singletons)
* [Semicolons](#semicolons)
* [Miscellaneous](#miscellaneous)

----
## Naming
Names should be meaningful and compact, written in camelCase. Try to ask yourself whether the name of a type sufficiently explains its behavior. Meaningful naming is very important to other developers because they define some expectations about their own roles.
* **Always** use meaningful english names that are self-descriptive.
* **Always** start class names, typedefs and enums type with an uppercase letter.
* **Always** start names of variables, functions, and methods with a lowercase letter.
* Abbreviations should be **avoided**, except where they are widely accepted *(e.g. fileURL)*
* **Never** create similar names. In particular, do not create names that differ only by case.
* **Never** use identifiers that begin with an underscore; these are reserved by Apple.
* **Never** use single or simple character names (e.g., "j", "iii") except for local loop and array indices.
* As far as possible, try not to name your classes ending with words like `Manager`, `Helper` or `Utility` because their meanings are very generic and their role can be easily misinterpreted. We tend to dump a lot of code to such classes.

#### Functions & Arguments
For functions and init methods, prefer named parameters for all arguments unless the context is very clear. Include external parameter names if it makes function calls more readable.
```swift
func dateFromString(dateString: String) -> NSDate
func convertPointAt(column column: Int, row: Int) -> CGPoint
func timedAction(afterDelay delay: NSTimeInterval, perform action: SKAction) -> SKAction!

// would be called like this:
dateFromString("2014-03-14")
convertPointAt(column: 42, row: 13)
timedAction(afterDelay: 1.0, perform: someOtherAction)
```

#### Class Prefixes
Swift types are automatically namespaced by the module that contains them and you need not add a class prefix. 

## Code Organization
Source files should have the following kind of organization.

```swift
import StarWarsKit

class JediTemple {

    //MARK: Public properties
    let jediKnights: [Jedi]
    private(set) var force: Force
    
    //MARK: Private properties
    private let aliveKnights: [Jedi]
    
    //MARK: Init
    init(jedi: [Jedi], force: Force)
    
    //MARK: Public methods
    func calcTotalForce()
    
    //MARK: Private methods
    private func councilMembers() -> [Jedi]
}

//MARK: SomeProtocol methods
extension JediTemple: Printable {
    var dialogue: String {
        return "Fight, together we must!"
    }
}
```

## Comments
In general, code should be self-documenting as much as possible. However, there are behaviors that cannot be statically expressed within code and can only be determined at runtime. Under these circumstances, comments should be provided to explain _why_ a particular piece of code does something. All comments must be kept up-to-date or deleted.

## Mutability-Immutability
Constants are defined using the `let` keyword, and variables with the `var` keyword. Always use `let` instead of `var` if the value of the variable will not change.

* A `let`-binding guarantees and clearly signals to the programmer that its value will never change. Subsequent code can thus make stronger assumptions about its usage.

* It becomes easier to reason about code. Had you used `var` while still making the assumption that the value never changed, you would have to manually check that.

* A good technique is to define everything using `let` and only change it to `var` if the compiler complains.
* `let` also allows the compiler to make optimizations.

## Types
Use native Swift types before you come up with your own. Every type can be extended, so sometimes instead of introducing new types, it's convenient to extend or alias existing ones.

Remember that Objective-C classes that have native Swift equivalents are not automatically bridged, e.g. `NSString` is not implicitly bridged to `String` in the following example.

```swift
func lowercase(string String) -> String

let string: NSString = /* ... */

lowercase(string) // compile error
lowercase(string as String) // no error
```

Types should be inferred whenever possible. Don't duplicate type identifier if it can be resolved in compile time:

```swift
// preferred

let name = "Anakin Skywalker"
let planets = [.Tatooine, .Coruscant]
let colors = ["red": 0xff0000, "green": 0x00ff00]
```

```swift
// not preferred

let name: String = "Padmé"
let planets: [Planet] = [.Naboo, .Coruscant]
let colors: [String: UInt32] = ["blue": 0x0000ff, "white": 0xffffff]
```

Also, associate colon with type identifier.

```swift
// preferred

class VideoArticle: Article
let events: [Timestamp: Event]
```

```swift
// not preferred

class VideoArticle : Article
let events : [Timestamp : Event]
```

## Optionals
Declare variables and function return types as optional with `?` where a `nil` value is acceptable.

#### Avoid Using Force-Unwrapping of Optionals
If you have an identifier `foo` of type `FooType?`, don't force-unwrap (`foo!`) it to get to the underlying value. Force unwrapping is more prone to lead to runtime crashes.

Instead, use *Optional Binding* to safely unwrap the value:
```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```
Unwrapping several optionals in nested if-let statements is forbidden, as it leads to "pyramid of doom". Swift allows you to unwrap multiple optionals in one statement.
```swift
let name: String?
let age: Int?

if let name = name, age = age where age >= 13 {
    /* ... */
}
```
Or use `guard` statements
```swift
guard let modelURL = NSBundle.mainBundle().URLForResource(modelName, withExtension:"momd") else {
            fatalError("Error loading model from bundle")
        }
        
        guard let managedObjectModel = NSManagedObjectModel(contentsOfURL: modelURL) else {
            fatalError("Error initializing mom from: \(modelURL)")
        }
        
        let coordinator = NSPersistentStoreCoordinator(managedObjectModel: managedObjectModel)
```
You might want to use Swift's Optional Chaining in some of these cases, such as:
```swift
// Call the function if `foo` is not nil. If `foo` is nil, ignore we ever tried to make the call
foo?.callSomethingIfFooIsNotNil()
```
#### Avoid Using Implicitly Unwrapped Optionals
Wherever possible, use `let foo: FooType?` instead of `let foo: FooType!`.

Use implicitly unwrapped types only for instance variables that you know will be initialized later before use, such as subviews that will be set up in viewDidLoad.

## Closure Expressions

If the last argument of a function is a closure, use trailing closure syntax. 

Trailing closure syntax should be used if a function accepts a closure as its last argument. If it's its only argument, parentheses may be ommited. Unused closure arguments should be replaced with `_` (or fully ommited if no arguments are used). Argument types should be inferred.

```swift
func executeRequest<T>(request: Request<T>, completion: (T, Error?) -> Void)

executeRequest(someRequest) { (result, _) in /* ... */ }
```

Use implicit `return` in one-line closures with clear context.

```swift
let numbers = [1, 2, 3, 4, 5]
let even = filter(numbers) { $0 % 2 == 0 }
```

## Classes and Structures
Remember, structs have value semantics. Use structs for things that do not have an identity. An array that contains [a, b, c] is really the same as another array that contains [a, b, c] and they are completely interchangeable. It doesn't matter whether you use the first array or the second, because they represent the exact same thing. That's why arrays are structs.

Classes have reference semantics. Use classes for things that do have an identity or a specific life cycle. You would model a person as a class because two person objects are two different things. Just because two people have the same name and birthdate, doesn't mean they are the same person. But the person's birthdate would be a struct because a date of 3 March 1950 is the same as any other date object for 3 March 1950. The date itself doesn't have an identity.

> _Value types_ are great for representing **data** in the app.
> _Reference_ types are great to represent **behavior** in the app.

Note that inheritance is (by itself) usually not a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

For example, this class hierarchy:
```swift
class Vehicle {
    let name: String
    let numberOfWheels: Int
    
    init(numberOfWheels: Int) {
        self.numberOfWheels = numberOfWheels
    }

    func description() -> String {
        return "The vehicle is a \(name) which runs on \(numberOfWheels) wheels."
    }
}

class Bicycle: Vehicle {
    init() {
        super.init(numberOfWheels: 2)
    }
}

class Car: Vehicle {
    init() {
        super.init(numberOfWheels: 4)
    }
}
```
could be refactored into these definitions:
```swift
protocol Vehicle {
    var numberOfWheels: Int { get }
    var name: String { get} 
    func description() -> String
}

struct Bicycle: Vehicle {
    let name = "Bicycle"
    let numberOfWheels = 2
}
struct Car: Vehicle {
    let name = "Car"
    let numberOfWheels = 4
}

extension Vehicle {
    func description() -> String {
        return "The vehicle is a \(name) which runs on \(numberOfWheels) wheels."
    }
}
```
#### Use of self
For conciseness, avoid using `self` since Swift does not require it to access an object's properties or invoke its methods.

Use `self` when required to differentiate between property names and arguments in initializers, and when referencing properties in closure expressions (as required by the compiler):
```swift
class BoardLocation {
  let row: Int, column: Int

  init(row: Int, column: Int) {
    self.row = row
    self.column = column

    let closure = {
      println(self.row)
    }
  }
}
```
#### Protocol Conformance
When adding protocol conformance to a class, prefer adding a separate class extension for the protocol methods. This keeps the related methods grouped together with the protocol and can simplify instructions to add a protocol to a class with its associated methods.

Also, don't forget the `// MARK: ` comment to keep things well-organized!
**Preferred :**
```swift
class MyViewcontroller: UIViewController {
  // class stuff here
}

// MARK: - UITableViewDataSource
extension MyViewcontroller: UITableViewDataSource {
  // table view data source methods
}

// MARK: - UIScrollViewDelegate
extension MyViewcontroller: UIScrollViewDelegate {
  // scroll view delegate methods
}
```
**Not Preferred :**
```swift
class MyViewcontroller: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
  // all methods
}
```

#### Struct Intializers
Use the native Swift struct initializers rather than the legacy constructors.

**Preferred:**
```swift
let bounds = CGRect(x: 40, y: 20, width: 120, height: 80)
let centerPoint = CGPoint(x: 96, y: 42)
```
**Not Preferred:**
```swift
let bounds = CGRectMake(40, 20, 120, 80)
let centerPoint = CGPointMake(96, 42)
```
Prefer the struct-scope constants `CGRect.infinite`, `CGRect.null`, etc. over global constants `CGRectInfinite`, `CGRectNull`, etc. For existing variables, you can use the shorter `.zero`.

## Singletons
Mark the `init` method as `private`. This prevents others from using the default '()' initializer for the class.
```swift
final class Analytics {
    static let sharedInstance = Analytics()
    private init() {} 
}
```
Also, there is no need to wrap it inside  `dispatch_once` for thread-safety or concurrency concers. Quoting from [Apple's blog](https://developer.apple.com/swift/blog/?id=7) -
>The lazy initializer for a global variable (also for static members of structs and enums) is run the first time that global is accessed, and is launched as `dispatch_once` to make sure that the initialization is atomic. This enables a cool way to use `dispatch_once` in your code: just declare a global variable with an initializer and mark it private.

That said, use of singletons can be exploited. 
> 'Wisely use them, you must!'

## Semicolons
Swift does not require a semicolon after each statement in your code. They are only required if you wish to combine multiple statements on a single line.

Do not write multiple statements on a single line separated with semicolons.

The only exception to this rule is the `for-conditional-increment` construct, which requires semicolons. However, alternative `for-in` constructs should be used where possible.

## Miscellaneous
* All methods and functions shall have a single exit point. Exceptions if you're using `guard` statements.
* Do not use numeric values or strings (i.e., Magic numbers); use symbolic values instead.
* Declare each variable with the smallest possible scope and initialize it at the same time.
* Don't commit code that will never execute; just delete it. This applies to:
    * Methods that are never called.
    * Commented-out code
* Don't commit code that serves no purpose. This applies to:
    * Code automatically generated by Xcode that does nothing except call `super`

----

## Credits
This guide is inspired (shamelessly-taken-from) these sources -
* [The Official raywenderlich.com Swift Style Guide](https://github.com/raywenderlich/swift-style-guide)
* [Github Swift Style Guide](https://github.com/github/swift-style-guide)
* A little help from [@darcwader](https://github.com/darcwader), [@peculiar](https://github.com/peculiar).

----
If you have any suggestions, feel free to send a pull request.


