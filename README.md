[![Workflow Status](https://github.com/enarx/mmarinus/workflows/test/badge.svg)](https://github.com/enarx/mmarinus/actions?query=workflow%3A%22test%22)
[![Average time to resolve an issue](https://isitmaintained.com/badge/resolution/enarx/mmarinus.svg)](https://isitmaintained.com/project/enarx/mmarinus "Average time to resolve an issue")
[![Percentage of issues still open](https://isitmaintained.com/badge/open/enarx/mmarinus.svg)](https://isitmaintained.com/project/enarx/mmarinus "Percentage of issues still open")
![Maintenance](https://img.shields.io/badge/maintenance-activly--developed-brightgreen.svg)

# mmarinus

The `mmarinus` crate wraps the underlying system `mmap()` call in safe semantics.

For example:

```rust
use std::num::NonZeroUsize;
use mmarinus::{Kind, Builder, perms};

let mut zero = std::fs::File::open("/dev/zero").unwrap();

let map = Builder::map(4096)
    .near(128 * 1024 * 1024)
    .from(&mut zero, 0)
    .known::<perms::Read>(Kind::Private)
    .unwrap();

assert_eq!(&*map, &[0; 4096]);
```

You can also remap an existing mapping:

```rust
use std::num::NonZeroUsize;
use mmarinus::{Kind, Builder, perms};

let mut zero = std::fs::File::open("/dev/zero").unwrap();

let mut map = Builder::map(4096)
    .anywhere()
    .from(&mut zero, 0)
    .known::<perms::Read>(Kind::Private)
    .unwrap();

assert_eq!(&*map, &[0; 4096]);

let mut map = map.remap()
    .from(&mut zero, 0)
    .known::<perms::ReadWrite>(Kind::Private)
    .unwrap();

assert_eq!(&*map, &[0; 4096]);
for i in map.iter_mut() {
    *i = 255;
}
assert_eq!(&*map, &[255; 4096]);
```

Alternatively, you can just change the permissions:

```rust
use std::num::NonZeroUsize;
use mmarinus::{Kind, Builder, perms};

let mut zero = std::fs::File::open("/dev/zero").unwrap();

let mut map = Builder::map(4096)
    .at(128 * 1024 * 1024)
    .from(&mut zero, 0)
    .known::<perms::Read>(Kind::Private)
    .unwrap();

assert_eq!(&*map, &[0; 4096]);

let mut map = map.reprotect::<perms::ReadWrite>().unwrap();

assert_eq!(&*map, &[0; 4096]);
for i in map.iter_mut() {
    *i = 255;
}
assert_eq!(&*map, &[255; 4096]);
```

Mapping a whole file into memory is easy:

```rust
use std::num::NonZeroUsize;
use mmarinus::{Kind, Builder, perms};

let map = Kind::Private.load::<perms::Read, _>("/etc/os-release").unwrap();
```

License: Apache-2.0
