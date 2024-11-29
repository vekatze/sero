# marshal

`marshal` is a [Neut](https://vekatze.github.io/neut/) module that saves values to files and loads them from there.

## Installation

```sh
neut get marshal https://github.com/vekatze/marshal/raw/main/archive/0-1-16.tar.zst
```

## Types

### Main Definitions

```neut
// A "class" that specifies how to encode/decode values of type `a`
data marshal(a) {
| Marshal(
    encode: (buffer: &builder, value: &a) -> unit,
    decode: (bytes: &binary, cursor: &cell(int)) -> ?a,
    necess: (a) -> meta a, // ← used to load values in a memory-safe way
  )
}

// Encodes a value into a binary and saves it to a file at `path`.
define save<a>(m: marshal(a), value: &a, path: &text, buffer-size: int): system(unit)

// Decodes a value from a file at `path`.
define load<a>(m: marshal(a), path: &text): system(?a)
```

### Instance Constructors

You can quickly construct values of type `marshal(a)` by using some of the following terms:

```neut
constant this.class.binary.as-marshal: marshal(binary)

constant this.class.bool.as-marshal: marshal(bool)

constant this.class.float64.as-marshal: marshal(float64)

constant this.class.int32.as-marshal: marshal(int32)

constant this.class.int64.as-marshal: marshal(int64)

constant this.class.rune.as-marshal: marshal(rune)

constant this.class.text.as-marshal: marshal(text)

inline this.class.either.as-marshal<a, b>(!m1: marshal(a), !m2: marshal(b)): marshal(either(a, b))

inline this.class.list.as-marshal<a>(!m: marshal(a)): marshal(list(a))

inline this.class.matrix.as-marshal<a>(!m: marshal(a)): marshal(matrix(a))

inline this.class.pair.as-marshal<a, b>(!m1: marshal(a), !m2: marshal(b)): marshal(pair(a, b))

inline this.class.vector.as-marshal<a>(!m: marshal(a)): marshal(vector(a))
```

## Example

```neut
// Creates a class to save/load values of type list(int)
constant marshal-list-int: marshal(list(int)) {
  marshal.class.list.as-marshal(marshal.class.int64.as-marshal)
}

define zen(): unit {
  pin value: list(int) = [1, 2, 3, 4, 5, 6] in
  let path = "test.bin" in
  // Saves the value into `path`
  let _ = save(marshal-list-int, value, path, 10) in
  // Loads the value from `path`
  let v = load(marshal-list-int, path) in
  match v {
  | Left(e) =>
    printf("error: {}\n", [get-error-message(e)]);
  | Right(v) =>
    match v {
    | Left(_) =>
      Unit
    | Right(xs) =>
      for(xs, function (x) {
        printf("{}\n", [show-int(x)])
      })
    }
  }
}

```
