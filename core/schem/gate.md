# Gate

A **Gate** is a component that produces an output when provided with current inputs (e.g. an AND gate). It is allowed to have state which can be used and changed when calculating an output (e.g. a flip flop).
- A **Gate** can have multiple input and outputs.
- On each *tick*, the simulator provides the gate with the current value of its inputs and requests the value of its outputs.
- A **Gate** can be placed anywhere in the project world, it can also be rotated in the four directions. 

> [!NOTE]
> A **Gate** cannot be broken down into smaller gates, it is the most basic component.

## Definition

A **Gate** is a crate with the following public structs and functions.

- A struct that implements the **Gate** trait.
    ```rs
    /// the request contains the values of the current inputs
    /// the gate should return the list of outputs in order in definition
    /// the gate may modify its state
    fn tick(&mut self, request: GateTickRequest) -> Box<[*const ()]>;

    /// draws the (vector) graphics of its current state
    fn draw(&self, request: &GateDrawRequest) -> Graphic;

    /// returns the gate definition (details below)
    fn definition(&self) -> GateDefinition;

    /// returns the properties container (detail below)
    fn properties_container(&self) -> &dyn PropertiesContainer;
    fn properties_container_mut(&mut self) -> &mut dyn PropertiesContainer;

    /// serialize self into bytes
    fn serialize(&self) -> Box<[u8]>;
    ```

    > **PropertiesContainer** represents properties of the Gate that can be changed by the user. E.g. number of inputs for an AND gate.
    > See [PropertiesContainer](./props.md) for more info.
    >
    > **GateDefinition** includes information about the gate that will not change as the simulation runs. It is allowed to use information from Gate properties.
    > - The list of input and output [**Type**](./type.md) for the Gate.
    > - The visual *bounding box* for the Gate.
    > - The position of the input and outputs of the Gate: where can Connections connect to. These positions must be on a side (not corner) of the bounding box so their directions are unambiguous.
    > - The unique identifier **(package-name, gate-name, crate semver major, minor)** of the gate.
    > - The schema version of the GateDefinition, so new fields can be added to the schema without breaking Gate using older versions of the schema.

- Top-level functions that is exported by **lib.rs**
    ```rs
    /// create a new gate
    pub fn create_gate() -> Box<dyn Gate>;
    
    /// reconstructs gate from byte vector
    pub fn deserialize_gate(gate: Box<[u8]>, properties: Box<[u8]>) -> Result<Box<dyn Gate>, Box<str>>;

    /// reconstructs gate from and older version of byte vector
    pub fn deserialize_gate_upgrade(gate: Box<[u8]>, properties: Box<[u8]>, from_version: (u16, u16)) -> Option<Result<Box<dyn Gate>, Box<str>>>;
    
    /// reconstructs properties container from byte vector
    /// returning the actual type (not boxed or anything)
    pub fn deserialize_gate_property(properties: Box<[u8]>) -> Result<(concrete_type), Box<str>>;

    /// reconstructs properties container of an older version from byte vector
    /// returning the actual type (not boxed or anything)
    pub fn deserialize_gate_property_upgrade(properties: Box<[u8]>) -> Option<Result<(concrete_type), Box<str>>>;
    ```

### Naming and Distribution

- Crate must have name **xdsim-[package name]-gate**, for example **xdsim-and-gate**.
- Each crate can only contain one gate.

<details>
<summary>Example implementation</summary>

#### Cargo.toml
```toml
[package]
name = "xdsim-t-ff-gate" # t type flip flop
version = "0.1.0"
edition = "2024"

[dependencies]
```

#### lib.rs

```rs
// TODO
```

</details>
