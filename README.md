# QExcel

**Oracle SQL and a local DuckDB SQL engine, inside Excel.**

Fetch data from Oracle Fusion once, join it with the tables of your workbook in SQL, take
parameters from cells, and refresh your tables and formulas with one click.

QExcel is not another table downloader. Exports from BI Publisher or OTBI give you a dead file
each time; Smart View brings cubes and balances, not the transactions behind them. QExcel pairs
Excel with [DuckDB](https://duckdb.org), an analytical database that runs on your computer:

1. **Oracle SQL** fetches data from Fusion — any table or view your role can read — and keeps it
   on your computer.
2. **DuckDB SQL** totals, filters and joins what was fetched, together with tables of your
   workbook, on your computer, without going back to Fusion.

Runs on **macOS and Windows**.

## Why use it

- **Two SQL engines working together.**
  - *From Fusion, in Oracle SQL:* AP, AR, GL, SLA, PO, FA, HR. Results are kept in an encrypted
    local database — a million rows by default, up to 100 million for a query that only feeds
    calculations.
  - *Calculated on your computer, in DuckDB SQL:* `JOIN`, `GROUP BY`, window functions, `CASE`
    over what was fetched — change the grouping or a filter without waiting on Fusion again.
- **Your sheets are tables in SQL.** Any Excel table or range — a budget, an account mapping, a
  bank statement — becomes a table that SQL joins with Fusion data. No chains of `XLOOKUP`s, no
  Power Query.
- **Parameters live on your sheet.** Point a parameter (`:period` in Oracle SQL, `$period` in
  DuckDB SQL) to a cell. Change the cell, click **Refresh**, and the Excel table updates in place
  — the formulas around it keep working.
- **Keep the detail out of the sheet.** Line-item data stays in the local database; the sheet gets
  what the report needs, as a table or with `=OF.TABLE(…)` and `=OF.CALL(…)`. Formulas read the
  local data and never call Fusion themselves.
- **Workbooks that travel, safely.** Queries, calculations and their cell bindings are kept in the
  `.xlsx`. A colleague opens it, signs in with their own Fusion account and refreshes. DEV, TEST
  and PROD are shown in colour; queries from someone else's workbook run only after you confirm
  them; HR tables are read through their secured views.

## Example: the two SQL layers

**1. Fetch from Fusion once — Oracle SQL.** A step `gl_actuals`, the posted journal lines of a
period:

```sql
SELECT cc.segment2 AS cost_center,      -- the segments depend on your chart of accounts
       cc.segment3 AS account,
       NVL(l.accounted_dr, 0) - NVL(l.accounted_cr, 0) AS amount
FROM gl_je_lines l
JOIN gl_code_combinations cc ON cc.code_combination_id = l.code_combination_id
WHERE l.period_name = :period
  AND l.status = 'P'
```

**2. Read a table of your workbook.** Select the budget table on a sheet and add it as an Excel
source, `budget`, with columns `cost_center`, `account`, `budget_amount`.

**3. Calculate on your computer — DuckDB SQL.** A step `variance` reads the other two by their
names:

```sql
WITH actual AS (
    SELECT cost_center, account, SUM(amount) AS actual
    FROM gl_actuals
    GROUP BY cost_center, account
)
SELECT a.cost_center, a.account, a.actual,
       b.budget_amount AS budget,
       a.actual - b.budget_amount AS variance
FROM actual a
LEFT JOIN budget b USING (cost_center, account)
WHERE a.cost_center = $dept
```

Put `variance` on a sheet as a refreshable Excel table, or in a cell:

```
=OF.TABLE("variance", B1, B2)      the period in B1, the cost center in B2
=OF.ASOF("gl_actuals", B1)         when the period was fetched
```

Change the cost center in `B2` and the formula is calculated again on your computer, from the
journal lines already fetched — Fusion is not asked. Change the period in `B1` and **Refresh**
fetches it.

## QExcel and the standard tools

| | BI Publisher / OTBI export | Smart View | QExcel |
|---|---|---|---|
| Your own SQL over Fusion tables (AP, AR, GL, SLA, PO, FA…) | Through a report built in Fusion | No: cubes and analyses | Yes, from Excel |
| A SQL engine over data already fetched | No | No | Yes: DuckDB on your computer |
| Fusion data joined with tables of your workbook | By hand | With Excel formulas | In SQL |
| Parameters from Excel cells | No: entered in the browser | For cube members | Yes |
| Result updated in your workbook | No: a new file each time | Yes | Yes |

## Download and install

Get the installer for macOS (Apple Silicon, macOS 13 or later) or Windows (10 or 11, 64-bit)
from [Releases](https://github.com/krokozyab/qexcel/releases).

- [Installing on macOS](docs/install-macos.md)
- [Installing on Windows](docs/install-windows.md)

QExcel runs entirely on your computer: the add-in pane in Excel and a small local service. No
server in the middle, no administrator rights. You also need Excel (Excel for Mac, or Excel from
Microsoft 365 on Windows), and Google Chrome, Microsoft Edge or Chromium for single sign-on.

### One report in Fusion

QExcel does not work against a stock Fusion instance: it runs SQL through one small BI
Publisher report deployed in your instance.

- **Already using [ofjdbc](https://github.com/krokozyab/ofjdbc)?** It is the same report. Use the
  report path from your JDBC URL as the connection's **SQL report path**, and you are done.
- **Otherwise, deploy it once.** Un-archive `DM_ARB.xdm.catalog` and `RP_ARB.xdo.catalog` from
  [ofjdbc/otbireport](https://github.com/krokozyab/ofjdbc/tree/master/otbireport) into
  `/Shared Folders/Custom/Financials`. QExcel's default path, `/Custom/Financials/RP_ARB.xdo`,
  then works as is; another folder works too, with its path as the **SQL report path**.

## Documentation

The [user guide](docs/user-guide.md) covers the QExcel pane: connections, parameters from cells,
the query editor, calculations in DuckDB SQL, Excel sources, formulas, the query header and what
to do when something goes wrong.
