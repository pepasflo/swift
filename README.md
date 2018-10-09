# Swift snippets

Some examples of Swift code.

Note: type annotations have been added for clarity, but are not required (unless noted).

# `typealias`

```swift
typealias APIToken = String
```


# Enumerating the keys and values in a Dictionary

```swift
for (k, v) in somedict {
    ...
}
```

# List of Dictionary keys as Array

It turns out that `Dictionary.keys` does not return an `Array` of the keys (it returns an object of type `Dictionary.Keys`).

```swift
Array(somedict.keys)
```


# Dictionary key membership

To check if a key exists in a dictionary:

```swift
if dict.keys.contains(key) {
    ...
}
```

# Deprecation

Marking a function as deprecated:

```swift
@available(*, deprecated)
func myFunc() {}
```

More: https://stackoverflow.com/a/25406285/7543271


# String to Data

```swift
let s: String = "hello"
let d: Data = s.data(using: .utf8)!
```

# Data to String

```swift
let d: Data = Data(bytes: [104, 101, 108, 108, 111, 0]) // "hello"
let s: String = String(data: d, encoding: .utf8)!
```

Resources:
- https://www.objc.io/blog/2018/02/13/string-to-data-and-back/


# Codable (Encodable & Decodable)

Resources:
- https://benscheirman.com/2017/06/swift-json/
- https://hackernoon.com/everything-about-codable-in-swift-4-97d0e18a2999

## Encoding primitives:

You can't do this (yet):

```swift
let s: String = "hello"
let d: Data = try! JSONEncoder().encode(s)
```

> Thread 1: Fatal error: 'try!' expression unexpectedly raised an error: Swift.EncodingError.invalidValue("hello", Swift.EncodingError.Context(codingPath: [], debugDescription: "Top-level String encoded as string JSON fragment.", underlyingError: nil))

The top-level object of the resulting JSON output must be a dictionary or an array (not a string, number, boolean, or nil).

However, this may be supported in the future, see:
- https://bugs.swift.org/browse/SR-7213
- https://bugs.swift.org/browse/SR-6163


## Encoding / Decoding a basic `struct`:

```swift
struct Movie: Codable {
    let id: Int
    let title: String
}
```

Encoding:

```swift
let m: Movie = Movie(id: 1, title: "Goonies")
let d: Data = try! JSONEncoder().encode(m)
```

Decoding from `Data`:

```
let m2: Movie = try! JSONDecoder().decode(Movie.self, from: d)
```

Decoding from `String`:

Convert the `String` to `Data` first, then decode as above:

```swift
let s: String = """
{
    "id": 1,
    "title": "Goonies"
}
"""
let d3: Data = s.data(using: .utf8)!
let m3: Movie = try! JSONDecoder().decode(Movie.self, from: d3)
```

## Encoding / Decoding a nested `struct`:

As long as the `struct`s are all `Codable`, they can be nested:

```swift
struct Author: Codable {
    let name: String
}

struct Book: Codable {
    let title: String
    let author: Author
}
```

Encoding:

```swift
let b: Book = Book(title: "Not Always So", author: Author(name: "Shunryu Suzuki"))
let d: Data = try! JSONEncoder().encode(b)
```

Decoding:

```swift
let b2: Book = try! JSONDecoder().decode(Book.self, from: d)
```

## Decoding ISO8601 dates

```swift
struct Movie: Codable {
    let title: String
    let releaseDate: Date

    enum CodingKeys: String, CodingKey {
        case title
        case releaseDate = "release_date"
    }
}
```

This doesn't work out of the box:

```swift
let s: String = """
{
    "title": "Groundhog Day",
    "release_date": "1993-02-12T22:47:51+0000"
}
"""

let d: Data = s.data(using: .utf8)!
let m: Movie = try! JSONDecoder().decode(Movie.self, from: d)
```

> Thread 1: Fatal error: 'try!' expression unexpectedly raised an error: Swift.DecodingError.typeMismatch(Swift.Double, Swift.DecodingError.Context(codingPath: [CodingKeys(stringValue: "release_date", intValue: nil)], debugDescription: "Expected to decode Double but found a string/data instead.", underlyingError: nil))

You need to set the `dateDecodingStrategy` on the `JSONDecoder`:

```swift
let d: Data = s.data(using: .utf8)!
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
let m: Movie = try! decoder.decode(Movie.self, from: d)
```

Resources:
- https://useyourloaf.com/blog/swift-codable-with-custom-dates/

## `Codable` extensions

We can write a pair of extensions to make this a bit more succinct, for the common case where we simply want an optional returned (and aren't interested in examining the exception).

```swift
struct Author: Codable {
    let name: String
}
```

Encoding:

```swift
extension Encodable {
    public var encoded: Data? {
        return try? JSONEncoder().encode(self)
    }
}
```

```swift
let a: Author = Author(name: "Mark Twain")
let d: Data = a.encoded!
```

Decoding:

```swift
let a2: Author = d.decoded()!
```

Note: the type annotation on `a2` is required.


## Custom key mapping

Define a `CodingKeys` enum which adheres to `String, CodingKey`:

```swift
struct Author: Codable {
    let firstName: String
    let age: Int

    enum CodingKeys: String, CodingKey {
        case firstName = "first_name"
        case age
    }
}
```

Encoding:

```swift
let a: Author = Author(firstName: "Mark", age: 45)
let d: Data = try! JSONEncoder().encode(a)
```

Check the result:

```swift
let s: String = String(data: d, encoding: .utf8)!
print(s)
```

```
{"age":45,"first_name":"Mark"}
```

Decoding:

```swift
let s2: String = """
{
    "first_name": "Mark",
    "age": 45
}
"""
let d2: Data = s2.data(using: .utf8)!
let a2: Author = try! JSONDecoder().decode(Author.self, from: d2)
```

# Using a local JSON file in unit tests

```swift
struct Movie: Codable {
    let title: String
}
```

`movie.json` (make sure this file is added to your unit test target, not your main target):

```json
{
    "title": "Groundhog Day"
}
```

```swift
import XCTest
@testable import CodableDemo

class MovieTests: XCTestCase {
    func testMovieDecode() {
        let b: Bundle = Bundle(for: type(of: self))
        let u: URL = b.url(forResource: "movie", withExtension: "json")!
        let d: Data = try! Data(contentsOf: u)
        let m: Movie = try! JSONDecoder().decode(Movie.self, from: d)

        XCTAssertEqual(m.title, "Groundhog Day")
    }
}
```

# Dynamic UITableViewCell height

```swift
self.tableView.rowHeight = UITableViewAutomaticDimension
self.tableView.estimatedRowHeight = UITableViewAutomaticDimension
```

- Remove any related function definitions (e.g. `heightForRowAtIndexPath`).
- Ensure your cells have a continuous chain of vertical constraints from top to bottom.


# Type declaration with protocol conformance

```swift
var playableCell: UITableViewCell & Playable = ...
```


# `updateConstraints`

## Call to `super`

`super.updateConstraints` should be called last.  Use defer:

```swift
public override func updateConstraints() {
    defer {
        super.updateConstraints()
    }

    ...
}
```

## Performance considerations

[WWDC 2018 #220](https://developer.apple.com/videos/play/wwdc2018/220/) is a great resource for building an intuition around the performance impact of calling [`updateConstraints`](https://developer.apple.com/documentation/uikit/uiview/1622512-updateconstraints).  TL;DR: keep this method as optimized as possible.


# Programatically creating UIWindow

```swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?
    ) -> Bool {

        let window = UIWindow(frame: UIScreen.main.bounds)
        window.makeKeyAndVisible()
        window.rootViewController = UIViewController()
        self.window = window

        return true
    }
}
```

Also remember to empty out the "Main Interface" field under the "General" tab of your target's settings.


# Switching on the type of something:

without casting:

```swift
switch item {
case is Int:
    ...
case is Double:
    ...
```

with casting:

```swift
switch item {
case let i as Int:
    ...
case let d as Double:
    ...
```

# Switching on a tuple:

Sometimes you need to switch on the cases of two enums:

```swift
enum Content {
    case article(URL)
    case video(URL)
}

let a: Content = .article(URL(string: "http://google.com")!)
let b: Content = .video(URL(string: "http://yahoo.com")!)

switch (a, b) {
case (.article(let url1), .article(let url2)):
    ...
case (.video(let url1), .video(let url2)):
    ...
default:
    ...
}
```

# Adding a custom extension to `Observable` (RxSwift):

```swift
extension Observable where E == Int {
    public func foo() -> Observable<Int> {
        ...
    }
}
```

# Importing a specific member of a framework

```swift
import struct FloSportsCore.Event
```
