---
sidebar_position: 4
---

# hxbit/HXS Data (Save file payloads)

HxBit is a binary serialization and network synchronization library for the Haxe programming language. The corresponding file format, often with an `.hxs` extension, is designed for speed and efficiency, using a compact binary representation instead of a more verbose text-based format. This makes it ideal for applications requiring fast loading times and a small data footprint, such as games and networked applications.

Unlike standard Haxe serialization, which uses runtime type checking, HxBit employs macros to generate strictly-typed serialization code. This approach results in very fast I/O operations. The format also supports versioning by storing the schema of each class within the serialized data, which allows for robust handling of data structure changes over time, such as adding or removing fields.

## Format

An HXS file is a structured binary file composed of a header, class definitions, schema definitions, and the serialized object data.

### File Layout

| Size (bytes) | Name | Description | Struct |
|---|---|---|---|
| 4 | Header | File magic and version number. | [Header](#header) |
| Variable | Class Definitions | A list of all class types present in the file, terminated by a null marker. | [Class Definitions](#class-definitions) |
| Variable | Schema Section | Describes the structure of each class, prefixed by its total size. | [Schema Section](#schema-section) |
| Variable | Object Data | The actual instances of the serialized objects, starting with the root object. | [Object Data](#object-data) |

---

### Header

The header is a fixed-size block that identifies the file as being in the HxBit format and specifies its version.

| Size (bytes) | Name | Description | Struct |
|---|---|---|---|
| 3 | Magic | The ASCII string "HXS". | ASCII-encoded text |
| 1 | Version | The version of the HXS format, typically `0x01`. | Unsigned byte |

---

### Class Definitions

This section lists all the classes that are serialized in the file. It is a variable-length sequence of `Class Definition Entry` structures. The entire section is terminated by a single `VarInt` with a value of 0, which acts as a null marker for the class name.

#### Single Class Definition Entry

| Size (bytes) | Name | Description | Struct |
|---|---|---|---|
| Variable | Name | The fully qualified name of the class. | [String](#string) |
| 2 | CLID | A 2-byte unique identifier for the class. | Big-endian unsigned short |
| 4 | CRC32 | A CRC32 checksum of the class definition. | Little-endian unsigned int |

---

### Schema Section

The schema section provides a detailed description of the data structure for each class, including its fields and their types. This metadata is crucial for deserialization and version management.

| Size (bytes) | Name | Description | Struct |
|---|---|---|---|
| Variable | Schema Size | The total size of the following schema data in bytes. | [VarInt](#varint) |
| `Schema Size` | Schemas | A list of schema definitions for each class. | [Single Schema Entry](#single-schema-entry) |

#### Single Schema Entry

Each schema entry defines the structure of one class.

| Size (bytes) | Name | Description |
|---|---|---|
| Variable | UID | A unique identifier for this specific schema, encoded as a [VarInt](#varint). This UID is used in the [Object Data](#object-data) section to reference instances of this type. |
| Variable | CLID | The corresponding class identifier, matching a `CLID` from the [Class Definitions](#class-definitions) section. Encoded as a [VarInt](#varint). |
| Variable | Field Names List | A list of strings representing the names of the class fields. The list is prefixed by its `count+1` as a [VarInt](#varint). A `count+1` of 1 means an empty list. |
| Variable | Field Types List | A list of `PropType` structures defining the type of each field. The list is prefixed by its `count+1` as a [VarInt](#varint). A `count+1` of 1 means an empty list. |

---

### Object Data

This is the main content of the file, containing the serialized data for the root object and any other objects it references. The structure is hierarchical and begins with a reference to the root object. Object references are used to handle shared instances and circular dependencies efficiently.

An object is serialized as a reference followed by its payload if it's the first time it's encountered.

| Size (bytes) | Name | Description |
|---|---|---|
| Variable | Object Reference | A [VarInt](#varint) representing the object's UID. A value of 0 indicates a `null` reference. If the UID has been seen before, this is all that is written. |
| Variable | Object Payload | If this is the first time an object with this UID is being serialized, its reference is followed by its payload. The payload consists of the concatenated binary data of all its fields, serialized according to its [Schema](#single-schema-entry). |

---

## Core Data Types

The HXS format is built upon a set of core data types that are used to represent various kinds of information.

### VarInt

A variable-length integer designed to use fewer bytes for smaller numbers.

| Condition | Size (bytes) | Format |
|---|---|---|
| Value is between `0` and `127` | 1 | The value is stored directly as a single byte. |
| Value is `128` or greater | 5 | A marker byte `0x80` followed by the full 32-bit integer value in little-endian format. |

### String

A string is encoded with a length prefix. This structure allows for `null` strings to be represented compactly.

| Size (bytes) | Name | Description |
|---|---|---|
| Variable | Length+1 | The length of the string plus one, encoded as a [VarInt](#varint). A `VarInt` value of `0` indicates a `null` string. A value of `1` indicates an empty string (`""`). |
| `Length` | UTF-8 Data | The string data encoded in UTF-8. This part is only present if `Length+1` is greater than 1. |

### PropType (Property Type)

A `PropType` is a type descriptor that defines the type of a schema field. It consists of a `Kind` byte followed by an optional, type-dependent definition payload.

| Size (bytes) | Name | Description |
|---|---|---|
| 1 | Kind | An unsigned byte indicating the base type (e.g., Int, Float, String, Array, Object). A value of `0` indicates an untyped or null property. |
| Variable | Definition | An optional payload that provides more detail about the type. For example, for an `PArray`, the definition is another `PropType` that describes the type of elements in the array. For a `PObj` (Object), the definition describes the fields of the inline object structure. For simple types like `PInt`, this part is empty. |
