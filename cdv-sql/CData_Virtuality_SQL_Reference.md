# CData Virtuality (Data Virtuality) SQL Reference Guide

This reference guide provides comprehensive documentation for building and converting SQL statements in CData Virtuality (formerly Data Virtuality).

---

## Table of Contents

1. [Core SQL Syntax](#core-sql-syntax)
2. [Identifiers](#identifiers)
3. [Data Types](#data-types)
4. [Type Conversions](#type-conversions)
5. [Scalar Functions](#scalar-functions)
   - [Array Functions](#array-functions)
   - [Choice Functions](#choice-functions)
   - [Date and Time Functions](#date-and-time-functions)
   - [JSON Functions](#json-functions)
   - [Numeric Functions](#numeric-functions)
   - [Security Functions](#security-functions)
   - [String Functions](#string-functions)
   - [System Functions](#system-functions)
   - [XML Functions](#xml-functions)
6. [Virtual Procedures](#virtual-procedures)

---

## Core SQL Syntax

### Key Differences from Standard SQL

- **Statement Termination**: All Data Virtuality SQL commands end with **double semicolon (;;)**
  - Commands within a procedure end with a single semicolon (;)
  - The procedure itself ends with double semicolon (;;)

- **SQL Compatibility**: Provides nearly all SQL-92 DML functionality with SQL-99+ features
- **SELECT statements** are mostly PostgreSQL compatible

### Important Notes

- Only use functions explicitly listed in the Scalar Functions section
- Functions from other SQL variants must be rewritten using available DV functions
- Implicit type conversions happen automatically in criteria and expressions
- All queries are processed in the context of a Virtual Database (VDB)

---

## Identifiers

### Format

**Fully-Qualified Names:**
```
TABLE:  <schema_name>.<table_spec>
COLUMN: <schema_name>.<table_spec>.<column_name>
```

### Syntax Rules

- Consist of alphanumeric characters and underscore (_)
- Must begin with an alphabetic character
- Any Unicode character may be used
- Identifiers in double quotes can contain any characters
- Double-quote character escaped with additional double quote: `"some "" id"`
- **Case-insensitive** (even when quoted)
- Columns, schemas, and aliases cannot contain a dot
- Table specifications can contain dots (multi-part names)

### Examples

Valid table identifiers:
```sql
MySchema.Portfolios
"MySchema.Portfolios"
MySchema.MyCatalog.dbo.Authors
```

Valid column identifiers:
```sql
MySchema.Portfolios.portfolioID
"MySchema.Portfolios"."portfolioID"
MySchema.MyCatalog.dbo.Authors.lastName
```

---

## Data Types

### Core Data Types

| Type | Description | Java Class | JDBC Type | ODBC Type |
|------|-------------|------------|-----------|-----------|
| **string** or **varchar** | Variable length character string (max 4000) | java.lang.String | VARCHAR | VARCHAR |
| **varbinary** | Variable length binary string (max 8192) | byte[] | VARBINARY | VARBINARY |
| **char** | Single Unicode character | java.lang.Character | CHAR | CHAR |
| **boolean** | TRUE, FALSE, or NULL | java.lang.Boolean | BIT | SMALLINT |
| **byte** or **tinyint** | Signed 8-bit integer | java.lang.Byte | TINYINT | SMALLINT |
| **short** or **smallint** | Signed 16-bit integer | java.lang.Short | SMALLINT | SMALLINT |
| **integer** or **serial** | Signed 32-bit integer | java.lang.Integer | INTEGER | INTEGER |
| **long** or **bigint** | Signed 64-bit integer | java.lang.Long | BIGINT | NUMERIC |
| **biginteger** | Arbitrary precision integer (up to 1000 digits) | java.lang.BigInteger | NUMERIC | NUMERIC |
| **float** or **real** | 32-bit IEEE 754 floating-point | java.lang.Float | REAL | FLOAT |
| **double** | 64-bit IEEE 754 floating-point | java.lang.Double | DOUBLE | DOUBLE |
| **bigdecimal** or **decimal** | Arbitrary precision decimal (up to 1000 digits) | java.math.BigDecimal | NUMERIC | NUMERIC |
| **date** | Date (year, month, day) | java.sql.Date | DATE | DATE |
| **time** | Time (hours, minutes, seconds, milliseconds) | java.sql.Time | TIME | TIME |
| **timestamp** | Date and time | java.sql.Timestamp | TIMESTAMP | TIMESTAMP |
| **object** | Any Java object (must implement Serializable) | Any | JAVA_OBJECT | VARCHAR |
| **blob** | Binary large object | java.sql.Blob | BLOB | VARCHAR |
| **clob** | Character large object | java.sql.Clob | CLOB | VARCHAR |
| **xml** | XML document | java.sql.SQLXML | JAVA_OBJECT | VARCHAR |

---

## Type Conversions

### Implicit vs Explicit Conversions

- **Implicit**: Automatic conversion in criteria and expressions
- **Explicit**: Use `CONVERT()` or `CAST()` functions

### Conversion Functions

```sql
CONVERT(x, type)    -- JDBC/ODBC syntax
CAST(x AS type)     -- Standard SQL syntax
```

### Type Conversion Matrix

| Source Type | Valid Implicit Target Types | Valid Explicit Target Types |
|-------------|----------------------------|----------------------------|
| string | clob | char, boolean, byte, short, integer, long, biginteger, float, double, bigdecimal, xml |
| char | string | - |
| boolean | string, byte, short, integer, long, biginteger, float, double, bigdecimal | - |
| byte | string, short, integer, long, biginteger, float, double, bigdecimal | boolean |
| short | string, integer, long, biginteger, float, double, bigdecimal | boolean, byte |
| integer | string, long, biginteger, double, bigdecimal | boolean, byte, short, float |
| long | string, biginteger, bigdecimal | boolean, byte, short, integer, float, double |
| biginteger | string, bigdecimal | boolean, byte, short, integer, long, float, double |
| bigdecimal | string | boolean, byte, short, integer, long, biginteger, float, double |
| date | string, timestamp | - |
| time | string, timestamp | - |
| timestamp | string | date, time |
| clob | string | - |

### Special Conversions

**String to Boolean:**
- 'false' → FALSE
- 'unknown' → NULL
- other → TRUE

**Numeric to Boolean:**
- 0 → FALSE
- other → TRUE

**Date/Time Literals:**

```sql
{d 'yyyy-mm-dd'}                      -- DATE
{t 'hh:mm:ss'}                        -- TIME
{ts 'yyyy-mm-dd hh:mm:ss.[fff...]'}   -- TIMESTAMP
```

---

## Scalar Functions

### Array Functions

**Declaring Arrays:**
```sql
DECLARE STRING[] my_array = ARRAY('John', 'Paul', 'George', 'Stuart');
```

**Note**: Array indexes start at position 1 (not 0)

| Function | Description | Syntax | Return Type |
|----------|-------------|--------|-------------|
| **ARRAY_GET** | Get element at index | `ARRAY_GET(array, index)` | object |
| **ARRAY_LENGTH** | Get array length | `ARRAY_LENGTH(array)` | integer |
| **ARRAY_ADD** | Add element(s) to array | `ARRAY_ADD(array, element_or_array)` | object |
| **ARRAY_SUB** | Get subarray | `ARRAY_SUB(array, start, count)` | object |
| **ARRAY_IN** | Find element index | `ARRAY_IN(array, value)` | integer (0 if not found) |
| **ARRAY_LIKE** | LIKE search | `ARRAY_LIKE(array, like_pattern)` | boolean |
| **ARRAY_LIKE_REGEX** | Regex search | `ARRAY_LIKE_REGEX(array, regex)` | boolean |

---

### Choice Functions

| Function | Description | Constraints |
|----------|-------------|-------------|
| **COALESCE(x, y+)** | Returns first non-null parameter | All parameters must be compatible types |
| **IFNULL(x, y)** / **NVL(x, y)** | If x is null, return y; else return x | x and y must be same type |
| **NULLIF(param1, param2)** | If param1 = param2, return NULL; else return param1 | Parameters must be compatible |

---

### Date and Time Functions

**Pattern Format**: Uses `java.text.SimpleDateFormat` conventions

#### Key Functions

| Function | Description | Example |
|----------|-------------|---------|
| **CURDATE()** | Current date | Returns DATE |
| **CURTIME()** | Current time | Returns TIME |
| **NOW()** | Current timestamp | Returns TIMESTAMP |
| **DATE_TRUNC('precision', x)** | Truncate to precision | `DATE_TRUNC('MONTH', startDate)` |
| **DAYNAME(x)** | Day name | Returns string |
| **DAYOFMONTH(x)** | Day of month | Returns integer (1-31) |
| **DAYOFWEEK(x)** | Day of week | Returns integer (Sunday=1) |
| **DAYOFYEAR(x)** | Julian day number | Returns integer |
| **EXTRACT(field FROM x)** | Extract field | `EXTRACT(YEAR FROM date)` |
| **FORMATDATE(x, y)** | Format date | `FORMATDATE(date, 'yyyy-MM-dd')` |
| **FORMATTIME(x, y)** | Format time | `FORMATTIME(time, 'HH:mm:ss')` |
| **FORMATTIMESTAMP(x, y)** | Format timestamp | `FORMATTIMESTAMP(ts, 'yyyy-MM-dd HH:mm:ss')` |
| **FROM_UNIXTIME(seconds)** | Unix timestamp to TIMESTAMP | Max value: 2147483647 |
| **HOUR(x)** | Hour (0-23) | Returns integer |
| **MINUTE(x)** | Minute | Returns integer |
| **MONTH(x)** | Month | Returns integer (1-12) |
| **MONTHNAME(x)** | Month name | Returns string |
| **PARSEDATE(x, y)** | Parse date string | `PARSEDATE('20030102', 'yyyyMMdd')` |
| **PARSETIME(x, y)** | Parse time string | `PARSETIME('22:08:56', 'HH:mm:ss')` |
| **PARSETIMESTAMP(x, y)** | Parse timestamp | `PARSETIMESTAMP(str, format)` |
| **QUARTER(x)** | Quarter (1-4) | Returns integer |
| **SECOND(x)** | Seconds | Returns integer |
| **SERVERTIMEZONE()** | Server timezone | Returns string (tz format) |
| **TIMESTAMPADD(interval, count, ts)** | Add interval to timestamp | See intervals below |
| **TIMESTAMPCREATE(date, time)** | Create timestamp | Returns TIMESTAMP |
| **TIMESTAMPDIFF(interval, start, end)** | Calculate difference | Returns long |
| **WEEK(x)** | Week in year | Returns integer |
| **YEAR(x)** | Four-digit year | Returns integer |

**TIMESTAMPADD/TIMESTAMPDIFF Intervals:**
- SQL_TSI_FRAC_SECOND (billionths of a second)
- SQL_TSI_SECOND
- SQL_TSI_MINUTE
- SQL_TSI_HOUR
- SQL_TSI_DAY
- SQL_TSI_WEEK (Sunday as first day)
- SQL_TSI_MONTH
- SQL_TSI_QUARTER
- SQL_TSI_YEAR

**DATE_TRUNC Precision Values:**
- MICROSECONDS, MILLISECONDS, SECOND, MINUTE, HOUR, DAY, WEEK, MONTH, QUARTER, YEAR, DECADE, CENTURY, MILLENNIUM

---

### JSON Functions

#### Core JSON Functions

| Function | Description | Return Type |
|----------|-------------|-------------|
| **JSONTOXML(rootElementName, json)** | Transform JSON to XML | XML |
| **JSONPATHVALUE(value, path, [nullLeafOnMissing])** | Extract single JSON value as string | string |
| **JSONQUERY(value, path, [nullLeafOnMissing])** | Evaluate JsonPath, return JSON | JSON (clob) |
| **JSONTABLE(...)** | Produce tabular output from JSON | Table |
| **JSONARRAY(value...)** | Create JSON array | clob (JSON) |
| **JSONOBJECT(value [as name] ...)** | Create JSON object | clob (JSON) |
| **JSONPARSE(value, wellformed)** | Validate and return JSON | clob (JSON) |
| **JSONARRAY_AGG(...)** | Aggregate into JSON array | clob (JSON) |

**JsonPath Support**: Uses Jayway JsonPath (0-based indexing)

**JSONTABLE Syntax:**
```sql
JSONTABLE(value, path [, nullLeafOnMissing]
  COLUMNS <COLUMN>, ... ) AS name

COLUMN := name (FOR ORDINALITY | (datatype [PATH string]))
```

**Examples:**
```sql
-- Create JSON array
SELECT JSONARRAY('a"b', 1, NULL, FALSE, {d'2010-11-21'});
-- Returns: ["a\"b",1,null,false,"2010-11-21"]

-- Create JSON object
SELECT JSONOBJECT('val' AS val, 1, NULL as "null");
-- Returns: {"val":"a\"b","expr2":1,"null":null}

-- Parse JsonPath
SELECT JSONPATHVALUE('[{"key":"value1"}, {"key":"value2"}]', '$..key');
-- Returns: value1
```

---

### Numeric Functions

**Standard Operators:** `+`, `-`, `*`, `/`

**Note**: BigDecimal division uses preferred scale of `max(16, dividend.scale + divisor.precision + 1)`

| Function | Description | Constraints |
|----------|-------------|-------------|
| **ABS(x)** | Absolute value | Same type as x |
| **ACOS(x)** | Arc cosine | x in {double, bigdecimal}, returns double |
| **ASIN(x)** | Arc sine | x in {double, bigdecimal}, returns double |
| **ATAN(x)** | Arc tangent | x in {double, bigdecimal}, returns double |
| **ATAN2(x, y)** | Arc tangent of x and y | x, y in {double, bigdecimal}, returns double |
| **CEILING(x)** | Ceiling | x in {double, float}, returns double |
| **COS(x)** | Cosine | x in {double, bigdecimal}, returns double |
| **COT(x)** | Cotangent | x in {double, bigdecimal}, returns double |
| **DEGREES(x)** | Radians to degrees | x in {double, bigdecimal}, returns double |
| **EXP(x)** | e^x | x in {double, float}, returns double |
| **FLOOR(x)** | Floor | x in {double, float}, returns double |
| **LOG(x)** | Natural log | x in {double, float}, returns double |
| **LOG10(x)** | Log base 10 | x in {double, float}, returns double |
| **MOD(x, y)** | Modulus (remainder) | Returns same type as x |
| **PI()** | Value of Pi | Returns double |
| **POWER(x, y)** | x to the y power | Returns same type as x |
| **RADIANS(x)** | Degrees to radians | x in {double, bigdecimal}, returns double |
| **RAND()** | Random (0.0 <= x < 1.0) | Returns double |
| **RAND_SEED(x)** | Random with seed | x is integer, returns double |
| **ROUND(x, y)** | Round to y places | y is integer, returns same type as x |
| **SIGN(x)** | Sign (-1, 0, 1) | Returns integer |
| **SIN(x)** | Sine | x in {double, bigdecimal}, returns double |
| **SQRT(x)** | Square root | x in {long, double, bigdecimal}, returns double |
| **TAN(x)** | Tangent | x in {double, bigdecimal}, returns double |
| **BITAND(x, y)** | Bitwise AND | x, y in {integer}, returns integer |
| **BITOR(x, y)** | Bitwise OR | x, y in {integer}, returns integer |
| **BITXOR(x, y)** | Bitwise XOR | x, y in {integer}, returns integer |
| **BITNOT(x)** | Bitwise NOT | x in {integer}, returns integer |

**Parse Functions:** (use `java.text.DecimalFormat` conventions)
- PARSEBIGDECIMAL(x, y), PARSEBIGINTEGER(x, y), PARSEDOUBLE(x, y), PARSEFLOAT(x, y), PARSEINTEGER(x, y), PARSELONG(x, y)

**Format Functions:** (use `java.text.DecimalFormat` conventions)
- FORMATBIGDECIMAL(x, y), FORMATBIGINTEGER(x, y), FORMATDOUBLE(x, y), FORMATFLOAT(x, y), FORMATINTEGER(x, y), FORMATLONG(x, y)

---

### Security Functions

| Function | Description | Constraints |
|----------|-------------|-------------|
| **HASROLE([roleType,] roleName)** | Check if user has role | roleType: 'data', roleName: string, returns boolean |
| **HASHCODE(string)** | MD5 hex as Numeric(65,0) | Returns biginteger |
| **MD5(x)** | MD5 hash | x in {string}, returns string |
| **MD5_BINARY(x)** | MD5 hash binary | x in {string, varbinary}, returns varbinary |
| **SHA1(x)** | SHA-1 hash | x in {string, varbinary}, returns varbinary |
| **SHA2_256(x)** | SHA-2 256-bit hash | x in {string, varbinary}, returns varbinary |
| **SHA2_512(x)** | SHA-2 512-bit hash | x in {string, varbinary}, returns varbinary |
| **ENCRYPT(x)** | DES encryption | x is string, returns string |

**Note**: Use `TO_CHARS(VALUE, 'HEX')` to convert varbinary hash to hex representation

---

### String Functions

| Function | Description | Constraints |
|----------|-------------|-------------|
| **x \|\| y** | Concatenation operator | x, y in {string, clob} |
| **ASCII(x)** | ASCII value of leftmost char | Returns integer (null for empty string) |
| **CHR(x)** / **CHAR(x)** | Character for ASCII value | x in {integer} |
| **CONCAT(x, y)** | Concatenate (ANSI semantics) | If either is null, returns null |
| **CONCAT2(x, y)** | Concatenate (non-ANSI) | If both null, returns null |
| **INITCAP(x)** | Capitalize first letter of each word | x in {string} |
| **INSERT(str1, start, length, str2)** | Insert string | str1, str2 in {string}, start, length in {integer} |
| **LCASE(x)** | Lowercase | x in {string} |
| **LEFT(x, y)** | Get left y characters | x in {string, clob}, y in {integer} |
| **LENGTH(x)** | Length | Returns integer |
| **LOCATE(x, y)** | Find position | Returns integer (1-based) |
| **LOCATE(x, y, z)** | Find position starting at z | z in {integer} |
| **LPAD(x, y [, z])** | Left pad | y is length, z is pad character |
| **LTRIM(x)** | Left trim | x in {string, clob} |
| **QUERYSTRING(path [, expr [AS name] ...])** | URL-encoded query string | Returns string |
| **REPEAT(str, instances)** | Repeat string | instances in {integer} |
| **REPLACE(x, y, z)** | Replace all y with z | x, y, z in {string, clob} |
| **REGEXP_REPLACE(str, pattern, sub [, flags])** | Regex replace | flags: g (global), m (multiline), i (case insensitive) |
| **RIGHT(x, y)** | Get right y characters | x in {string, clob}, y in {integer} |
| **RPAD(x, y [, z])** | Right pad | y is length, z is pad character |
| **RTRIM(x)** | Right trim | x in {string, clob} |
| **SPLIT_PART(string, delimiter, position)** | Split and get position | position is 1-based |
| **SUBSTRING(x, y [, z])** | Substring | y is start, z is length |
| **TO_CHARS(x, encoding)** | BLOB to CLOB | Encodings: BASE64, HEX, Java Charset names |
| **TO_BYTES(x, encoding)** | CLOB to BLOB | Encodings: BASE64, HEX, Java Charset names |
| **TRANSLATE(x, y, z)** | Replace characters | Replace each char in y with corresponding char in z |
| **TRIM([[LEADING\|TRAILING\|BOTH] [x] FROM] y)** | Trim | Default: BOTH and space |
| **UCASE(x)** | Uppercase | x in {string} |
| **UNESCAPE(x)** | Unescape string | Handles \b, \t, \n, \f, \r, \uXXXX, \XXX |
| **URLENCODE(x, encoding)** | URL encode | x in {string, clob} |
| **URLDECODE(x, encoding)** | URL decode | x in {string, clob} |
| **UUID()** | Generate UUID | Returns string (format: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX) |

---

### System Functions

| Function | Description | Return Type |
|----------|-------------|-------------|
| **CALLER()** | Name of user executing query | string |
| **CURRENT_DATABASE()** | Catalog name | string |
| **DV_SESSION_SET('name', value)** | Set session variable | object (returns old value) |
| **DV_SESSION_GET('name')** | Get session variable | object |
| **ENV('key')** | System property | string (disabled by default) |
| **SESSION_ID()** | Current session ID | string |
| **REQUEST_ID()** | Current request ID | long |
| **USER()** | User owning query | string |
| **LASTUPDATED('fqn')** | Oldest materialization timestamp | timestamp |

**Special Variables:**
- `dv.maxRecursion`: Control max recursion depth

**Enable ENV function:**
```sql
CALL SYSADMIN.executeCli('/subsystem=teiid:write-attribute(name=allow-env-function,value=true)');
```

---

### XML Functions

| Function | Description | Return Type |
|----------|-------------|-------------|
| **JSONTOXML(rootElementName, json)** | JSON to XML | XML |
| **XMLCOMMENT(comment)** | Create XML comment | XML |
| **XMLCONCAT(content [, content]*)** | Concatenate XML | XML |
| **XMLELEMENT([NAME] name [, NSP] [, ATTR] [, content]*)** | Create XML element | XML |
| **XMLESCAPENAME(name, fullEscape)** | Escape name per ISO 9075 | string |
| **XMLFOREST(content [NSP] [, AS name] ...)** | Create XML elements | XML |
| **XMLPARSE((DOCUMENT\|CONTENT) expr [WELLFORMED])** | Parse XML | XML |
| **XMLPI([NAME] name [, content])** | Create processing instruction | XML |
| **XMLQUERY([NSP] xquery [PASSING] [(NULL\|EMPTY) ON EMPTY])** | Execute XQuery | XML |
| **XMLSERIALIZE([(DOCUMENT\|CONTENT)] xml [AS datatype])** | Serialize to string | string/clob |
| **XSLTRANSFORM(doc, xsl)** | Apply XSL stylesheet | clob |
| **XPATHVALUE(doc, xpath)** | Extract text via XPath | string |

**XMLNAMESPACES Syntax:**
```sql
XMLNAMESPACES((uri AS prefix | DEFAULT uri | NO DEFAULT))+
```

**XMLATTRIBUTES Syntax:**
```sql
XMLATTRIBUTES(exp [AS name] [, exp [AS name]]*)
```

**Examples:**
```sql
-- Create XML element
SELECT XMLELEMENT(NAME "MI6",
    XMLNAMESPACES('uri' as ns1),
    XMLELEMENT("head", 'M'),
    XMLELEMENT("agent",
        XMLATTRIBUTES('007' AS "id", 'Martini' AS "drink"),
        'Bond, James'
    )
);

-- XQuery
SELECT XMLQUERY('root/name/text()'
    PASSING CAST('<root><name>John</name></root>' AS xml)
    NULL ON EMPTY
);
```

---

## Virtual Procedures

### Overview

- Created only in virtual schemas (like views)
- Can be **named** (called from SQL) or **anonymous** (called at definition time)
- Commands within procedure: single semicolon (;)
- Between procedures: double semicolon (;;)

### Creating Virtual Procedures

**Syntax:**
```sql
CREATE [OR REPLACE] [PRIVATE] [VIRTUAL] PROCEDURE <schema_name>.<procedure_name>
    ([[<parameter type>] <name> <data type> [DEFAULT <value>][<nullable>][RESULT], ...])
    [RETURNS (<name> <data type> [<nullable>], ... )]
AS
BEGIN
    <procedure code>;
END;;
```

**Parameter Types:**
- **IN**: Input parameter (default if not specified)
- **OUT**: Output parameter
- **INOUT**: Input and output parameter
- **RESULT**: Special OUT parameter notation

### Parameter Examples

**Simple Procedure with IN parameter:**
```sql
CREATE PROCEDURE views.anonymize(IN plain_text STRING NOT NULL)
    RETURNS (anonymized STRING NOT NULL)
AS
BEGIN
    SELECT LEFT(plain_text, 1) || '***' || RIGHT(plain_text, 1);
END;;
```

**Procedure with Default Values:**
```sql
CREATE VIRTUAL PROCEDURE views.proc1(IN i INTEGER DEFAULT '1')
    RETURNS (i INTEGER)
AS
BEGIN
    SELECT i;
END;;

-- Usage
SELECT * FROM views.proc1();      -- returns 1
SELECT * FROM views.proc1(2);     -- returns 2
```

### Output Parameters

**OUT Parameter with RESULT notation:**
```sql
CREATE VIRTUAL PROCEDURE views.proc1(OUT i INTEGER RESULT)
AS
BEGIN
    i = 1;
END;;

-- Usage
BEGIN
    DECLARE INTEGER var;
    var = EXEC views.proc1();
    SELECT var;
END;;  -- returns 1
```

**Multiple OUT and INOUT parameters:**
```sql
CREATE VIRTUAL PROCEDURE views.proc_outs(
    OUT i INTEGER RESULT,
    OUT j INTEGER,
    INOUT k INTEGER
)
AS
BEGIN
    i = 1;
    j = 2;
    k = 3;
END;;

-- Get all values
SELECT * FROM (EXEC views.proc_outs()) x;;  -- returns 1, 2, 3

-- Named parameter assignment
BEGIN
    DECLARE INTEGER var1;
    DECLARE INTEGER var2;
    EXEC views.proc_in_inout(i => var1, j => var2);
    SELECT var1, var2;
END;;  -- returns 1, 2
```

**RETURNS clause (ResultSet):**
```sql
CREATE VIRTUAL PROCEDURE views.proc1()
    RETURNS(i INTEGER, j INTEGER)
AS
BEGIN
    SELECT 1, 2
    UNION
    SELECT 3, 4;
END;;

-- Usage
EXEC views.proc1();
-- or
SELECT * FROM (EXEC views.proc1()) x;;

/* Returns:
1  2
3  4
*/
```

### Executing Virtual Procedures

**Using CALL/EXECUTE:**
```sql
CALL <schema_name>.<procedure_name>[(...)];
```

**Using SELECT (for ResultSet procedures):**
```sql
SELECT * FROM <schema_name>.<procedure_name>[(...)];

-- or
SELECT * FROM (CALL <schema_name>.<procedure_name>[(...)]) AS x;
```

### Viewing Virtual Procedures

**List all procedures:**
```sql
SELECT "id", "name", "definition", "creationDate", "lastModifiedDate",
       "state", "failureReason", "creator", "modifier"
FROM "SYSADMIN.ProcDefinitions";;
```

**Check procedure state:**
```sql
SELECT "state"
FROM "SYSADMIN.ProcDefinitions"
WHERE name = '<procedure_name>';;
```

States: **READY** or **FAILED**

### Altering Virtual Procedures

```sql
ALTER [VIRTUAL] PROCEDURE <schema_name>.<procedure_name>
    ([...parameters...])
    [RETURNS (...)]
AS
BEGIN
    <procedure code>;
END;;
```

### Deleting Virtual Procedures

```sql
DROP [VIRTUAL] PROCEDURE <schema_name>.<procedure_name>;;
```

---

## Function Determinism Levels

Understanding determinism is important for query optimization and caching:

| Level | Description | Functions |
|-------|-------------|-----------|
| **Deterministic** | Always same result for given inputs | Most functions |
| **User Deterministic** | Same result for same user | HASROLE, user functions |
| **Session Deterministic** | Same result in same session | ENV |
| **Command Deterministic** | Deterministic within command scope | CURDATE, CURTIME, NOW |
| **Nondeterministic** | Fully nondeterministic | RAND |

---

## Quick Reference Tips

### Common Patterns

**Current timestamp in different formats:**
```sql
SELECT NOW();                                          -- timestamp
SELECT FORMATDATE(CURDATE(), 'yyyy-MM-dd');           -- date as string
SELECT FORMATTIMESTAMP(NOW(), 'yyyy-MM-dd HH:mm:ss'); -- formatted timestamp
```

**String manipulation:**
```sql
SELECT CONCAT('Hello', ' ', 'World');                 -- Hello World
SELECT REPLACE('Hello World', 'World', 'CData');      -- Hello CData
SELECT SUBSTRING('Hello World', 1, 5);                -- Hello
SELECT UCASE('hello');                                -- HELLO
```

**Date calculations:**
```sql
-- Add 7 days
SELECT TIMESTAMPADD(SQL_TSI_DAY, 7, NOW());

-- Difference in days
SELECT TIMESTAMPDIFF(SQL_TSI_DAY, {d '2024-01-01'}, {d '2024-12-31'});

-- First day of month
SELECT DATE_TRUNC('MONTH', NOW());
```

**JSON operations:**
```sql
-- Create JSON
SELECT JSONOBJECT('id' AS id, 'name' AS name, 'active' AS status);

-- Parse JSON path
SELECT JSONPATHVALUE('{"user":{"name":"John"}}', '$.user.name');
```

**Type conversions:**
```sql
SELECT CAST('123' AS integer);
SELECT CONVERT('2024-01-15', date);
```

---

## Index of All Functions

ARRAY_GET, ARRAY_LENGTH, ARRAY_ADD, ARRAY_SUB, ARRAY_IN, ARRAY_LIKE, ARRAY_LIKE_REGEX, COALESCE, IFNULL, NVL, NULLIF, CURDATE, CURTIME, NOW, DATE_TRUNC, DAYNAME, DAYOFMONTH, DAYOFWEEK, DAYOFYEAR, EXTRACT, FORMATDATE, FORMATTIME, FORMATTIMESTAMP, FROM_UNIXTIME, HOUR, MINUTE, MODIFYTIMEZONE, MONTH, MONTHNAME, PARSEDATE, PARSETIME, PARSETIMESTAMP, QUARTER, SECOND, SERVERTIMEZONE, TIMESTAMPADD, TIMESTAMPCREATE, TIMESTAMPDIFF, WEEK, YEAR, JSONTOXML, JSONPATHVALUE, JSONQUERY, JSONTABLE, JSONARRAY, JSONOBJECT, JSONPARSE, JSONARRAY_AGG, ABS, ACOS, ASIN, ATAN, ATAN2, CEILING, COS, COT, DEGREES, EXP, FLOOR, FORMATBIGDECIMAL, FORMATBIGINTEGER, FORMATDOUBLE, FORMATFLOAT, FORMATINTEGER, FORMATLONG, LOG, LOG10, MOD, PARSEBIGDECIMAL, PARSEBIGINTEGER, PARSEDOUBLE, PARSEFLOAT, PARSEINTEGER, PARSELONG, PI, POWER, RADIANS, RAND, RAND_SEED, ROUND, SIGN, SIN, SQRT, TAN, BITAND, BITOR, BITXOR, BITNOT, HASROLE, HASHCODE, MD5, MD5_BINARY, SHA1, SHA2_256, SHA2_512, ENCRYPT, ASCII, CHR, CHAR, CONCAT, CONCAT2, INITCAP, INSERT, LCASE, LEFT, LENGTH, LOCATE, LPAD, LTRIM, QUERYSTRING, REPEAT, REPLACE, REGEXP_REPLACE, RIGHT, RPAD, RTRIM, SPLIT_PART, SUBSTRING, TO_CHARS, TO_BYTES, TRANSLATE, TRIM, UCASE, UNESCAPE, URLENCODE, URLDECODE, UUID, CALLER, CURRENT_DATABASE, DV_SESSION_SET, DV_SESSION_GET, ENV, SESSION_ID, REQUEST_ID, USER, LASTUPDATED, CONVERT, CAST, XMLCOMMENT, XMLCONCAT, XMLELEMENT, XMLESCAPENAME, XMLFOREST, XMLPARSE, XMLPI, XMLQUERY, XMLSERIALIZE, XSLTRANSFORM, XPATHVALUE

---

*End of Reference Guide*
