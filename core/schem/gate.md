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
    /// returns the gate definition (details below)
    fn definition(&self) -> GateDefinition;

    /// the request contains the values of the current inputs
    /// the gate should return the list of outputs in order in definition
    /// the gate may modify its state
    fn tick(&mut self, request: GateTickRequest) -> Box<[*const ()]>;

    /// draws the (vector) graphics of its current state
    fn draw(&self, request: &GateDrawRequest) -> Graphic;

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
    pub fn deserialize_gate(gate: Box<[u8]>, props: Box<[u8]>) -> Result<Box<dyn Gate>, Box<str>>;

    /// reconstructs gate from and older version of byte vector
    pub fn deserialize_gate_upgrade(gate: Box<[u8]>, props: Box<[u8]>, from_version: (u16, u16)) -> Option<Result<Box<dyn Gate>, Box<str>>>;
    
    /// reconstructs properties container from byte vector
    /// returning the actual type (not boxed or anything)
    pub fn deserialize_gate_property(props: Box<[u8]>) -> Result<(concrete_type), Box<str>>;

    /// reconstructs properties container of an older version from byte vector
    /// returning the actual type (not boxed or anything)
    pub fn deserialize_gate_property_upgrade(props: Box<[u8]>) -> Option<Result<(concrete_type), Box<str>>>;
    ```

### Naming and Distribution

- Crate must have name **xdsim-[package-name]-gate**, for example **xdsim-and-gate**.
- Each crate can only contain one gate.
- The Gate is compiled to a dynamic library and placed in the folder so it is loaded by the core program.

<details>
<summary>Example implementation</summary>

#### Cargo.toml
```toml
[package]
name = "xdsim-t-ff-gate" # t type flip flop
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
pub struct TFlipFlop {
    value: bool,
    props: NoProperties
}

impl Gate for TFlipFlop {
    fn definition(&self) -> GateDefinition {
        GateDefinition {
            version: 0,

            bounding_box: Vec2 { x: 1.0, y: 1.0 },
            identifier: ("t-flip-flop", "tflipflop", 0, 1), // package=t-flip-flop, struct=TFlipFlop, version=0.1

            inputs: Box::new([
                GateIOEntry {
                    name: "T".into(),
                    data_type: ("common", "bit", 0, 1),
                    position: Vec2 { x: 0.0, y: 0.5 }
                }
            ]),

            outputs: Box::new([
                GateIOEntry {
                    name: "Q".into(),
                    data_type: ("common", "bit", 0, 1),
                    position: Vec2 { x: 1.0, y: 0.7 },
                },
                GateIOEntry {
                    name: "QBar".into(),
                    data_type: ("common", "bit", 0, 1),
                    position: Vec2 { x: 1.0, y: 0.3 },
                }
            ])
        }
    }

    fn tick(&mut self, request: GateTickRequest) -> Box<[*const ()]> {
        let input_t: &Bool = request.get_input<Bool>(0);

        if input_t.0 {
            self.value = !self.value;
        }

        Box::new([
            mem::transmute(Box::new(self.value.clone()).into_raw())
        ])
    }

    fn draw(&self, _request: &GateDrawRequest) -> Graphic {
        Graphic::from_vec(vec![
            Element::Rect {
                pos: Vec2 { x: 0.0, y: 0.0 },
                size: Vec2 { x: 0.0, y: 0.0 },
                stroke: StrokeStyle { colour: Colour::Fg },
                fill: FillStyle {
                    colour: Colour::Transparent,
                },
            },
            Element::Rect {
                pos: Vec2 { x: 0.4, y: 0.4 },
                size: Vec2 { x: 0.2, y: 0.2 },
                stroke: StrokeStyle {
                    colour: Colour::Black,
                },
                fill: FillStyle {
                    colour: if self.value.0 {
                        Colour::Black
                    } else {
                        Colour::Yellow
                    },
                },
            },
        ])
    }

    fn properties_container(&self) -> &dyn PropertiesContainer {
        &self.props
    }

    fn properties_container_mut(&mut self) -> &mut dyn PropertiesContainer {
        &mut self.props
    }

    fn serialize(&self) -> Box<[u8]> {
        self.value.serialize()
    }
}

pub fn create_gate() -> Box<dyn Gate> {
    Box::new(TFlipFlop {
        value: Bit(false),
        props: NoProperties,
    })
}

pub fn deserialize_gate(gate: Box<[u8]>, props: Box<[u8]>) -> Box<dyn Gate> {
    Box::new(TFlipFlop {
        value: Bit::deserialize(gate),
        props: deserialize_gate_property(props),
    })
}

pub fn deserialize_gate_property(_props: Box<[u8]>) -> NoProperties {
    NoProperties
}
```

</details>
