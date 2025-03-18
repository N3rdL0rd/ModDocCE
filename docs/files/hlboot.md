---
sidebar_position: 2
---

# HashLink bytecode (and hlboot.dat)

:::note

Based on the documentation at the excellent [hlbc wiki](https://github.com/Gui-Yom/hlbc/wiki/Bytecode-file-format), and improved with information from the development of [crashlink](https://github.com/N3rdL0rd/crashlink).

:::

When compiling Haxe code for HashLink, the compiler generates a single binary file. The extension of such file is
usually `.hl`, if the app is distributed with a HashLink vm, the bytecode file will be named `hlboot.dat` and will be
found automatically by the main vm executable.

The binary contains everything that's needed to run which is data, type definitions and code. HashLink will generate
native code for every function on startup (this is actually an AOT JIT if such term exists, everything gets compiled on
startup, not lazily like on the JVM).

The bytecode is entirely typed and contains definition for every possible type in the program (including primitives and
function types). The code is register based per function, a function declares some typed registers to be used by the
instructions.

There are constant pools for signed integers, double precision floating point numbers, strings and bytes.

The bytecode also optionally contains debug info like source file names, lines and variable assignments.

Because most of the values in the structures are of variable size, it is impossible to know the position of anything
without parsing the whole file beforehand.

## Structures

*var* is a [variable size integer](#variable-sized-integers).

### Main structure

| size (bytes/struct)                      | name          | description                                           |
|------------------------------------------|---------------|-------------------------------------------------------|
| 3                                        | `magic`       | "HLB"                                                 |
| 1                                        | `version`     |                                                       |
| *var*                                    | `flags`       | has debug info                                        |
| *var*                                    | `nints`       |                                                       |
| *var*                                    | `nfloats`     |                                                       |
| *var*                                    | `nstrings`    |                                                       |
| *var*                                    | `nbytes`      | if version >= 5                                       |
| *var*                                    | `ntypes`      |                                                       |
| *var*                                    | `nglobals`    |                                                       |
| *var*                                    | `nnatives`    |                                                       |
| *var*                                    | `nfunctions`  |                                                       |
| *var*                                    | `nconstants`  | if version >= 4                                       |
| *var*                                    | `entrypoint`  | findex                                                |
| 4 * `nints`                              | `ints`        | i32 constant pool                                     |
| 8 * `nfloats`                            | `floats`      | f64 constant pool                                     |
| [**strings**](#strings-block)            | `strings`     | string constant pool                                  |
| 4                                        | `bytes_size`  | amount to read for bytes data                         |
| `bytes_size`                             | `bytes_data`  | byte strings data                                     |
| *var* * `nbytes`                         | `bytes_pos`   | pos of byte strings in `bytes_data`                   |
| 4                                        | `ndebugfiles` | if has debug info, number of debug files entries      |
| [**strings**](#strings-block)            | `debugfiles`  | if has debug info                                     |
| `ntypes` * [**type**](#types)            | `types`       | types definitions                                     |
| *var* * `nglobals`                       | `globals`     | types of each globals                                 |
| `nnatives` * [**native**](#natives)      | `natives`     | Native functions to be loaded from external libraries |
| `nfunctions`* [**function**](#functions) | `functions`   | Function definitions                                  |
| `nconstants`* [**constant**](#constants) | `constants`   | Constant definitions                                  |

### Strings block

| size               | name            | description                           |
|--------------------|-----------------|---------------------------------------|
| 4                  | `strings_size`  | amount to read next                   |
| `strings_size`     | `strings_data`  | strings list, zero terminated strings |
| *var* * `nstrings` | `strings_sizes` | sizes of each string                  |

### Types

| size | name         | description                               |
|------|--------------|-------------------------------------------|
| 1    | `kind`       | [type kind](#type-kinds)                  |
| *?*  | `definition` | empty or some data based on the type kind |

#### Type kinds

| kind | name                  | description                                                 |
|------|-----------------------|-------------------------------------------------------------|
| 0    | void                  | *no data*                                                   |
| 1    | u8                    | *no data*                                                   |
| 2    | u16                   | *no data*                                                   |
| 3    | i32                   | *no data*                                                   |
| 4    | i64                   | *no data*                                                   |
| 5    | f32                   | *no data*                                                   |
| 6    | f64                   | *no data*                                                   |
| 7    | bool                  | *no data*                                                   |
| 8    | bytes                 | *no data*                                                   |
| 9    | Dyn                   | *no data*, dynamic type, can be anything                    |
| 10   | [Fun](#fun)           | function type / signature                                   |
| 11   | [Obj](#obj)           | object type (haxe class)                                    |
| 12   | Array                 | *no data*, arrays are dynamic                               |
| 13   | Type                  | *no data*, the Type type, for reflection                    |
| 14   | [Ref](#ref)           | reference                                                   |
| 15   | [Virtual](#virtual)   | like an anonymous class                                     |
| 16   | DynObj                | *no data*, like Dyn but it is an object                     |
| 17   | [Abstract](#abstract) | abstract class                                              |
| 18   | [Enum](#enum)         | enum with its variants                                      |
| 19   | [Null](#null)         | Can possibly be null type, types aren't nullable by default |
| 20   | [Method](#method)     | like Fun, but different                                     |
| 21   | [Struct](#struct)     | like Obj, but different                                     |
| 22   | [Packed](#packed)     | Packs the inner structure                                   |

#### Fun

| size            | name    | description                          |
|-----------------|---------|--------------------------------------|
| *var*           | `nargs` |                                      |
| *var* * `nargs` | `args`  | type references of the function args |
| *var*           | `ret`   | return type                          |

##### Function names

As you can see, a function has no name. Its name can be inferred from where it is used though. They are bound as class
methods (protos) or into class fields (bindings), so you can name them from that (after parsing the whole thing).

#### Obj

| size                                  | name        | description                                                      |
|---------------------------------------|-------------|------------------------------------------------------------------|
| *var*                                 | `name`      | string ref                                                       |
| *var*                                 | `super`     | type ref, supertype (can be &lt; 0, -> no supertype)                |
| *var*                                 | `global`    | global ref, global representing the static fields                |
| *var*                                 | `nfields`   | number of fields                                                 |
| *var*                                 | `nprotos`   | number of methods                                                |
| *var*                                 | `nbindings` | number of field bindings                                         |
| `nfields` * [**field**](#field)       | `fields`    | class fields                                                     |
| `nprotos` * [**proto**](#proto)       | `protos`    | class methods                                                    |
| `nbindings` * [**binding**](#binding) | `protos`    | bindings between fields and functions (usually static functions) |

##### Field

Field definition for an Obj or a Virtual

| size  | name   | description           |
|-------|--------|-----------------------|
| *var* | `name` | string ref, name      |
| *var* | `type` | type ref of the field |

###### Field references

Fields references are positive numbers only valid in a given type hierarchy. They are the field index in an array of all
the fields in the hierarchy in order. Example :

```haxe
class A {
  var a: Int;
}

class B extends A {
  var b: Int;
}
```

In this example, the index of the field `b` is 1, while the index of `a` is 0.

A consequence of this system is that you need to traverse the entire type hierarchy to gather all fields to know their indexes.

##### Proto

A class instance method.

| size  | name     | description           |
|-------|----------|-----------------------|
| *var* | `name`   | string ref, name      |
| *var* | `findex` | global function index |
| *var* | `pindex` | ?                     |

##### Binding

Binds a field to a function, usually for static functions or constructors.

| size  | name     | description           |
|-------|----------|-----------------------|
| *var* | `field`  | field ref             |
| *var* | `findex` | global function index |

See [field references](#field-references).

#### Ref

The type of reference : Ref&lt;T>. Same structure as other wrapper types.

| size  | name   | description               |
|-------|--------|---------------------------|
| *var* | `type` | type behind the reference |

#### Virtual

An anonymous data class with fields.

| size                            | name      | description      |
|---------------------------------|-----------|------------------|
| *var*                           | `nfields` | number of fields |
| `nfields` * [**field**](#field) | `fields`  | the fields       |

#### Abstract

| size  | name   | description |
|-------|--------|-------------|
| *var* | `name` | string ref  |

#### Enum

| size  | name          | description                     |
|-------|---------------|---------------------------------|
| *var* | `name`        | string ref                      |
| *var* | `global`      |                                 |
| *var* | `nconstructs` | number of constructs (variants) |

##### Construct

| size              | name      | description               |
|-------------------|-----------|---------------------------|
| *var*             | `name`    | string ref                |
| *var*             | `nparams` | number of parameters      |
| *var* * `nparams` | `params`  | type ref, parameters type |

#### Null

Same structure as other wrapper types.

| size  | name   | description |
|-------|--------|-------------|
| *var* | `type` | inner type  |

#### Method

Same structure as [Fun](#fun).

#### Struct

Same structure as [Obj](#obj).

#### Packed

Same structure as other wrapper types.

| size  | name   | description |
|-------|--------|-------------|
| *var* | `type` | inner type  |

### Natives

| size  | name     | description               |
|-------|----------|---------------------------|
| *var* | `lib`    | string ref, lib name      |
| *var* | `name`   | string ref, function name |
| *var* | `type`   | type ref, function type   |
| *var* | `findex` | global function index     |

`lib` is the name of the external library to load. An exception is `std` which points directly to the standard library
present in HashLink. If `lib` contains a `?`, it means the library is optional and can possibly not be present.

### Functions

| size                            | name        | description                                                                     |
|---------------------------------|-------------|---------------------------------------------------------------------------------|
| *var*                           | `type`      | type ref, function type                                                         |
| *var*                           | `findex`    | global function index                                                           |
| *var*                           | `nregs`     | number of registers                                                             |
| *var*                           | `nops`      | number of instructions                                                          |
| *var* * `nregs`                 | `regs`      | registers types                                                                 |
| `nops` * [**opcode**](#opcodes) | `ops`       | instructions                                                                    |
| *?* * `nops`                    | `debuginfo` | if has debug info, complicated encoding for file/line info for each instruction |
| *var*                           | `nassigns`  | if has debug info && version >= 3                                               |
| 2 **var** `nassigns`          | `assigns`   | tuples (variable name ref, opcode number)                                       |

### Opcodes

An opcode consists of a variable size integer for the opcode index and a piece of data for each argument, determined by the argument type. Here is a list of all opcodes:

Basic Operations:

- Mov: Copy value from src into dst register (dst = src)
- Int: Load i32 constant from pool into dst (dst = @ptr)
- Float: Load f64 constant from pool into dst (dst = @ptr)
- Bool: Set boolean value in dst (dst = true/false)
- Bytes: Load byte array from constant pool into dst (dst = @ptr)
- String: Load string from constant pool into dst (dst = @ptr)
- Null: Set dst register to null (dst = null)

Arithmetic:

- Add: Add two numbers (dst = a + b)
- Sub: Subtract two numbers (dst = a - b)
- Mul: Multiply two numbers (dst = a * b)
- SDiv: Signed division (dst = a / b)
- UDiv: Unsigned division (dst = a / b)
- SMod: Signed modulo (dst = a % b)
- UMod: Unsigned modulo (dst = a % b)

Bitwise:

- Shl: Left shift (dst = a &lt;&lt; b)
- SShr: Signed right shift (dst = a >> b)
- UShr: Unsigned right shift (dst = a >>> b)
- And: Bitwise AND (dst = a & b)
- Or: Bitwise OR (dst = a | b)
- Xor: Bitwise XOR (dst = a ^ b)
- Neg: Negate value (dst = -src)
- Not: Boolean NOT (dst = !src)

Increment/Decrement:

- Incr: Increment value (dst++)
- Decr: Decrement value (dst--)

Function Calls:

- Call0: Call function with no args (dst = fun())
- Call1: Call function with 1 arg (dst = fun(arg0))
- Call2: Call function with 2 args (dst = fun(arg0, arg1))
- Call3: Call function with 3 args (dst = fun(arg0, arg1, arg2))
- Call4: Call function with 4 args (dst = fun(arg0, arg1, arg2, arg3))
- CallN: Call function with N args (dst = fun(args...))
- CallMethod: Call method with N args (dst = obj.field(args...))
- CallThis: Call this method with N args (dst = this.field(args...))
- CallClosure: Call closure with N args (dst = fun(args...))

Closures:

- StaticClosure: Create closure from function (dst = fun)
- InstanceClosure: Create closure from object method (dst = obj.fun)
- VirtualClosure: Create closure from object field (dst = obj.field)

Global Variables:

- GetGlobal: Get global value (dst = @global)
- SetGlobal: Set global value (@global = src)

Fields:

- Field: Get object field (dst = obj.field)  
- SetField: Set object field (obj.field = src)
- GetThis: Get this field (dst = this.field)
- SetThis: Set this field (this.field = src)
- DynGet: Get dynamic field (dst = obj[field])
- DynSet: Set dynamic field (obj[field] = src)

Control Flow:

- JTrue: Jump if true (if cond jump by offset)
- JFalse: Jump if false (if !cond jump by offset)
- JNull: Jump if null (if reg == null jump by offset)
- JNotNull: Jump if not null (if reg != null jump by offset)
- JSLt/JSGte/JSGt/JSLte: Signed comparison jumps
- JULt/JUGte: Unsigned comparison jumps
- JNotLt/JNotGte: Negated comparison jumps
- JEq: Jump if equal (if a == b jump by offset)
- JNotEq: Jump if not equal (if a != b jump by offset)
- JAlways: Unconditional jump
- Label: Target for backward jumps (loops)
- Switch: Multi-way branch based on integer value

Type Conversions:

- ToDyn: Convert to dynamic type (dst = (dyn)src)
- ToSFloat: Convert to signed float (dst = (float)src)
- ToUFloat: Convert to unsigned float (dst = (float)src)
- ToInt: Convert to int (dst = (int)src)
- SafeCast: Safe type cast with runtime check
- UnsafeCast: Unchecked type cast
- ToVirtual: Convert to virtual type

Exception Handling:

- Ret: Return from function (return ret)
- Throw: Throw exception
- Rethrow: Rethrow exception
- Trap: Setup try-catch block
- EndTrap: End try-catch block
- NullCheck: Throw if null (if reg == null throw)

Memory Operations:

- GetI8: Read i8 from bytes (dst = bytes[index])
- GetI16: Read i16 from bytes (dst = bytes[index])
- GetMem: Read from memory (dst = bytes[index])
- GetArray: Get array element (dst = array[index])
- SetI8: Write i8 to bytes (bytes[index] = src)
- SetI16: Write i16 to bytes (bytes[index] = src)
- SetMem: Write to memory (bytes[index] = src)
- SetArray: Set array element (array[index] = src)

Objects:

- New: Allocate new object (dst = new typeof(dst))
- ArraySize: Get array length (dst = len(array))
- Type: Get type object (dst = type ty)
- GetType: Get value's type (dst = typeof src)
- GetTID: Get type ID (dst = typeof src)

References:

- Ref: Create reference (dst = &src)
- Unref: Read reference (dst = *src)
- Setref: Write reference (*dst = src)
- RefData: Get reference data
- RefOffset: Get reference with offset

Enums:

- MakeEnum: Create enum variant (dst = construct(args...))
- EnumAlloc: Create enum with defaults (dst = construct())
- EnumIndex: Get enum tag (dst = variant of value)
- EnumField: Get enum field (dst = (value as construct).field)
- SetEnumField: Set enum field (value.field = src)

Other:

- Assert: Debug break
- Nop: No operation
- Prefetch: CPU memory prefetch hint
- Asm: Inline x86 assembly

Opcodes can have one of the following argument types:

- Reg: Register index (VarInt)
- Regs: Register indexes (VarInt for count, followed by count VarInts)
- RefInt: i32 constant index (VarInt)
- RefFloat: f64 constant index (VarInt)
- InlineBool: Boolean value - 0 or 1 (VarInt)
- RefBytes: Byte array constant index (VarInt)
- RefString: String constant index (VarInt)
- RefFun: Function index (VarInt)
- RefField: Field index (VarInt)
- RefGlobal: Global index (VarInt)
- JumpOffset: Jump offset (VarInt)
- JumpOffsets: Jump offsets (VarInt for count, followed by count VarInts)
- RefType: Type index (VarInt)
- RefEnumConstant: Enum constant index (VarInt)
- RefEnumConstruct: Enum construct index (VarInt)
- InlineInt: Inline integer value (VarInt)

#### Jump offsets

Jump offsets counts the number of instructions to jump (not the bytes). They may be negative for a backward jump, in
this case, they always target a `Label` instruction.

### Constants

Constants are used to initialize globals without code.

| size  | name      | description          |
|-------|-----------|----------------------|
| *var* | `global`  | global to initialize |
| *var* | `nfields` | number of fields     |
| *var* | `fields`  | field initializers   |

## About bytecode data types

### Variable sized integers

Since much of the linking in the bytecode happens with integer indexes, most of the integers are encoded in a variable
byte size format spanning 1, 2 or 4 bytes (*var*). This was probably done to save space. You can find an example on how
to
decode one [here](https://github.com/Gui-Yom/hlbc/blob/71212fa56cf52e4688e0468b8340a9e7f4bb6d6d/src/deser.rs#L41) and to
encode one [here](https://github.com/Gui-Yom/hlbc/blob/71212fa56cf52e4688e0468b8340a9e7f4bb6d6d/src/ser.rs#L33).

### Function indexes

A function reference is a function index (findex) but unlike other indexes it does not reference a function in the pool
directly, as natives and functions share the findex space. We need to parse the whole file to map findexes to regular
function or native indexes.

## See also

- [hlbc](https://github.com/Gui-Yom/hlbc)
- [crashlink](https://github.com/N3rdL0rd/crashlink)
- [Using crashlink](/tutorials/crashlink)
