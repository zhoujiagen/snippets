# MySQL Data Type

Convention:

(1) **M**

- For integer types, M indicates the maximum display width.
- For floating-point and fixed-point types, M is the total number of digits that can be stored (the precision).
- For string types, M is the maximum length.
- The maximum permissible value of M depends on the data type.

(2) **D**

- D applies to floating-point and fixed-point types and indicates the number of digits following the decimal point (the scale).
- The maximum possible value is 30, but should be no greater than M−2.

(3) **fsp**

- fsp applies to the TIME, DATETIME, and TIMESTAMP types and represents fractional seconds precision; that is, the number of digits following the decimal point for fractional parts of seconds.
- The fsp value, if given, must be in the range 0 to 6. A value of 0 signifies that there is no fractional part. If omitted, the default precision is 0.

(4) **[]**

- Square brackets ([ and ]) indicate optional parts of type definitions.


## Numeric Data Types

|<div style="width: 300px">Syntax</div>|Description|
|:---|:---|
|`BIT[(M)]`|A bit-value type. M indicates the number of bits per value, from 1 to 64. The default is 1 if M is omitted.|
|`TINYINT[(M)] [UNSIGNED] [ZEROFILL]`|A very small integer. The signed range is -128 to 127. The unsigned range is 0 to 255.|
|`BOOL, BOOLEAN`|These types are synonyms for TINYINT(1). A value of zero is considered false. Nonzero values are considered true.|
|`SMALLINT[(M)] [UNSIGNED] [ZEROFILL]`|A small integer. The signed range is -32768 to 32767. The unsigned range is 0 to 65535.|
|`MEDIUMINT[(M)] [UNSIGNED] [ZEROFILL]`|A medium-sized integer. The signed range is -8388608 to 8388607. The unsigned range is 0 to 16777215.|
|`INT[(M)] [UNSIGNED] [ZEROFILL]`|A normal-size integer. The signed range is -2147483648 to 2147483647. The unsigned range is 0 to 4294967295.|
|`INTEGER[(M)] [UNSIGNED] [ZEROFILL]`|This type is a synonym for INT.|
|`BIGINT[(M)] [UNSIGNED] [ZEROFILL]`|A large integer. The signed range is -9223372036854775808 to 9223372036854775807. The unsigned range is 0 to 18446744073709551615.<br/>SERIAL is an alias for BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE.|
|`DECIMAL[(M[,D])] [UNSIGNED] [ZEROFILL]`|A packed “exact” fixed-point number. M is the total number of digits (the precision) and D is the number of digits after the decimal point (the scale). The decimal point and (for negative numbers) the - sign are not counted in M. If D is 0, values have no decimal point or fractional part. The maximum number of digits (M) for DECIMAL is 65. The maximum number of supported decimals (D) is 30. If D is omitted, the default is 0. If M is omitted, the default is 10.|
|`DEC[(M[,D])] [UNSIGNED] [ZEROFILL]`<br/>`NUMERIC[(M[,D])] [UNSIGNED] [ZEROFILL]`<br/>`FIXED[(M[,D])] [UNSIGNED] [ZEROFILL]`|These types are synonyms for DECIMAL.|
|`FLOAT[(M,D)] [UNSIGNED] [ZEROFILL]`|A small (single-precision) floating-point number. Permissible values are -3.402823466E+38 to -1.175494351E-38, 0, and 1.175494351E-38 to 3.402823466E+38. These are the theoretical limits, based on the IEEE standard. The actual range might be slightly smaller depending on your hardware or operating system.<br/>FLOAT(M,D) is a nonstandard MySQL extension.|
|`FLOAT(p) [UNSIGNED] [ZEROFILL]`|A floating-point number. p represents the precision in bits, but MySQL uses this value only to determine whether to use FLOAT or DOUBLE for the resulting data type. If p is from 0 to 24, the data type becomes FLOAT with no M or D values. If p is from 25 to 53, the data type becomes DOUBLE with no M or D values. The range of the resulting column is the same as for the single-precision FLOAT or double-precision DOUBLE data types described earlier in this section.<br/>FLOAT(p) syntax is provided for ODBC compatibility.|
|`DOUBLE[(M,D)] [UNSIGNED] [ZEROFILL]`|A normal-size (double-precision) floating-point number. Permissible values are -1.7976931348623157E+308 to -2.2250738585072014E-308, 0, and 2.2250738585072014E-308 to 1.7976931348623157E+308. These are the theoretical limits, based on the IEEE standard. The actual range might be slightly smaller depending on your hardware or operating system.<br/>M is the total number of digits and D is the number of digits following the decimal point. If M and D are omitted, values are stored to the limits permitted by the hardware. A double-precision floating-point number is accurate to approximately 15 decimal places.<br/>DOUBLE(M,D) is a nonstandard MySQL extension.|
|`DOUBLE PRECISION[(M,D)] [UNSIGNED] [ZEROFILL]`<br/>`REAL[(M,D)] [UNSIGNED] [ZEROFILL]`|These types are synonyms for DOUBLE. Exception: If the REAL_AS_FLOAT SQL mode is enabled, REAL is a synonym for FLOAT rather than DOUBLE.|

## Date and Time Data Types

|Syntax|Description|
|:---|:---|
|`DATE`|A date. The supported range is '1000-01-01' to '9999-12-31'. MySQL displays DATE values in 'YYYY-MM-DD' format, but permits assignment of values to DATE columns using either strings or numbers.|
|`DATETIME[(fsp)]`|A date and time combination. The supported range is '1000-01-01 00:00:00.000000' to '9999-12-31 23:59:59.999999'. MySQL displays DATETIME values in 'YYYY-MM-DD hh:mm:ss[.fraction]' format, but permits assignment of values to DATETIME columns using either strings or numbers.|
|`TIMESTAMP[(fsp)]`|A timestamp. The range is '1970-01-01 00:00:01.000000' UTC to '2038-01-19 03:14:07.999999' UTC. TIMESTAMP values are stored as the number of seconds since the epoch ('1970-01-01 00:00:00' UTC). A TIMESTAMP cannot represent the value '1970-01-01 00:00:00' because that is equivalent to 0 seconds from the epoch and the value 0 is reserved for representing '0000-00-00 00:00:00', the “zero” TIMESTAMP value.|
|`TIME[(fsp)]`|A time. The range is '-838:59:59.000000' to '838:59:59.000000'. MySQL displays TIME values in 'hh:mm:ss[.fraction]' format, but permits assignment of values to TIME columns using either strings or numbers.|
|`YEAR[(4)]`|A year in 4-digit format. MySQL displays YEAR values in YYYY format, but permits assignment of values to YEAR columns using either strings or numbers. Values display as 1901 to 2155, or 0000.|

## String Data Types

|<div style="width: 300px">Syntax</div>|Description|
|:---|:---|
|`[NATIONAL] CHAR[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]`|A fixed-length string that is always right-padded with spaces to the specified length when stored. M represents the column length in characters. The range of M is 0 to 255. If M is omitted, the length is 1.<br/>CHAR is shorthand for CHARACTER.<br/>The CHAR BYTE data type is an alias for the BINARY data type.|
|`[NATIONAL] VARCHAR(M) [CHARACTER SET charset_name] [COLLATE collation_name]`|A variable-length string. M represents the maximum column length in characters. The range of M is 0 to 65,535. The effective maximum length of a VARCHAR is subject to the maximum row size (65,535 bytes, which is shared among all columns) and the character set used.<br/>MySQL stores VARCHAR values as a 1-byte or 2-byte length prefix plus data. The length prefix indicates the number of bytes in the value. A VARCHAR column uses one length byte if values require no more than 255 bytes, two length bytes if values may require more than 255 bytes.|
|`BINARY[(M)]`|The BINARY type is similar to the CHAR type, but stores binary byte strings rather than nonbinary character strings. An optional length M represents the column length in bytes. If omitted, M defaults to 1.|
|`VARBINARY(M)`|The VARBINARY type is similar to the VARCHAR type, but stores binary byte strings rather than nonbinary character strings. M represents the maximum column length in bytes.|
|`TINYBLOB`|A BLOB column with a maximum length of 255 bytes. Each TINYBLOB value is stored using a 1-byte length prefix that indicates the number of bytes in the value.|
|`TINYTEXT [CHARACTER SET charset_name] [COLLATE collation_name]`|A TEXT column with a maximum length of 255 characters. The effective maximum length is less if the value contains multibyte characters. Each TINYTEXT value is stored using a 1-byte length prefix that indicates the number of bytes in the value.|
|`BLOB[(M)]`|A BLOB column with a maximum length of 65,535 bytes. Each BLOB value is stored using a 2-byte length prefix that indicates the number of bytes in the value.<br/>An optional length M can be given for this type. If this is done, MySQL creates the column as the smallest BLOB type large enough to hold values M bytes long.|
|`TEXT[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]`|A TEXT column with a maximum length of 65,535 characters. The effective maximum length is less if the value contains multibyte characters. Each TEXT value is stored using a 2-byte length prefix that indicates the number of bytes in the value. <br/>An optional length M can be given for this type. If this is done, MySQL creates the column as the smallest TEXT type large enough to hold values M characters long.|
|`MEDIUMBLOB`|A BLOB column with a maximum length of 16,777,215 bytes. Each MEDIUMBLOB value is stored using a 3-byte length prefix that indicates the number of bytes in the value.|
|`MEDIUMTEXT [CHARACTER SET charset_name] [COLLATE collation_name]`|A TEXT column with a maximum length of 16,777,215 characters. The effective maximum length is less if the value contains multibyte characters. Each MEDIUMTEXT value is stored using a 3-byte length prefix that indicates the number of bytes in the value.|
|`LONGBLOB`|A BLOB column with a maximum length of 4,294,967,295 or 4GB bytes. The effective maximum length of LONGBLOB columns depends on the configured maximum packet size in the client/server protocol and available memory. Each LONGBLOB value is stored using a 4-byte length prefix that indicates the number of bytes in the value.|
|`LONGTEXT [CHARACTER SET charset_name] [COLLATE collation_name]`|A TEXT column with a maximum length of 4,294,967,295 or 4GB characters. The effective maximum length is less if the value contains multibyte characters. The effective maximum length of LONGTEXT columns also depends on the configured maximum packet size in the client/server protocol and available memory. Each LONGTEXT value is stored using a 4-byte length prefix that indicates the number of bytes in the value.|
|`ENUM('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]`|An enumeration. A string object that can have only one value, chosen from the list of values 'value1', 'value2', ..., NULL or the special '' error value. ENUM values are represented internally as integers.<br/>An ENUM column can have a maximum of 65,535 distinct elements.<br/>A table can have no more than 255 unique element list definitions among its ENUM and SET columns considered as a group.|
|`SET('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]`|A set. A string object that can have zero or more values, each of which must be chosen from the list of values 'value1', 'value2', ... SET values are represented internally as integers.<br/>A SET column can have a maximum of 64 distinct members. A table can have no more than 255 unique element list definitions among its ENUM and SET columns considered as a group.|

## Spatial Data Types

|Type|Description|
|:---|:---|
|GEOMETRY||
|POINT||
|LINESTRING||
|POLYGON||
|MULTIPOINT||
|MULTILINESTRING||
|MULTIPOLYGON||
|GEOMETRYCOLLECTION||

## The JSON Data Type

> As of MySQL 5.7.8, MySQL supports a native JSON data type defined by RFC 7159 that enables efficient access to data in JSON (JavaScript Object Notation) documents.

The space required to store a JSON document is roughly the same as for LONGBLOB or LONGTEXT.

It is important to keep in mind that the size of any JSON document stored in a JSON column is limited to the value of the max_allowed_packet system variable.

A JSON column cannot have a non-NULL default value.
