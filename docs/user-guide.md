# QExcel user guide

QExcel brings Oracle Fusion Cloud data into Excel. You write a query once — Oracle SQL that
runs in Fusion, or a calculation over data already fetched — take its parameters from cells on
your sheet, and refresh its tables and formulas with one click.

This guide covers the QExcel pane in Excel. Installing QExcel is described in
[Installing on macOS](install-macos.md) and
[Installing on Windows](install-windows.md).

- [Quick start](#quick-start)
- [How QExcel works](#how-qexcel-works)
- [The pane](#the-pane)
- [Connections](#connections)
- [The Workbook tab](#the-workbook-tab)
- [Parameters from worksheet cells](#parameters-from-worksheet-cells)
- [Refreshing the workbook](#refreshing-the-workbook)
- [The Build tab](#the-build-tab)
- [The query editor](#the-query-editor)
- [Calculations: combining data on your computer](#calculations-combining-data-on-your-computer)
- [Excel sources: tables of the workbook in SQL](#excel-sources-tables-of-the-workbook-in-sql)
- [Your query library and the examples](#your-query-library-and-the-examples)
- [Formulas](#formulas)
- [Queries from someone else's workbook](#queries-from-someone-elses-workbook)
- [When something goes wrong](#when-something-goes-wrong)

## Quick start

### A colleague sent you a QExcel workbook

1. Open the workbook, then the pane: **Home > Add-ins > QExcel**.
2. If you have no connection yet, the line at the top of the Workbook tab asks for one: add it
   (see [Connections](#connections)) and **Sign in** with your own Fusion account.
3. **Needs attention** lists the workbook's queries to confirm. **Review** shows each query's
   text; **Trust** lets it run.
4. Change the report's parameters on the sheet — a period, a business unit — and click
   **Refresh workbook**.

### You are building a report

1. On the **Build** tab, **+ Add > From Fusion** — or **Use a saved query** to start from one of
   the examples (GL trial balance, AP open invoices, AR aging).
2. Write the Oracle SQL with parameters such as `:period`: `WHERE period_name = :period`.
3. Under **Parameters**, set `period` to **Cell** and click **Selected cell** with, say, `B1`
   selected. **Preview** shows the first 200 rows; then **Save**.
4. Select an empty cell and **Add to worksheet**. The new table takes `period` from `B1`: change
   `B1`, click **Refresh**, and the table follows.

## How QExcel works

QExcel has two parts:

- **The QExcel service** (`ofexceld`) runs in the background on your computer. It signs in to
  Fusion, runs the queries, and keeps what they fetched in an encrypted local database
  (DuckDB). Nothing leaves your computer except the queries sent to your own Fusion.
- **The QExcel pane** in Excel shows what the workbook is made of and lets you refresh and
  build it.

Because the fetched data is kept on your computer, formulas and calculations read it there:
recalculating the workbook does not go to Fusion. Fusion is asked only when you refresh.

A workbook is made of **steps**, of three kinds:

| Step | What it is | Where it runs |
|---|---|---|
| **From Fusion** | An Oracle SQL query | In Fusion, when you refresh; the result is kept on your computer |
| **From this workbook** (an *Excel source*) | A table or range of the workbook, read as data | On your computer |
| **Calculated from saved data** | DuckDB SQL joining or totalling the other steps | On your computer, without Fusion |

> **For example — reconciling payments with the bank:**
>
> 1. **From Fusion**, `ap_payments` fetches the supplier payments.
> 2. **From this workbook**, `bank_statement` reads the bank statement table on another sheet.
> 3. **Calculated from saved data**, `unmatched` joins the two and writes only the
>    differences to a worksheet table.

What a step writes or feeds is its **output**: a worksheet table, or formulas (`OF.TABLE`,
`OF.CALL`) that read it. A step with no output of its own — `ap_payments` above — is still
fetched when a calculation needs it.

The queries and where their tables take their parameters from are kept **in the workbook**:
whoever opens it gets the same steps and refreshes them with their own Fusion account.

## The pane

Open it with **Home > Add-ins > QExcel**.

- **The top line** says whether the QExcel service runs: "QExcel service · running since
  09:12". Click it for more, and for what to do when the service is stopped.
- **Workbook** — the results and refreshing them. This is where you start.
- **Build** — the steps the workbook is made of, and the query editor.

The pane remembers the tab you were on in each workbook; a new workbook opens on Workbook.

## Connections

A connection is one Fusion environment and how you sign in to it. The line at the top of the
Workbook tab shows the connections in short — "fusion_test · TEST · signed in" — and opens the
list when you click it, or by itself when one needs you to sign in.

**Add a connection** asks for:

- **Name** — how queries refer to it, for example `fusion_test`.
- **Environment** — DEV, TEST or PROD. It is shown in colour wherever data from it appears, so
  that PROD and TEST are never confused.
- **Fusion address** — the address of your Fusion, `https://….oraclecloud.com`.
- **SQL report path** — QExcel runs SQL in Fusion through a BI Publisher report deployed once in
  your instance: the same report as [ofjdbc](https://github.com/krokozyab/ofjdbc), so if you use
  ofjdbc, take the path from its JDBC URL. Otherwise un-archive `DM_ARB.xdm.catalog` and
  `RP_ARB.xdo.catalog` from [ofjdbc/otbireport](https://github.com/krokozyab/ofjdbc/tree/master/otbireport)
  into `/Shared Folders/Custom/Financials`. The form starts with that path,
  `/Custom/Financials/RP_ARB.xdo`.
- **Sign in with** — *Single sign-on, in a browser window*, or *Username and password*.

Then:

- **Test** runs a one-line query in Fusion through the connection.
- **Sign in** opens a browser window for single sign-on; finish signing in there. For a
  username and password, **Enter password** asks for it once, and QExcel keeps it in your
  computer's own password store (the macOS keychain, the Windows Credential Manager).
- **Use in formulas** makes it the connection formulas use when their query names none.
- **Sign out** ends the sign-in; **Forget password** removes the saved password from your
  computer.

## The Workbook tab

**This workbook** lists the results, one card per worksheet table. A query can have several
tables, each with its own parameters. Formula results are grouped by query.

A card shows:

- the query's title, and the table — `Sheet1!tblBalances ↗` takes you to it;
- where the data comes from: the Fusion environment ("PROD"), or "Calculated · TEST" for a
  calculation, "TEST + PROD" when its sources come from both;
- the parameters, next to how fresh it is: "ledger ← A1 · currency = USD";
- what happened, as it happened: "Updated today 09:45 · 1,234 rows", "Last refresh failed: …".
  QExcel says when it last wrote, not whether Fusion has changed since;
- the refresh button, and **⋯**:
  - **Go to worksheet**;
  - **Refresh from sources** — fetch again from Fusion, then calculate and write;
  - **Recalculate saved data** — calculate again from what was fetched, without Fusion;
  - **Edit query** — opens it on the Build tab, with a way back here;
  - **Output settings** — this table's connection, and where each parameter comes from;
  - **History** — the last refreshes;
  - **Remove worksheet output** — stops writing the table; the query stays.

**Sources and refresh details** on a card shows, one level at a time, what the result was
made from: each Fusion source with its environment and when it was fetched, each Excel source
with its range and when it was read, when the calculation ran and when the table was written.

**Needs attention** gathers what wants you to do something: queries to confirm, a connection on
another Fusion, tables whose refresh failed or was written part way, data the formulas asked
for, Excel sources that cannot be read or are missing. Each line has its action: Review, Show,
Retry write, Fetch, Use selection.

**Sources used by this workbook** lists the steps without an output of their own.

**Advanced & diagnostics**, at the bottom, has the service's maintenance and **Copy
diagnostics**, for a support request.

## Parameters from worksheet cells

A query's parameters are written in its SQL — `:period` in Oracle SQL, `$period` in a
calculation — and each worksheet table decides where its values come from:

- **a cell** — `period ← B1`: change the cell, refresh, and the table follows;
- **a fixed value** — `currency = USD`;
- **the query's default** — when the parameter has one.

In the editor, a parameter's **Value** or **Cell** is for **Preview**, and becomes the starting
point of a table you **Add to worksheet**. After that, each table keeps its own: change them with
**⋯ > Output settings** on its card. Saving the query never changes them.

That is what lets one query feed two tables: `gl_trial_balance` once with the period from `B1`,
and once with the prior period from `C1`, side by side.

A text parameter in a cell may need the cell formatted as text: see
[When something goes wrong](#when-something-goes-wrong).

## Refreshing the workbook

**Refresh workbook** first shows its plan:

- what it will fetch from Fusion, and on which connection;
- which Excel sources it will read;
- what it will calculate from saved data;
- which worksheet tables it will write;
- what it will skip, and why — for example a query not confirmed yet.

While it runs, you see each step's progress ("Refreshing the workbook: 2 of 5 steps over"), and
**Stop** ends it. Existing results stay until new data arrives. A step that fails does not stop
the others; what depends on it uses the data saved before, and says so.

**Retry failed** runs again only what failed: what succeeded is not fetched again.

A single card refreshes on its own with its button: a Fusion table from Fusion; a calculation
from its sources, or with **Recalculate saved data** only from what was already fetched.

## The Build tab

Build shows the workbook's steps in the order the data flows: **Sources** (from Fusion, from
this workbook) first, then **Calculated from saved data**, each after what it reads. A card
says:

- what the step is, and its environment;
- **Uses:** and **Used by:** — the other steps, each a link to its card;
- its outputs — worksheet tables ↗ — and the formulas that call it. "Used in at least 3
  formulas" counts the formulas that name the query as text, like `=OF.CALL("gl_balance")`;
- **Edit** (or **Open** for a query from your library), **Read now** for an Excel source.

The list also has queries from your library or the examples that the workbook needs: called
by its formulas, or read by one of its calculations.

**Find a step** filters the list. **+ Add**:

- **From Fusion** — a new Oracle SQL query;
- **From this workbook** — an Excel source: select the table or range first;
- **Calculate from saved data** — a new calculation;
- **Use a saved query** — a copy of a query from your library or the examples.

**⋯ > Remove from workbook** removes a step. It is refused, with what still needs it, while
another step, a worksheet table or a formula uses it.

## The query editor

The editor opens as a screen of its own, with **← Build** (or **← Workbook**) to go back. Its
head says the query's title and its state: **Saved**, **Not saved**, or **Not in this
workbook** for a query opened from your library. A banner says who else is changed by saving:
"Editing a query used by Bank reconciliation, the table tblRecon and at least 2 formulas:
saving changes them all".

From the top:

1. **Oracle | Local** — where the SQL runs. For Oracle, the connection: the query's default,
   and the one Preview uses. The line under it says where it runs: "Oracle · fusion_test ·
   TEST" or "Local · DuckDB".
2. **Name, Title, Description, Version.** Formulas and calculations call the query by its
   name: `=OF.TABLE("gl_balance")`. A name is lower-case letters, digits and `_`.
3. **Sources** — for a calculation; see the next section.
4. **The SQL.** Oracle SQL takes parameters as `:period`; DuckDB SQL as `$period`. Problems
   are shown under it as you type, with their line.
5. **Parameters.** Each has a **type**, a **default** and a **description**, saved with the
   query. A parameter without a default is required. A parameter the SQL uses but that is not
   declared is listed with **Declare**; **✕** removes one. **Value** or **Cell** is what Preview
   uses: see [Parameters from worksheet cells](#parameters-from-worksheet-cells).
6. **Preview** — the first 200 rows. It is not needed before saving.
7. **Save** — into this workbook.

> **Two things to know about Oracle SQL in QExcel**
>
> - **The time of day.** Oracle `DATE` columns arrive as a date only, without the time. To keep
>   the time, write `CAST(column AS TIMESTAMP)`. Columns such as `CREATION_DATE` and
>   `LAST_UPDATE_DATE` are timestamps in Fusion and keep their time as they are.
> - **HR tables.** HR tables are read through their secured views, which show only the people
>   your Fusion role lets you see: `PER_ALL_PEOPLE_F` is sent as `PER_PERSON_SECURED_LIST_V`,
>   `PER_ALL_ASSIGNMENTS_M` as `PER_ASSIGNMENT_SECURED_LIST_V`, and so on for eleven tables. A
>   view may lack a column the table has: the one for `CMP_SALARY` has no `ANNUAL_SALARY` or
>   `CURRENCY_CODE`, for example.

After saving, choose how the workbook uses the query:

| Choice | Best for | What it does |
|---|---|---|
| **Add to worksheet** | Registers and extracts, up to a million rows | Fetches every row into an Excel table at the selected cell. Refreshing keeps the table's style, your formatting, and your own columns to the right of the result |
| **Insert a formula** | One number in a report (`OF.CALL`), or a small result as a spilled array (`OF.TABLE`, up to 10,000 rows) | Writes `=OF.CALL(…)` in the selected cell when Preview showed a single value, `=OF.TABLE(…)` otherwise |
| **Use as a source** | A step that only feeds calculations | Writes nothing; calculations read it |

**⋯ > Save a copy to my library** keeps a copy for other workbooks; see below.

What the header of a query file holds besides these fields (`@column`, `@refresh`,
`@max_rows`, comments) is kept as written and named above the SQL. **Show the header in the
SQL** shows the whole text, header included, to change it by hand.

## Calculations: combining data on your computer

A calculation reads other steps — its **sources** — as tables, by their names:

```sql
SELECT a.account, a.balance, b.amount AS budget
FROM gl_trial_balance a
JOIN budget b USING (account)
```

The **Sources** list above the SQL holds them:

- **+ Add source** — a step of this workbook, of your library or the examples. A source that
  reads this calculation, directly or through others, cannot be added: that would be a loop.
- **In SQL: `gl_trial_balance` [Insert]** — the name the SQL reads it by; Insert puts it at
  the cursor. The first source of a new calculation replaces its placeholder with
  `SELECT * FROM gl_trial_balance`.
- **Parameters a source takes** are passed from the calculation: `period ← $period` by
  default. You can pass a fixed value instead — `'US Primary'`, a number, a date,
  `TRUE` — or another parameter, `$prior_period`. A parameter a source needs is added to the
  calculation.
- **⋯ > Columns** shows a source's columns; **Remove source** is refused while the SQL still
  reads it.
- When the SQL mentions a step that is not a source, QExcel suggests adding it rather than
  adding it on its own.

Refreshing a calculation fetches its Fusion sources and reads its Excel sources first; **Sources
and refresh details** on its card shows the whole chain.

## Excel sources: tables of the workbook in SQL

An Excel source is a table or range of the workbook that calculations read as data — a budget,
a mapping, a bank statement, a list of accounts.

1. Select the table (or a range with a header row) on the sheet.
2. Build > **+ Add > From this workbook**.
3. Check the name and the columns' types QExcel guessed, and change them if needed.
4. **Import**.

An Excel Table is followed as it grows; a range is always the same cells.
**Read from the worksheet before each calculation that uses it** keeps the calculation in step
with the sheet; without it, **Read now** reads it when you choose.

A calculation copied from your library may need an Excel source the workbook does not have:
Needs attention and the source's card say "needs Excel source budget". Select the table on a
sheet and choose **Use selection**: the import starts with that name.

## Your query library and the examples

**My query library** keeps queries on your computer for any workbook; the **examples** come
with QExcel.

- In the editor, **⋯ > Save a copy to my library**. The workbook keeps its own query; later
  changes to either do not change the other.
- Build > **+ Add > Use a saved query** lists your library and the examples. **Use** copies the
  query into the workbook, with the queries it reads, and opens the copy: "Added a copy to this
  workbook. Changes here do not change your library."
  - If the workbook already has a query of that name, QExcel asks before replacing it.
  - A query it reads that the workbook has already stays the workbook's own.
- A library query the workbook's formulas call is listed on Build with **⋯ > Copy into this
  workbook**.

Copy what a workbook needs into it before you send it to someone: their computer does not have
your library.

## Formulas

```
=OF.TABLE("gl_trial_balance", "US Primary", B1)
=OF.CALL("open_ap_total", B1)
=OF.ASOF("gl_trial_balance", "US Primary", B1)
=OF.STATUS()
```

| Formula | Returns |
|---|---|
| `OF.TABLE(query, args…)` | The result as an array that spills, headers in the first row. Up to 10,000 rows: a larger result belongs in a worksheet table |
| `OF.CALL(query, args…)` | A single value: a query whose result is one row and one column |
| `OF.ASOF([query, args…])` | When the data was last fetched; without arguments, the oldest refresh of the workbook |
| `OF.STATUS()` | `OK`, `NOT_SIGNED_IN`, `STALE` or `OFFLINE` |

- The arguments are the query's parameters in the order they are declared. An empty argument
  means the parameter's default: `=OF.TABLE("ar_aging", , B2)`.
- A formula reads data already fetched; it does not go to Fusion by itself. Data it asks for and
  does not find shows `#N/A`, and is listed under Needs attention as "Formulas need data", with
  **Fetch**. **Refresh workbook** fetches it too.
- A calculation (Local) runs on your computer when the formula is calculated.
- Dates come back as numbers: **Format formula dates** on the Build tab gives their cells a date
  format.
- A formula used the wrong way — SQL in place of a query's name, an unknown query, a wrong
  argument — shows `#VALUE!`, and the pane lists it under **Formulas with errors** with the
  reason.

## Queries from someone else's workbook

A workbook you receive may contain queries. They do not run until you confirm them: **This
workbook has 2 queries to confirm** shows each query's text, who saved it and when, where it
runs and what it reads, with **Trust** and **Don't trust**. Until then its tables and formulas
wait. A query you saved yourself, or one with the same text as in your library, is trusted
already.

If the workbook was made for another Fusion than your connection, its tables say so before they
run, with both addresses.

Advanced & diagnostics lists the trusted queries of the workbook; **Revoke** stops one until you
trust it again.

## When something goes wrong

- **"QExcel service stopped"** — click the QExcel icon in the menu bar (macOS) or the
  notification area (Windows) and choose **Start**. The pane comes back by itself.
- **"Not signed in"** — open the connections line and **Sign in**.
- **A table says "Last refresh failed"** — the reason is on the card; Needs attention has the
  same, with **Show**. Fix it and refresh again.
- **"Written part way"** — the fetched data is saved; **Retry write** writes it without Fusion.
- **A text parameter got a number** — Excel turned a period name typed into a cell, like
  `Mon-YY`, into a date. The card says "period expects text, but Sheet1!B1 holds a number":
  format the cell as text.
- **A formula shows `#VALUE!`** — the reason is under **Formulas with errors** in the pane.
- For a support request, **Advanced & diagnostics > Copy diagnostics**: versions, files,
  connections and recent errors. It names your connections and their Fusion addresses: read it
  before you send it.
