# Type

A (data) **Type** that can be used as an input or output to a gate.
E.g. **Bit** or **Byte**.
- The output of a **Gate** can be connected to an input of **Gate** only if they have the same **Type**. E.g. an input accepting a **Bit** will not accept a **Byte** as input.
- Each **Connection** supports only one **Type**. E.g. a *Bit wire* can only connect to **Bit** input and outputs. A *Byte wire* can connect only to **Byte** input and outputs.

## Definition

A **Type** is any struct that implements the following functions.

```rs
/// (package name, lowercase struct name, semver major, minor)
pub fn id() -> (&'static str, &'static str, u16, u16);

/// serialize self into byte vector for saving
/// requires being able to reconstruct self from the byte vector
pub fn serialize(&self) -> Vec<u8>;

/// reconstructs self from byte vector
/// from_version: (semver major, minor)
pub fn deserialize(bytes: Vec<u8>, from_version: (u16, u16)) -> Result<Self, String>;
```

### Naming and Distribution

- Crate must have name **xdsim-[package-name]-type**, for example **xdsim-bit-type**.
- Each crate is allowed to have more than one **Type**.
