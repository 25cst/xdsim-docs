# Schema

All schema are defined in [libxdsim](https://github.com/25cst/libxdsim), they contain only structs and traits to help external libraries created by the user to communicate with the  core GUI program.

Each page contain the schema for the struct/trait, how it is implemented, and the macros to make implementation easier.

## Pages

|Page|Description|
|---|---|
|[Gate](./gate.md)|A **Gate** is a component that has state and functionality (computing output based on current input and current state).|
|[Connection](./conn.md)|A **Connection** is a component used to connect the output of one Gate to the input of another Gate. It has no state and no functionality.|
|[Type](./type.md)|Each input and output of a Gate has a predetermined **Type**, each Connection can only connect to input and output of a type that it supports.|

## Commonalities

**Gate** and **Connection** are provided to the program as dynamic library files. For a Connection to be able to connect to the input or output of the gate, it needs to support the exact type that the gate inputs/outputs. So both the Gate and Connection must depend on the same **Type** crate.
