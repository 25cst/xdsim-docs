# Type

A (data) **Type** that can be used as an input or output to a gate.
E.g. **Bit** or **Byte**.
- The output of a **Gate** can be connected to an input of **Gate** only if they have the same **Type**. E.g. an input accepting a **Bit** will not accept a **Byte** as input.
- Each **Connection** supports only one **Type**. E.g. a *Bit wire* can only connect to **Bit** input and outputs. A *Byte wire* can connect only to **Byte** input and outputs.

## Responsibilities

|Task|Description|
|---|---|
|Return **Type** identifier|Return a unique identifier of the Type containing the *package name*, the *struct name* in lowercase, and the *major and minor semver* of the crate.
|Serialize to byte array|Convert self into a byte array.|
|Deserialize from byte array|Convert byte array into self.|
|Deserialize from byte array of older version of self.|Given the version of Type that the byte array represents, convert byte array into self.|

## Definition

A **Type** is any struct that implements the following functions.

> [!IMPORTANT]
> Different parts of the program may depend on a Type.
> All of them need to agree on the byte layout of the Type.
>
> The Rust compiler may use a different byte layout depending on factors such as the operating system.
> You MUST tell the compiler to use the same byte layout everywhere by adding the tag **#[repr(C)]** before the Type struct.
>
> All structs your struct contains must also have the **#[repr(C)]** tag. Data types such as Vec and String don't have a stable byte layout.

```rs
/// (package name, lowercase struct name, semver major, minor)
pub const fn ident() -> (&'static str, &'static str, u16, u16);

/// serialize self into byte vector for saving
/// requires being able to reconstruct self from the byte vector
pub fn serialize(&self) -> Box<[u8]>;

/// reconstructs self from byte vector
pub fn deserialize(bytes: Box<[u8]>) -> Result<Self, Box<str>>;

/// reconstructs self from a byte vector,
/// but the byte vector represents an older version of self
/// return none if conversion from version is not supported
pub fn deserialize_upgrade(bytes: Box<[u8]>, from_version: (u16, u16)) -> Option<Result<Self, Box<str>>>;
```

### Naming and Distribution

- Crate must have name **xdsim-[package-name]-type**, for example **xdsim-bit-type**.
- Each crate is allowed to have more than one **Type**.

<details>
<summary>Example implementation</summary>

#### Cargo.toml
```toml
[package]
name = "xdsim-dummy-type"
version = "0.1.0"
edition = "2024"

[dependencies]
```

#### lib.rs

```rs
#[repr(C)] // IMPORTANT
#[derive(Clone, Copy)]
pub struct Bit(pub bool);

impl Bit {
    pub const fn ident() -> (&'static str, &'static str, u16, u16) {
        ("dummy", "bit", 0, 1)
    }

    pub fn serialize(&self) -> Vec<u8> {
        vec![if self.0 { 1 } else { 0 }]
    }

    pub fn deserialize(bytes: Vec<u8>) -> Result<Self, String> {
        match bytes.as_slice() {
            [ 0 ] => Ok(Self(false)),
                [ 1 ] => Ok(Self(true)),
                _ => Err(format!("invalid {:bytes?}"))
        }
    }

    pub fn deserialize_upgrade(bytes: Vec<u8>, from_version: (u16, u16)) -> Option<Result<Self, String>> {
        None
    }
}
```

</details>
