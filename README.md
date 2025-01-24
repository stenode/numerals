# numerals

## Summary

Numerals is a [profile]() that uses certain conventions to identify different numeric types without breaking JSON compatibility. 

## Motivation

JSON has support for a single type to represent all numerals i.e [Number](https://datatracker.ietf.org/doc/html/rfc8259#section-6). However, it often falls short when the data is mapped to corresponding types in programming languages. This profile is an attempt to provide more clarity by adopting certain conventions to represent different data types. 

## Design

JSON `number` follows the format: `[ minus ] int [ frac ] [ exp ]`. The below design uses the `.` separator and exponent characters `e`/`E` to support more numeric data types. The choice of data types and its corresponding definitions follows the [Postgres specification](https://www.postgresql.org/docs/current/datatype-numeric.html).

| Data Type   | Size (bytes)| Description                     | JSON Schema | OpenAPI | GraphQL | Protobuf | Numerals Format                      | Examples              |
|-------------|-------------|---------------------------------|-------------|---------|---------|----------|--------------------------------------|-----------------------|
| smallint    | 2           | small-range integer             | integer     | int16   | Int     | int32    | `[ minus ] int`                      | -4, 1                 |
| integer     | 4           | typical choice for integer      | integer     | int32   | Int     | int32    | `[ minus ] int exp(with 'e')`        | -4e2, 47483647        |
| bigint      | 8           | large-range integer             | integer     | int64   | Int     | int64    | `[ minus ] int exp(with 'E')`        | 4E5, 51241228399      |
| float(real) | 4           | variable-precision, inexact     | number      | float   | Float   | float    | `[ minus ] int frac exp(with 'e')`   | 4.12e-2, 2.314e0      |
| double      | 8           | variable-precision, inexact     | number      | double  | Float   | double   | `[ minus ] int frac exp(with 'E')`   | 4.5E2, 1.112E0        |
| decimal     | ?           | user-specified precision, exact | number      | decimal | Float   | double   | `[ minus ] int frac exp(with 'E00')` | -4.19920, 0.199e-0012 |
| numeric     | ?           | user-specified precision, exact | number      | decimal | Float   | double   | `[ minus ] int frac exp(with 'E00')` | 4.513930, 0.199E0012  |


### Guidelines

1. Default data type for a value is given below:
    - `smallint` : If it doesn't have a fractional component and if it has the range: -32768 to +32767
    - `integer`  : If it doesn't have a fractional component and if it has the range: -2147483648 to +2147483647
    - `bigint`   : If it doesn't have a fractional component and if it has the range: -9223372036854775808 to +9223372036854775807
    - `decimal`  : If it has a fractional component.
2. The exponent character in `decimal`/`numeric` MUST be padded with two `00` digit for specifying exact precision numbers.
3. Protobuf supports more data types for the below given types:
    - `integer`  : unint32, sint32, fixed32, sfixed32
    - `bigint`   : unint64, sint64, fixed64, sfixed64
