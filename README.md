# Swift snippets

Some examples of Swift code.


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

Here's a basic `struct`:

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
