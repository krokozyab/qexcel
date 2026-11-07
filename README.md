# QExcel

**Live Oracle Fusion SQL inside Excel — parameters in worksheet cells, one-click refresh,
formulas that work from local data.**

Stop exporting `.xlsx` files from BI Publisher or OTBI and pasting them into your workbooks.
Smart View brings cubes and balances into Excel; QExcel brings the transactions behind them. It
runs your own Oracle SQL against Fusion tables and views — AP, AR, PO, FA, SLA, HR — keeps the
result on your computer in an embedded [DuckDB](https://duckdb.org) database, and writes it
into Excel tables and formulas that refresh in place.

QExcel runs on **macOS and Windows**.

## Why finance teams and Fusion consultants use it

- **Report parameters live on your sheet.** Point a query parameter (`:period`, `:ledger`,
  `:bu`) to a cell. Change `Oct-26` to `Nov-26`, click **Refresh**, and the Excel table updates
  in place — the formulas around it keep working.
- **Fusion only when you ask.** Formulas (`OF.TABLE`, `OF.CALL`) read data already fetched; they
  never call Fusion by themselves. Fusion is asked when you click **Refresh** or **Fetch** — so
  recalculating the workbook does not wait on the network.
- **Combine Fusion data with your own tables.** Join a Fusion result with a bank statement, a
  budget or an account mapping that sits in your workbook, in plain SQL on your computer —
  without chains of `XLOOKUP`s or Power Query.
- **Self-contained workbooks.** Queries and their parameter bindings travel inside the `.xlsx`.
  Send the workbook to a colleague: they sign in with their own Fusion account and refresh.
- **Built-in guardrails.**
  - DEV, TEST and PROD shown in colour wherever their data appears.
  - Queries from someone else's workbook run only after you confirm them.
  - HR tables are read through their secured views, so you see only the people your role
    allows.

## How it works in 30 seconds

1. **Connect once** to your Fusion environment — DEV, TEST or PROD — with single sign-on in a
   browser, or with a username and password.
2. **Add a query.** Write Oracle SQL, pick one from your library or the examples, or calculate
   from data already fetched:

   ```sql
   SELECT invoice_num, invoice_date, invoice_amount
   FROM ap_invoices_all
   WHERE org_id = :bu
     AND invoice_date >= :from_date
   ```

3. **Bind the parameters to cells** (`bu ← B1`, `from_date ← B2`) and place the result as a
   refreshable Excel table, or read it with formulas:

   ```
   =OF.TABLE("ap_invoices", B1, B2)
   =OF.CALL("open_ap_total", B1)
   =OF.ASOF("ap_invoices", B1, B2)      when the data was last fetched
   ```

## QExcel and the standard tools

| | BI Publisher / OTBI export | Smart View | QExcel |
|---|---|---|---|
| Your own SQL over Fusion tables (AP, AR, SLA, PO, FA…) | Through a report built in Fusion | No: cubes and analyses | Yes, from Excel |
| Parameters from Excel cells | No: entered in the browser | For cube members | Yes |
| Result updated in your workbook | No: a new file each time | Yes | Yes |
| Combine with tables in your workbook | By hand | With Excel formulas | SQL on your computer |

## Download and install

Get the installer for macOS (Apple Silicon, macOS 13 or later) or Windows (10 or 11, 64-bit)
from [Releases](https://github.com/krokozyab/qexcel/releases).

- [Installing on macOS](docs/install-macos.md)
- [Installing on Windows](docs/install-windows.md)

QExcel runs entirely on your computer: the add-in pane in Excel and a small local service. No
server in the middle, no administrator rights. You also need Excel (Excel for Mac, or Excel from
Microsoft 365 on Windows), and Google Chrome, Microsoft Edge or Chromium for single sign-on.

> **Once in Fusion:** QExcel runs SQL through a small BI Publisher report that your Fusion
> administrator sets up once — see [ofquack](https://github.com/krokozyab/ofquack).

## Documentation

The [user guide](docs/user-guide.md) covers the QExcel pane: connections, parameters, the query
editor, calculations, Excel sources, formulas and what to do when something goes wrong.

## Author

QExcel is made by Sergey Rudenko.

QExcel is not affiliated with or endorsed by Oracle. Oracle, Oracle Fusion, BI Publisher, OTBI
and Smart View are trademarks of Oracle and/or its affiliates.
