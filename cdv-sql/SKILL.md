---
name: cdv-sql
description: >
  Write and execute CData Virtuality (CDV) SQL queries. Covers CDV-specific
  syntax rules, all supported functions, virtual procedures, data types,
  helper scripts for query execution, and system catalog queries.
  Use this skill whenever writing, converting, or debugging SQL for a
  CData Virtuality (formerly Data Virtuality) server.
---

# CData Virtuality SQL Skill

## 1. Critical Syntax Rules

- **All statements end with `;;`** (double semicolon) — NOT single `;`
- Single `;` is ONLY used inside virtual procedure bodies between statements; the procedure itself ends with `;;`
- SELECT syntax is **PostgreSQL compatible**
- **Only use functions listed in this skill or in `CData_Virtuality_SQL_Reference.md`** — do NOT assume functions from MySQL, SQL Server, Oracle, or other dialects exist
- **Array indexes start at 1** (not 0)
- Identifiers are **case-insensitive** (even when double-quoted)
- Double-quote character escaped by doubling: `"some "" id"`
- Fully qualified names: `schema.table`, `schema.table.column`
- Columns, schemas, and aliases cannot contain a dot; table specifications can

---

## 2. Data Types

| Type | Aliases | Description |
|------|---------|-------------|
| **string** | varchar | Variable length string (max 4000) |
| **char** | | Single Unicode character |
| **boolean** | | TRUE, FALSE, or NULL |
| **byte** | tinyint | Signed 8-bit integer |
| **short** | smallint | Signed 16-bit integer |
| **integer** | serial | Signed 32-bit integer |
| **long** | bigint | Signed 64-bit integer |
| **biginteger** | | Arbitrary precision integer (up to 1000 digits) |
| **float** | real | 32-bit IEEE 754 |
| **double** | | 64-bit IEEE 754 |
| **bigdecimal** | decimal | Arbitrary precision decimal (up to 1000 digits) |
| **date** | | Year, month, day |
| **time** | | Hours, minutes, seconds, milliseconds |
| **timestamp** | | Date and time |
| **varbinary** | | Binary string (max 8192) |
| **object** | | Any Java Serializable object |
| **blob** | | Binary large object |
| **clob** | | Character large object |
| **xml** | | XML document |

---

## 3. Type Conversions

```sql
CONVERT(x, type)    -- JDBC/ODBC syntax
CAST(x AS type)     -- Standard SQL syntax
```

**Date/Time Literals:**
```sql
{d 'yyyy-mm-dd'}                      -- DATE
{t 'hh:mm:ss'}                        -- TIME
{ts 'yyyy-mm-dd hh:mm:ss.[fff...]'}   -- TIMESTAMP
```

**String to Boolean:** `'false'` -> FALSE, `'unknown'` -> NULL, anything else -> TRUE
**Numeric to Boolean:** `0` -> FALSE, anything else -> TRUE

---

## 4. Function Reference

### 4a. Array Functions

**Declare arrays:** `DECLARE STRING[] my_array = ARRAY('a', 'b', 'c');`
**Array indexes are 1-based.**

| Function | Syntax | Returns |
|----------|--------|---------|
| **ARRAY_GET** | `ARRAY_GET(array, index)` | object |
| **ARRAY_LENGTH** | `ARRAY_LENGTH(array)` | integer |
| **ARRAY_ADD** | `ARRAY_ADD(array, element_or_array)` | object |
| **ARRAY_SUB** | `ARRAY_SUB(array, start, count)` | object |
| **ARRAY_IN** | `ARRAY_IN(array, value)` | integer (0 if not found) |
| **ARRAY_LIKE** | `ARRAY_LIKE(array, like_pattern)` | boolean |
| **ARRAY_LIKE_REGEX** | `ARRAY_LIKE_REGEX(array, regex)` | boolean |

### 4b. Choice Functions

| Function | Description | Notes |
|----------|-------------|-------|
| **COALESCE(x, y, ...)** | First non-null value | All params must be compatible types |
| **IFNULL(x, y)** / **NVL(x, y)** | If x null, return y | x and y must be same type |
| **NULLIF(a, b)** | If a = b, return NULL; else a | Params must be compatible |

### 4c. Date/Time Functions

**Format patterns use `java.text.SimpleDateFormat` conventions** (yyyy, MM, dd, HH, mm, ss, etc.)

| Function | Description | Example |
|----------|-------------|---------|
| **CURDATE()** | Current date | Returns DATE |
| **CURTIME()** | Current time | Returns TIME |
| **NOW()** | Current timestamp | Returns TIMESTAMP |
| **DATE_TRUNC(precision, x)** | Truncate to precision | `DATE_TRUNC('MONTH', ts)` |
| **DAYNAME(x)** | Day name | Returns string |
| **DAYOFMONTH(x)** | Day of month (1-31) | Returns integer |
| **DAYOFWEEK(x)** | Day of week (Sunday=1) | Returns integer |
| **DAYOFYEAR(x)** | Julian day number | Returns integer |
| **EXTRACT(field FROM x)** | Extract field | `EXTRACT(YEAR FROM date)` |
| **FORMATDATE(x, fmt)** | Format date to string | `FORMATDATE(date, 'yyyy-MM-dd')` |
| **FORMATTIME(x, fmt)** | Format time to string | `FORMATTIME(time, 'HH:mm:ss')` |
| **FORMATTIMESTAMP(x, fmt)** | Format timestamp to string | `FORMATTIMESTAMP(ts, 'yyyy-MM-dd HH:mm:ss')` |
| **FROM_UNIXTIME(seconds)** | Unix seconds to TIMESTAMP | Max: 2147483647 |
| **HOUR(x)** | Hour (0-23) | Returns integer |
| **MINUTE(x)** | Minute | Returns integer |
| **MONTH(x)** | Month (1-12) | Returns integer |
| **MONTHNAME(x)** | Month name | Returns string |
| **PARSEDATE(x, fmt)** | Parse string to date | `PARSEDATE('20240115', 'yyyyMMdd')` |
| **PARSETIME(x, fmt)** | Parse string to time | `PARSETIME('22:08:56', 'HH:mm:ss')` |
| **PARSETIMESTAMP(x, fmt)** | Parse string to timestamp | `PARSETIMESTAMP(str, fmt)` |
| **QUARTER(x)** | Quarter (1-4) | Returns integer |
| **SECOND(x)** | Seconds | Returns integer |
| **SERVERTIMEZONE()** | Server timezone | Returns string |
| **TIMESTAMPADD(interval, count, ts)** | Add interval to timestamp | See intervals below |
| **TIMESTAMPCREATE(date, time)** | Create timestamp from parts | Returns TIMESTAMP |
| **TIMESTAMPDIFF(interval, start, end)** | Difference between timestamps | Returns long |
| **WEEK(x)** | Week in year | Returns integer |
| **YEAR(x)** | Four-digit year | Returns integer |

**TIMESTAMPADD/TIMESTAMPDIFF Intervals:**
`SQL_TSI_FRAC_SECOND`, `SQL_TSI_SECOND`, `SQL_TSI_MINUTE`, `SQL_TSI_HOUR`, `SQL_TSI_DAY`, `SQL_TSI_WEEK`, `SQL_TSI_MONTH`, `SQL_TSI_QUARTER`, `SQL_TSI_YEAR`

**DATE_TRUNC Precision Values:**
`MICROSECONDS`, `MILLISECONDS`, `SECOND`, `MINUTE`, `HOUR`, `DAY`, `WEEK`, `MONTH`, `QUARTER`, `YEAR`, `DECADE`, `CENTURY`, `MILLENNIUM`

### 4d. JSON Functions

**JsonPath uses Jayway syntax (0-based indexing within JsonPath, even though CDV arrays are 1-based)**

| Function | Description | Returns |
|----------|-------------|---------|
| **JSONOBJECT(value [AS name], ...)** | Create JSON object | clob (JSON) |
| **JSONARRAY(value, ...)** | Create JSON array | clob (JSON) |
| **JSONPATHVALUE(json, path [, nullLeafOnMissing])** | Extract single value as string | string |
| **JSONQUERY(json, path [, nullLeafOnMissing])** | Evaluate JsonPath, return JSON | clob (JSON) |
| **JSONTABLE(json, path COLUMNS ...) AS name** | Tabular output from JSON | table |
| **JSONTOXML(rootName, json)** | JSON to XML | XML |
| **JSONPARSE(value, wellformed)** | Validate/return JSON | clob (JSON) |
| **JSONARRAY_AGG(...)** | Aggregate into JSON array | clob (JSON) |

**JSONTABLE Syntax:**
```sql
JSONTABLE(value, path [, nullLeafOnMissing]
  COLUMNS name (FOR ORDINALITY | (datatype [PATH string])), ...) AS alias
```

### 4e. Numeric Functions

**Operators:** `+`, `-`, `*`, `/`

| Function | Description | Returns |
|----------|-------------|---------|
| **ABS(x)** | Absolute value | same type |
| **ACOS(x)** | Arc cosine | double |
| **ASIN(x)** | Arc sine | double |
| **ATAN(x)** | Arc tangent | double |
| **ATAN2(x, y)** | Arc tangent of x/y | double |
| **CEILING(x)** | Ceiling | double |
| **COS(x)** | Cosine | double |
| **COT(x)** | Cotangent | double |
| **DEGREES(x)** | Radians to degrees | double |
| **EXP(x)** | e^x | double |
| **FLOOR(x)** | Floor | double |
| **LOG(x)** | Natural logarithm | double |
| **LOG10(x)** | Base-10 logarithm | double |
| **MOD(x, y)** | Modulus (remainder) | same type as x |
| **PI()** | Value of pi | double |
| **POWER(x, y)** | x to the y power | same type as x |
| **RADIANS(x)** | Degrees to radians | double |
| **RAND()** | Random (0.0 <= x < 1.0) | double |
| **RAND_SEED(x)** | Random with seed | double |
| **ROUND(x, y)** | Round x to y decimal places | same type as x |
| **SIGN(x)** | Sign (-1, 0, 1) | integer |
| **SIN(x)** | Sine | double |
| **SQRT(x)** | Square root | double |
| **TAN(x)** | Tangent | double |
| **BITAND(x, y)** | Bitwise AND | integer |
| **BITOR(x, y)** | Bitwise OR | integer |
| **BITXOR(x, y)** | Bitwise XOR | integer |
| **BITNOT(x)** | Bitwise NOT | integer |

**Parse/Format Functions** (use `java.text.DecimalFormat` conventions):
- Parse: `PARSEBIGDECIMAL`, `PARSEBIGINTEGER`, `PARSEDOUBLE`, `PARSEFLOAT`, `PARSEINTEGER`, `PARSELONG`
- Format: `FORMATBIGDECIMAL`, `FORMATBIGINTEGER`, `FORMATDOUBLE`, `FORMATFLOAT`, `FORMATINTEGER`, `FORMATLONG`

### 4f. String Functions

| Function | Description | Notes |
|----------|-------------|-------|
| **x \|\| y** | Concatenation operator | x, y in {string, clob} |
| **ASCII(x)** | ASCII value of leftmost char | Returns integer |
| **CHR(x)** / **CHAR(x)** | Character for ASCII value | x is integer |
| **CONCAT(x, y)** | Concatenate (ANSI) | If either null, returns null |
| **CONCAT2(x, y)** | Concatenate (non-ANSI) | Null only if both null |
| **INITCAP(x)** | Capitalize first letter of each word | |
| **INSERT(str1, start, length, str2)** | Insert string | 1-based start |
| **LCASE(x)** / **LOWER** | Lowercase | |
| **LEFT(x, y)** | Left y characters | |
| **LENGTH(x)** | String length | Returns integer |
| **LOCATE(x, y [, z])** | Find position of x in y | 1-based, z = start position |
| **LPAD(x, y [, z])** | Left pad to length y | z = pad char (default space) |
| **LTRIM(x)** | Left trim whitespace | |
| **QUERYSTRING(path [, expr [AS name] ...])** | Build URL query string | Returns string |
| **REPEAT(str, n)** | Repeat string n times | |
| **REPLACE(x, y, z)** | Replace all y with z in x | |
| **REGEXP_REPLACE(str, pattern, sub [, flags])** | Regex replace | flags: g, m, i |
| **RIGHT(x, y)** | Right y characters | |
| **RPAD(x, y [, z])** | Right pad to length y | z = pad char (default space) |
| **RTRIM(x)** | Right trim whitespace | |
| **SPLIT_PART(str, delimiter, position)** | Split and get part | 1-based position |
| **SUBSTRING(x, y [, z])** | Substring from y, length z | 1-based |
| **TO_CHARS(x, encoding)** | BLOB/varbinary to CLOB | Encodings: BASE64, HEX, or Java Charset |
| **TO_BYTES(x, encoding)** | CLOB/string to BLOB | Encodings: BASE64, HEX, or Java Charset |
| **TRANSLATE(x, y, z)** | Replace each char in y with corresponding in z | |
| **TRIM([[LEADING\|TRAILING\|BOTH] [x] FROM] y)** | Trim characters | Default: BOTH, space |
| **UCASE(x)** / **UPPER** | Uppercase | |
| **UNESCAPE(x)** | Unescape string | Handles \b, \t, \n, \f, \r, \uXXXX |
| **URLENCODE(x, encoding)** | URL encode | |
| **URLDECODE(x, encoding)** | URL decode | |
| **UUID()** | Generate UUID | Format: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX |

**REGEXP_REPLACE flags:** `g` (global/all occurrences), `m` (multiline), `i` (case insensitive)

### 4g. Security Functions

| Function | Description | Notes |
|----------|-------------|-------|
| **HASROLE([roleType,] roleName)** | Check user role | roleType: 'data', returns boolean |
| **HASHCODE(x)** | MD5 hex as Numeric(65,0) | Returns biginteger |
| **MD5(x)** | MD5 hash | Returns string |
| **MD5_BINARY(x)** | MD5 hash binary | Returns varbinary |
| **SHA1(x)** | SHA-1 hash | Returns varbinary |
| **SHA2_256(x)** | SHA-2 256-bit hash | Returns varbinary |
| **SHA2_512(x)** | SHA-2 512-bit hash | Returns varbinary |
| **ENCRYPT(x)** | DES encryption | Returns string |

**Tip:** Use `TO_CHARS(SHA2_256(value), 'HEX')` to get hex string output from hash functions.

### 4h. System Functions

| Function | Description | Returns |
|----------|-------------|---------|
| **USER()** | User owning the query | string |
| **CALLER()** | User executing the query | string |
| **SESSION_ID()** | Current session ID | string |
| **REQUEST_ID()** | Current request ID | long |
| **CURRENT_DATABASE()** | Catalog/VDB name | string |
| **ENV('key')** | System property | string (disabled by default) |
| **DV_SESSION_SET('name', value)** | Set session variable | object (returns old value) |
| **DV_SESSION_GET('name')** | Get session variable | object |
| **LASTUPDATED('fqn')** | Oldest materialization timestamp | timestamp |

### 4i. XML Functions

| Function | Description | Returns |
|----------|-------------|---------|
| **JSONTOXML(rootName, json)** | JSON to XML | XML |
| **XMLCOMMENT(comment)** | Create XML comment | XML |
| **XMLCONCAT(content, ...)** | Concatenate XML | XML |
| **XMLELEMENT([NAME] name [, NSP] [, ATTR] [, content]*)** | Create element | XML |
| **XMLESCAPENAME(name, fullEscape)** | Escape per ISO 9075 | string |
| **XMLFOREST(content [AS name], ...)** | Create elements | XML |
| **XMLPARSE((DOCUMENT\|CONTENT) expr [WELLFORMED])** | Parse XML | XML |
| **XMLPI([NAME] name [, content])** | Processing instruction | XML |
| **XMLQUERY([NSP] xquery [PASSING] [(NULL\|EMPTY) ON EMPTY])** | XQuery | XML |
| **XMLSERIALIZE([(DOCUMENT\|CONTENT)] xml [AS type])** | Serialize to string | string/clob |
| **XSLTRANSFORM(doc, xsl)** | Apply XSL stylesheet | clob |
| **XPATHVALUE(doc, xpath)** | Extract text via XPath | string |

---

## 5. Virtual Procedures

### Create Syntax

```sql
CREATE [OR REPLACE] [PRIVATE] [VIRTUAL] PROCEDURE schema.procedure_name(
    [[IN|OUT|INOUT] param_name DATA_TYPE [DEFAULT value] [NOT NULL] [RESULT], ...]
)
    [RETURNS (col_name DATA_TYPE [NOT NULL], ...)]
AS
BEGIN
    -- statements separated by single semicolon ;
    SELECT ...;
END;;  -- double semicolon ends the procedure
```

**Parameter types:** IN (default), OUT, INOUT, RESULT (special OUT notation)

### Examples

**Simple function-like procedure:**
```sql
CREATE PROCEDURE views.anonymize(IN plain_text STRING NOT NULL)
    RETURNS (anonymized STRING NOT NULL)
AS
BEGIN
    SELECT LEFT(plain_text, 1) || '***' || RIGHT(plain_text, 1);
END;;
```

**Procedure with default values:**
```sql
CREATE VIRTUAL PROCEDURE views.proc1(IN i INTEGER DEFAULT '1')
    RETURNS (i INTEGER)
AS
BEGIN
    SELECT i;
END;;

-- Usage
SELECT * FROM views.proc1();;   -- returns 1
SELECT * FROM views.proc1(2);;  -- returns 2
```

**OUT/INOUT parameters:**
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

SELECT * FROM (EXEC views.proc_outs()) x;;  -- returns 1, 2, 3
```

### Execution

```sql
CALL schema.procedure_name(args);;
EXEC schema.procedure_name(args);;
SELECT * FROM schema.procedure_name(args);;
SELECT * FROM (CALL schema.procedure_name(args)) AS x;;
```

### Alter / Drop

```sql
ALTER VIRTUAL PROCEDURE schema.procedure_name(...) AS BEGIN ... END;;
DROP VIRTUAL PROCEDURE schema.procedure_name;;
```

---

## 6. System Catalog Queries

**List all tables and views:**
```sql
SELECT "VDBName", "SchemaName", "Name", "Type", "IsPhysical", "IsMaterialized"
FROM "SYS.Tables";;
```

Key columns:
- **SchemaName** — data source schema
- **Name** — table/view name
- **Type** — "Table" or "View"
- **IsPhysical** — `true` = physical source table, `false` = virtual view
- **IsMaterialized** — whether the view is materialized (cached)

**List columns for a specific table:**
```sql
SELECT "Name", "DataType", "Position", "NullType"
FROM "SYS.Columns"
WHERE "SchemaName" = 'your_schema'
  AND "TableName" = 'your_table'
ORDER BY "Position";;
```

**List procedures:**
```sql
SELECT "id", "name", "definition", "state", "creationDate", "lastModifiedDate"
FROM "SYSADMIN.ProcDefinitions";;
```

Procedure states: **READY** or **FAILED**

---

## 7. Query Execution — Helper Scripts

### 7a. Environment Setup

Create a `.env` file with your credentials (add `.env` to `.gitignore`):

```
DV_USERNAME=your_username
DV_PASSWORD=your_password
DV_HOST=your_host
DV_PORT=31000
DV_SSL=false
```

### 7b. dsql.jar Parameters

The `dsql.jar` tool executes SQL against a CData Virtuality server:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `username` | Database username | (required) |
| `password` | Database password | (required) |
| `host` | Server hostname | 127.0.0.1 |
| `port` | Server port | 31000 (SSL=false), 31001 (SSL=true) |
| `vdb` | Virtual Database name | (server default) |
| `ssl` | Enable SSL | false |
| `maxlines` | Max result rows (-1 = unlimited) | 20 |
| `sourcefile` | Path to SQL file | (mutually exclusive with `query`) |
| `query` | Inline SQL string | (mutually exclusive with `sourcefile`) |
| `output` | Output format: "screen" or "CSV" | screen |
| `outfilename` | Output filename for CSV | (optional) |

**Important:** If the password contains shell special characters (`^`, `!`, `&`, etc.), always use `sourcefile` instead of `query` to avoid shell interpretation issues.

### 7c. run-query.sh — Run SQL from File

```bash
#!/bin/bash
########################################
# CData Virtuality Query Helper Script
# Usage: ./run-query.sh <sql-file> [output-file]
########################################

if [ -z "$1" ]; then
    echo "Usage: ./run-query.sh <sql-file> [output-file]"
    exit 1
fi

SQL_FILE="$1"
if [ ! -f "$SQL_FILE" ]; then
    echo "ERROR: File not found: $SQL_FILE"
    exit 1
fi

OUTPUT_FILE="${2:-output.log}"

# Configuration — update these paths for your environment
JAVA_PATH="java"
DSQL_JAR="./bin/dsql.jar"
SECURITY_PROPS="./standalone/configuration/java-security.properties"

# Load credentials from .env
source .env

echo "========================================"
echo "Running CData Virtuality Query"
echo "========================================"
echo "SQL File: $SQL_FILE"
echo "Output:   $OUTPUT_FILE"
echo "========================================"

"$JAVA_PATH" \
    -Djavax.net.ssl.trustStoreType=WINDOWS-ROOT \
    -Djava.security.properties="$SECURITY_PROPS" \
    -jar "$DSQL_JAR" \
    "username=$DV_USERNAME" \
    "password=$DV_PASSWORD" \
    "host=$DV_HOST" \
    "ssl=$DV_SSL" \
    "sourcefile=$SQL_FILE" \
    "maxlines=-1" \
    > "$OUTPUT_FILE"

if [ $? -ne 0 ]; then
    echo "ERROR: Query execution failed. Check dsql-error.log for details."
    exit 1
fi

echo "Query completed successfully!"
echo "========================================"
cat "$OUTPUT_FILE"
echo "========================================"
echo "Full output saved to: $OUTPUT_FILE"
```

### 7d. quick-query.sh — Run Inline SQL

```bash
#!/bin/bash
########################################
# Quick Inline Query Helper
# Usage: ./quick-query.sh "SELECT 1;;"
########################################

if [ -z "$1" ]; then
    echo "Usage: ./quick-query.sh \"YOUR SQL QUERY;;\""
    echo "Note: Query must end with double semicolon ;;"
    exit 1
fi

TEMP_FILE="temp_query.sql"
OUTPUT_FILE="quick_output.log"

echo "$1" > "$TEMP_FILE"

# Configuration — update these paths for your environment
JAVA_PATH="java"
DSQL_JAR="./bin/dsql.jar"
SECURITY_PROPS="./standalone/configuration/java-security.properties"

# Load credentials from .env
source .env

echo "========================================"
echo "Running Quick Query"
echo "========================================"

"$JAVA_PATH" \
    -Djavax.net.ssl.trustStoreType=WINDOWS-ROOT \
    -Djava.security.properties="$SECURITY_PROPS" \
    -jar "$DSQL_JAR" \
    "username=$DV_USERNAME" \
    "password=$DV_PASSWORD" \
    "host=$DV_HOST" \
    "ssl=$DV_SSL" \
    "sourcefile=$TEMP_FILE" \
    > "$OUTPUT_FILE"

if [ $? -ne 0 ]; then
    echo "ERROR: Query failed. Check dsql-error.log for details."
    rm -f "$TEMP_FILE"
    exit 1
fi

cat "$OUTPUT_FILE"
rm -f "$TEMP_FILE"
```

### 7e. Windows .bat Equivalents

The same scripts can be written as `.bat` files using Windows syntax:
- Use `SET VAR=value` instead of `VAR="value"`
- Use `%VAR%` instead of `$VAR`
- Use `^` for line continuation instead of `\`
- Use `IF "%1"==""` for argument checks
- Use `DEL` instead of `rm`
- Load `.env` with a `FOR /F` loop:
  ```bat
  FOR /F "tokens=1,2 delims==" %%A IN (.env) DO SET %%A=%%B
  ```

### 7f. Usage Examples

```bash
# Run SQL from a file
./run-query.sh my_query.sql

# Run with custom output file
./run-query.sh my_query.sql results.txt

# Quick inline query
./quick-query.sh "SELECT 1;;"

# Use single outer quotes when query contains double-quoted identifiers
./quick-query.sh 'SELECT "Name" FROM "SYS.Tables" LIMIT 5;;'

# Escaped double quotes alternative
./quick-query.sh "SELECT \"Name\" FROM \"SYS.Tables\" LIMIT 5;;"
```

Make scripts executable: `chmod +x run-query.sh quick-query.sh`

---

## 8. Common Patterns Quick Reference

**Current date/time formatting:**
```sql
SELECT NOW();;
SELECT FORMATDATE(CURDATE(), 'yyyy-MM-dd');;
SELECT FORMATTIMESTAMP(NOW(), 'yyyy-MM-dd HH:mm:ss');;
```

**String manipulation:**
```sql
SELECT CONCAT('Hello', ' ', 'World');;
SELECT REPLACE('Hello World', 'World', 'CDV');;
SELECT SUBSTRING('Hello World', 1, 5);;
SELECT UCASE('hello');;
SELECT REGEXP_REPLACE('abc123def', '[0-9]+', 'NUM', 'g');;
```

**Date arithmetic:**
```sql
SELECT TIMESTAMPADD(SQL_TSI_DAY, 7, NOW());;              -- add 7 days
SELECT TIMESTAMPDIFF(SQL_TSI_DAY, {d '2024-01-01'}, {d '2024-12-31'});;  -- diff in days
SELECT DATE_TRUNC('MONTH', NOW());;                        -- first day of month
```

**JSON operations:**
```sql
SELECT JSONOBJECT('id' AS id, 'name' AS name);;
SELECT JSONPATHVALUE('{"user":{"name":"John"}}', '$.user.name');;
```

**Type conversions:**
```sql
SELECT CAST('123' AS integer);;
SELECT CONVERT('2024-01-15', date);;
```

**UUID generation:**
```sql
SELECT UUID();;
```

---

## 9. Full Reference

For exhaustive function documentation with full examples, constraints, edge cases, and the complete type conversion matrix, consult:

**`CData_Virtuality_SQL_Reference.md`**.

