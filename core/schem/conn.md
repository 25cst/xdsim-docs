# Connection

A **Connection** is a component that connects two points in a circuit. Such as from the output of one gate to the input of another.
- Each **Connection** has a [**Type**](./type.md) that it supports (e.g. a connection may support only the **Bit** Type). It can only connect to inputs and outputs of that specific Type.
- A Connection can have *junctions*, each junction must have at least one edge going into the junction, and at least 2 edges going out from the junction.

  Therefore, a Connection can connect to at most one output and any number of inputs.
  > A junction with less than two outgoing edges can be removed as there is no branching.

> [!NOTE]
> In the simulation, a **Connection** carries data from the output of one gate to the input of connected gates.
> - On each tick, the value of the connected outputs *before the tick* will be passed to the connected inputs.
> - If the Connection is not connected to any gate outputs, it will take the default value of the **Type**.

## Definition

A **Connection** is a crate with the following public structs and functions.

- A struct that implements the **Connection** trait.
  ```rs
    /// Produce a graphic of the connection.
    /// Request is a reference because it doesn't really make sense
    /// for the draw function to take ownership of anything.
    fn draw(&self, request: &ConnectionDrawRequest) -> Graphic;

    /// Returns connection definition. This may depend on options so takes `&self`.
    fn definition(&self) -> ConnectionDefinition;

    /// Get the property container
    /// this is to be implemented by macro
    fn properties_container(&self) -> &dyn PropertiesContainer;

    /// Get the property container (mutable)
    /// this is to be implemented by macro
    fn properties_container_mut(&mut self) -> &mut dyn PropertiesContainer;

    /// Serialize connection into bytes
    /// macro will remove the &self from the argument list
    fn serialize(&self) -> Box<[u8]>;
  ```
  > **PropertiesContainer** represents properties of the **Connection** that can be changed by the user. E.g. the display colour of the connection. See [PropertiesContainer](./props.md) for more info.
  >
  > **ConnectionDefinition** includes information about the gate that will not change as the simulation runs. It is allowed to use information from Connection properties.
  > - The [**Type**](./type.md) that the Connection supports.
  > - The unique identifier **(package-name, gate-name, crate semver major, minor)** of the connection.
  > - The schema version of the ConnectionDefinition, so new fields can be added to the schema without breaking Gate using older versions of the schema.
- Top-level functions that is exported by **lib.rs**
  ```rs
  /// create a new connection
  pub fn create_conn() -> Box<dyn Connection>;

  /// reconstructs properties container from byte vector
  pub fn deserialize_conn_property(props: Box<[u8]>) -> Result<(concrete_type), Box<str>>;

  /// reconstructs properties container of an older version from byte array
  /// returning the actual type (not boxed or anything)
  pub fn deserialize_conn_property_upgrade(props: Box<[u8]>, from_version: (u16, u16)) -> Option<Result<(concrete_type), Box<str>>>;
  ```

### Naming and Distribution

- Crate must have name **xdsim-[package-name]-conn**, for example xdsim-bit-conn.
- Each crate can only contain one connection.
- The Gate is compiled to a dynamic library and placed in the folder so it is loaded by the core program.

<details>
<summary>Example implementation</summary>

#### Cargo.toml
```toml
[package]
name = "xdsim-bit-conn" # t type flip flop
version = "0.1.0"
edition = "2024"

[dependencies]
# libxdsim and libxdsim-common-type

[lib]
crate-type = ["dylib"]
```

#### lib.rs

```rs
#[derive(Default)]
pub struct NoProperties;

impl PropertiesContainer for NoProperties {
    fn get_menu(&self) -> Menu {
        Menu { items: Box::new([]) }
    }

    fn get_option(&self) -> Option<MenuInputValue> {
        None
    }

    fn set_option(
        &mut self,
        _id: &str,
        _value: MenuInputValue,
    ) -> Result<(), PropertiesContainerSetError> {
        Err(PropertiesContainerSetError::PropertyDoesNotExist)
    }

    fn serialize(&self) -> Box<[u8]> {
        Box::new([])
    }
}

#[derive(Default)]
pub struct BitConn {
    props: NoProperties
}

impl Connection for BitConn {
    fn definition(&self) -> GateDefinition {
        GateDefinition {
            version: 0,

            identifier: ("bit", "bitconn", 0, 1), // package=t-flip-flop, struct=TFlipFlop, version=0.1
            data_type: ("common", "bit", 0, 1),
        }
    }

    fn draw(&self, _request: &ConnectionDrawRequest) -> Graphic {
        todo!()
    }

    fn properties_container(&self) -> &dyn PropertiesContainer {
        &self.props
    }

    fn properties_container_mut(&mut self) -> &mut dyn PropertiesContainer {
        &mut self.props
    }

    fn serialize_property(&self) -> Box<[u8]> {
        self.props.serialize()
    }
}

pub fn create_conn() -> Box<dyn Connection> {
    Box::new(BitConn {
        props: NoProperties,
    })
}

pub fn deserialize_conn(_props: Box<[u8]>) -> NoProperties {
    NoProperties
}
```

</details>
