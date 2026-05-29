# bincode

[![Zeta](https://img.shields.io/badge/Zeta-Language-%23FF6B6B?style=flat-square)](https://zeta-lang.org)
[![zorbs.io](https://img.shields.io/badge/zorbs.io-@encoding/bincode-%2300C4FF?style=flat-square)](https://zorbs.io/@encoding/bincode)
[![GitHub](https://img.shields.io/badge/GitHub-murphsicles/bincode-%23181717?style=flat-square)](https://github.com/murphsicles/bincode)

Compact binary serialization for Zeta. Bincode encodes Zeta data structures into a small, efficient binary format — ideal for network protocols, storage, and inter-process communication where JSON/TOML would be too large or slow.

## Quick Start

Add to your `zorb.toml`:

```toml
[dependencies]
@encoding/bincode = "1.0"
```

Serialize and deserialize any `Serializable` type:

```zeta
use @encoding/bincode::{serialize, deserialize};

fn main() {
    let original: [i32; 3] = [1, 2, 3];

    // Serialize to a byte vector
    let bytes = serialize(&original).unwrap();
    print("Serialized: {}\n", bytes.len(), "bytes");

    // Deserialize back
    let decoded: [i32; 3] = deserialize(bytes).unwrap();
    print("Decoded: {}, {}, {}\n", decoded[0], decoded[1], decoded[2]);
}
```

## Configuration

Bincode is highly configurable. Use `options()` for the builder API:

```zeta
use @encoding/bincode::{
    config::{DefaultOptions, Options},
    serialize, deserialize,
};

fn main() {
    let data: [i32; 3] = [42, 256, 65536];

    // Default: LittleEndian, Varint, Unlimited
    let bytes = DefaultOptions::new()
        .with_little_endian()
        .with_varint_encoding()
        .serialize(&data);

    // BigEndian with fixed integer encoding
    let bytes = DefaultOptions::new()
        .with_big_endian()
        .with_fixint_encoding()
        .with_limit(1024)          // Max 1KB output
        .serialize(&data);

    // Deserialize with same config
    let decoded: [i32; 3] = DefaultOptions::new()
        .with_big_endian()
        .with_fixint_encoding()
        .deserialize(bytes);
}
```

## API Overview

| Function / Type | Description |
|----------------|-------------|
| `serialize(value)` | Serialize a value into a `Vec<u8>` |
| `deserialize(bytes)` | Deserialize from a byte slice |
| `serialize_into(writer, value)` | Serialize into a writable stream |
| `deserialize_from(reader)` | Deserialize from a readable stream |
| `DefaultOptions` | Builder for configuration (endianness, int encoding, size limits) |
| `Options` trait | Trait implemented by all configuration combinators |

### Configuration Options

| Method | Description |
|--------|-------------|
| `.with_little_endian()` / `.with_big_endian()` | Byte order (default: Little) |
| `.with_varint_encoding()` / `.with_fixint_encoding()` | Integer encoding — varint is smaller, fixint is faster (default: Varint) |
| `.with_limit(N)` | Max output size in bytes (default: unlimited) |
| `.allow_trailing_bytes()` / `.reject_trailing_bytes()` | Behaviour for extra data after deserialization (default: reject) |

## When to Use Bincode

- **Network protocols**: Compact wire format without schema negotiation
- **Cache storage**: Faster than parsing JSON, smaller on disk
- **State serialization**: Save/restore application state quickly
- **Game state**: Binary format suits game save files and asset baking

## Tips

- Varint encoding produces smaller output for small integers but slightly larger for large ones
- Set a size limit with `.with_limit()` to prevent untrusted input from allocating huge buffers
- Bincode is **not a self-describing format** — the schema must be known on both sides (use serde_json or toml if you need human-readable or schema-less formats)

## License

MIT
