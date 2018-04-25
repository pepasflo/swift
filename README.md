# Swift snippets

Some examples of Swift code.

Note: type annotations have been added for clarity, but are not required (unless noted).


# Codable (Encodable & Decodable)


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

    enum CodingKeys: String, CodingKey {
        case firstName = "first_name"
    }
}
```

Encoding:

```swift
let a: Author = Author(firstName: "Mark")
let d: Data = try! JSONEncoder().encode(a)
```

Check the result:

```swift
let s: String = String(data: d, encoding: .utf8)!
print(s)
```

```
{"first_name":"Mark"}
```

Decoding:

```swift
        let s2: String = """
{
    "first_name": "Mark"
}
"""
let d2: Data = s2.data(using: .utf8)!
let a2: Author = try! JSONDecoder().decode(Author.self, from: d2)
```

