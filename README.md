# QExcel

QExcel brings Oracle Fusion Cloud data into Excel. Write a query once — Oracle SQL that
runs in Fusion, or a calculation over data already fetched — and keep its result in a
worksheet table or a formula that you refresh with one click.

- **Queries in Fusion, results in Excel.** Oracle SQL runs in your Fusion instance with your
  own sign-in, so you see only the data your roles allow.
- **Tables and formulas.** Load a result into an Excel table, or read it with `OF.TABLE`
  and `OF.CALL` formulas; parameters can come from cells.
- **Calculations on your computer.** Combine fetched results and tables of your workbook
  with SQL that runs locally, without going back to Fusion.
- **Refresh the whole workbook** in the right order, with progress and Stop.
- **DEV, TEST and PROD** connections, each with its colour, wherever a connection is shown.

QExcel is a preview: version 0.1.0-dev.

## Download

Get the latest files from [Releases](https://github.com/krokozyab/qexcel/releases):

| Platform | File |
|---|---|
| macOS (Apple Silicon, macOS 13 or later) | `qexcel-<version>-macos-arm64.dmg` |
| Windows 10 or 11 (64-bit) | `qexcel-<version>-windows-x64.zip` |

Each file comes with its SHA-256. The installers are not signed by Apple or Microsoft:
the checksum is how you know a file is the one published here.

You also need Excel (Excel for Mac, or Excel from Microsoft 365 on Windows), and Google
Chrome, Microsoft Edge or Chromium to sign in to Oracle Fusion with single sign-on.

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
