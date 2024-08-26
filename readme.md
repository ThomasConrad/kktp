# KTTP - Kristian Thomas Transfer Protocol

This repository contains the code and documentation for the Kristian Thomas Transfer Protocol (KKTP). KKTP is a simple binary protocol designed for efficiently transferring serialized data between two systems. The protocol aims to be straightforward to implement and use, while maintaining high performance and low overhead.

## Table of Contents
- [KTTP - Kristian Thomas Transfer Protocol](#kttp---kristian-thomas-transfer-protocol)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Protocol Overview](#protocol-overview)
    - [Key Design Principles](#key-design-principles)
  - [KKTP Wire Format](#kktp-wire-format)
    - [Variable Length Integers](#variable-length-integers)
      - [Example](#example)
    - [Endianness](#endianness)
    - [Primitive Types](#primitive-types)
    - [Meta Types](#meta-types)
  - [Menu Format](#menu-format)
    - [Example](#example-1)
    - [Menu Format Grammar](#menu-format-grammar)
  - [Usage](#usage)
    - [Example of Encoding and Decoding](#example-of-encoding-and-decoding)

## Introduction

KKTP (Kristian Thomas Transfer Protocol) is a binary protocol designed to transfer serialized data between two systems in a simple, efficient manner. KKTP is ideal for scenarios where simplicity and efficiency are paramount, such as embedded systems, IoT devices, and applications requiring low-latency communication.

## Protocol Overview

KKTP is designed to be a lightweight, simple and efficient binary protocol for data transfer.

### Key Design Principles

- **Simplicity**: KKTP is designed to be easy to implement across different platforms and programming languages.
- **Efficiency**: By using a binary format, KKTP minimizes the overhead associated with data transfer, making it suitable for high-performance applications.
- **Flexibility**: The protocol supports a variety of primitive and complex types, including nested messages, enums, and optional types.

## KKTP Wire Format

### Variable Length Integers

KKTP uses variable-length encoding for integers to optimize space usage. The encoding uses a variable number of bytes, where the most significant bit (MSB) of each byte indicates if there are more bytes to follow. The remaining 7 bits are used to store the integer value.

#### Example

An integer value of `300` would be encoded as follows:

1. Convert `300` to binary: `100101100`.
2. Split the binary into 7-bit groups from right: `0010110` `0000001`.
3. Since there are more bytes to follow, set the MSB of the first group: `10010110`.
4. The second group is the final byte, so keep the MSB clear: `00000010`.
5. The encoded result is two bytes: `0xAC 0x02`.

### Endianness

All multi-byte values in the KKTP protocol are encoded in little-endian byte order. This means the least significant byte (LSB) is stored first. This endianness choice aligns with most CPU architectures, optimizing data serialization and deserialization performance.

### Primitive Types

KKTP supports a variety of primitive types to provide flexibility in message design:

- `u8`: 8-bit unsigned integer
- `u16`: 16-bit unsigned integer, varint encoded
- `u32`: 32-bit unsigned integer, varint encoded
- `u64`: 64-bit unsigned integer, varint encoded
- `u16-raw`: 16-bit unsigned integer, fixed size
- `u32-raw`: 32-bit unsigned integer, fixed size
- `u64-raw`: 64-bit unsigned integer, fixed size
- `i8`: 8-bit signed integer, varint encoded
- `i16`: 16-bit signed integer, varint encoded
- `i32`: 32-bit signed integer, varint encoded
- `i64`: 64-bit signed integer, varint encoded
- `i16-raw`: 16-bit signed integer, fixed size
- `i32-raw`: 32-bit signed integer, fixed size
- `i64-raw`: 64-bit signed integer, fixed size
- `f32`: 32-bit floating point number
- `f64`: 64-bit floating point number
- `bool`: Boolean value, encoded as a single byte (`0x00` for `false`, `0x01` for `true`)
- `string`: A sequence of characters, UTF-8 encoded

### Meta Types

- **Variable Length Array**: Encoded as a `varint` length followed by `length` elements of the specified type.
- **Fixed Length Array**: A specified `length` number of elements of the given type.
- **Optional**: A `bool` indicating if the value is present (`true`), followed by the value if present.
- **Enum**: A `varint` indicating the variant, followed by the variant data.

## Menu Format

The KKTP protocol uses a non-self-describing format, meaning it relies on external definitions to understand message structures. The **menu format** is a simple textual description that outlines the structure of messages, including their fields and types. Menu files have the extension `.menu`, and are stored in plain text.

### Example

Below is an example of a menu format that defines several messages and their structures:

```plaintext
enum Rye {
    sourdough : u32,
    seeded : u16
}

enum Bread {
    white : u32,
    wholemeal : u16,
    rye : Rye
}   

dish Dinner {
    starter : [bread],
    main : u32,
    dessert : string,
    drink : [f16; 2]
}

dish Lunch {
    main : u32,
    drink : ?f16
}

services {
    1 : Dinner {
        request : Dinner,
        response : Lunch
    }

    2 : Lunch {
        1 : Pasta {
            request : u32,
            response : bool
        }

        2 : Salad {
            request : u32,
            response : ? u32
        }
    }

    3 : TableBread {
        response : Bread
    }
}
```

### Menu Format Grammar

```bnf
<grammar> ::= (<enum> | <dish> | <services>)*

<enum> ::= "enum" <identifier> "{" <enum_fields> "}"

<enum_fields> ::= <enum_field> ("," <enum_field>)*
<enum_field> ::= <identifier> ":" <type>

<dish> ::= "dish" <identifier> "{" <dish_fields> "}"

<dish_fields> ::= <dish_field> ("," <dish_field>)*
<dish_field> ::= <identifier> ":" <type>

<services> ::= "services" "{" <service_definitions> "}"

<service_definitions> ::= (<service_definition>)*
<service_definition> ::= <service_id> ":" <identifier> "{" <service_fields> | <service_definitions> "}" 

<service_id> ::= <number>
<service_fields> ::= <request_field> | <response_field> | <request_field> "," <response_field>
<request_field> ::= "request" ":" <type>
<response_field> ::= "response" ":" <type>

<type> ::= <primitive_type> | <array_type> | <enum_type> | <identifier> | <optional_type>
<primitive_type> ::= "u8" | "u16" | "u32" | "u64" | "u16-raw" | "u32-raw" | "u64-raw" | "i8" | "i16" | "i32" | "i64" | "i16-raw" | "i32-raw" | "i64-raw" | "f32" | "f64" | "bool" | "string"
<array_type> ::= "[" <type> "]" | "[" <type> ";" <number> "]"
<optional_type> ::= "?" <type>
<enum_type> ::= <identifier>

<identifier> ::= [a-zA-Z_][a-zA-Z0-9_]*
<number> ::= [0-9]+
```

## Usage

KKTP uses the menu format to define a collection of services. Each service is identified by a unique ID and specifies the messages it can send and receive. Dishes are used to define the structure of these messages. Each dish is a collection of fields, each with a specified type. Types can be primitive types, array types, enum types, or even other dishes.

### Example of Encoding and Decoding

Below is a step-by-step example showing how to encode a `Dinner` message:

1. **Define the Message**:
   ```plaintext
   dish Dinner {
       starter : [bread],
       main : u32,
       dessert : string,
       drink : [f16; 2]
   }
   ```

2. **Create the Message in Code**:
   ```pseudo
   dinner_message = Dinner()
   dinner_message.starter = [Bread.white, Bread.rye.sourdough]
   dinner_message.main = 42
   dinner_message.dessert = "Ice Cream"
   dinner_message.drink = [1.5, 2.0]
   ```

3. **Serialize the Message**:
   - Convert each field to its binary representation according to its type.
   - Concatenate all the binary fields together to form the serialized message.
   - The resulting binary data can be sent over the network or stored to disk.

4. **Decode the Message**:
    - Receive the binary data.
    - Parse the binary data according to the menu format.
    - Convert the binary fields back to their respective types.
    - The decoded message can be used in the application logic.
  
---
