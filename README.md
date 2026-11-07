# QExcel

**Live Oracle Fusion Cloud data in Excel, refreshed with one click.**

If you work with Fusion data in Excel — in finance, procurement, HR or as a consultant —
you know the routine: run a report, export it, paste it into the workbook, fix the
formats, and do it all again next week. QExcel ends that. Write a query once, and its
result lives in your workbook as a table or a formula that refreshes whenever you need
fresh numbers.

- **Oracle SQL against Fusion, from Excel.** Queries run in your Fusion instance with your
  own sign-in, so you see exactly the data your roles allow — nothing more.
- **Tables and formulas.** Load a result into an Excel table, or pull values with `OF.TABLE`
  and `OF.CALL` formulas that take their parameters from cells.
- **A real database inside.** QExcel carries [DuckDB](https://duckdb.org), a fast
  analytical database, on your own computer. Fetched results stay there, so you can join
  them, total them and combine them with tables from your workbook in plain SQL, without
  asking Fusion again.
- **The whole workbook at once.** Refresh every query in the right order, with progress
  and Stop.
- **DEV, TEST and PROD** connections, each with its own colour, so you always know where
  the numbers come from.
- **macOS and Windows.** The same add-in and the same workbooks on Excel for Mac and Excel
  for Windows.

QExcel is a preview: version 0.1.0-dev.

## Download

Get the installer for macOS (Apple Silicon, macOS 13 or later) or Windows (10 or 11,
64-bit) from [Releases](https://github.com/krokozyab/qexcel/releases).

You need Excel (Excel for Mac, or Excel from Microsoft 365 on Windows), and Google Chrome,
Microsoft Edge or Chromium to sign in to Oracle Fusion with single sign-on.

## Install

- [Installing on macOS](docs/install-macos.md)
- [Installing on Windows](docs/install-windows.md)

Both install for your user account only and need no administrator rights.

## Use

The [user guide](docs/user-guide.md) covers the QExcel pane in Excel: connections, the
Workbook and Build tabs, the query editor, calculations, Excel sources, formulas and what to
do when something goes wrong.

## Author

QExcel is made by Sergey Rudenko.
