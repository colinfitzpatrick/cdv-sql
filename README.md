# CDV SQL Skill for Claude Code

**Status:** Works for me

A Claude Code skill for writing and executing CData Virtuality (CDV) SQL queries.

## What It Does

When invoked via `/cdv-sql`, this skill gives Claude knowledge of:

- CDV-specific syntax rules (double semicolon `;;`, 1-based arrays, etc.)
- All supported scalar functions (Array, Date/Time, String, JSON, Numeric, Security, System, XML)
- Virtual procedure creation and execution
- Data types and type conversions
- System catalog queries (`SYS.Tables`, `SYS.Columns`, `SYSADMIN.ProcDefinitions`)
- Helper script setup for executing queries via `dsql.jar`

## Setup

The skill is automatically available in any Claude Code session within this project.

### Query Execution (Optional)

To execute queries against a CDV server, set up the helper scripts:

1. **Create a `.env` file** in the project root (add to `.gitignore`):
   ```
   DV_USERNAME=your_username
   DV_PASSWORD=your_password
   DV_HOST=your_host
   DV_PORT=31000
   DV_SSL=false
   ```

2. **Create helper scripts** — the skill contains ready-to-use templates for:
   - `run-query.sh` — execute SQL from a file
   - `quick-query.sh` — run inline SQL queries
   - Windows `.bat` equivalents

3. **Make scripts executable:**
   ```bash
   chmod +x run-query.sh quick-query.sh
   ```

4. **Ensure `dsql.jar`** is available at `./bin/dsql.jar` and Java is on your PATH.

## Usage

```
/cdv-sql
```

Then ask Claude to write queries, convert SQL from other dialects, create virtual procedures, or set up helper scripts.

### Examples

- "Write a query to list all materialized views"
- "Convert this MySQL query to CDV syntax"
- "Create a virtual procedure that anonymizes email addresses"
- "Set up run-query.sh for my environment"

## Files

```
.claude/skills/cdv-sql/
  SKILL.md    — Skill definition (loaded by Claude Code)
  README.md   — This file
```

## Full Reference

For exhaustive function documentation beyond what the skill contains, see `CData_Virtuality_SQL_Reference.md` in the project root.
