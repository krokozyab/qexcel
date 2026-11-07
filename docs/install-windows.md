# Install QExcel on Windows

QExcel runs on your PC: a pane in Excel, and the QExcel service (`ofexceld`) that runs in the
background, signs in to Oracle Fusion and keeps the fetched data. It installs for your user
account only and needs no administrator rights.

**You need:**

- Windows 10 or 11, 64-bit;
- Excel from Microsoft 365;
- Microsoft Edge or Google Chrome to sign in to Oracle Fusion with single sign-on (not needed
  for a connection that signs in with a username and password).

Nothing else: DuckDB and everything else the service needs is inside it.

## Install

1. Download `qexcel-<version>-windows-x64.zip` from
   [Releases](https://github.com/krokozyab/qexcel/releases).
2. **Extract it first:** right-click the `.zip`, choose **Extract All**. Do not run the
   installer from inside the zip window.
3. In the extracted folder, double-click **Install QExcel.cmd**.
4. QExcel is not signed, so Windows may warn first:
   - on **Windows protected your PC**, choose **More info**, then **Run anyway**;
   - on **Open File - Security Warning**, choose **Run**.
5. QExcel asks whether to install, then shows its steps in a console window.
6. **Windows asks whether to install a certificate, "QExcel Local CA": answer Yes.** Excel
   reaches QExcel over HTTPS, and this certificate is good only for your PC (`localhost`).
7. A window says when it is done. Close Excel and open it again.

## Open QExcel in Excel

**Home > Add-ins**, then **QExcel**. If QExcel is not in the list, choose **More Add-ins**, open
**My Add-ins**, and under **Developer Add-ins** select **QExcel** and choose **Add**. The pane
opens on the right; the [user guide](https://github.com/krokozyab/qexcel/blob/main/docs/user-guide.md) takes it from there.

## Update

Extract the newer `.zip` and double-click its **Install QExcel.cmd**. Your connections, queries
and data stay.

## The QExcel service and its icon

The **QExcel icon** in the notification area, next to the clock, shows whether the service
runs: a green circle with a Q while it runs, a grey one while it does not. Windows may put a new
icon under the arrow next to the clock: drag it out to keep it in sight. Its menu (left or right
click) says since when it runs and which version, and has **Start**, **Stop**, **Restart**,
**Open log** and **Uninstall QExcel...**. **Hide until next sign-in** takes only the icon away;
the service keeps running.

- **The pane says "QExcel service stopped", or does not open:** click the QExcel icon and
  choose **Start**. The pane comes back by itself within a few seconds.
- **No icon?** **Start menu > QExcel > QExcel Service**.

## Uninstall

**Start menu > QExcel > Uninstall QExcel**, or double-click **Uninstall QExcel.cmd** in the
extracted folder. It asks whether to delete your data too. If you keep it, installing again
finds your connections and data. The data is a copy of what Oracle Fusion holds: nothing is
lost there.

## For IT and the command line

### Check the download

The archive has its SHA-256 next to it in the release. QExcel is not signed; the checksum is
how you know the file is the one that was published. In PowerShell, in the folder you
downloaded to:

```powershell
Get-FileHash -Algorithm SHA256 .\qexcel-<version>-windows-x64.zip
Get-Content .\qexcel-<version>-windows-x64.zip.sha256
```

The two hashes must be the same.

### Install from PowerShell

In the extracted folder:

```powershell
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

`-ExecutionPolicy Bypass` lets this one script run although it is not signed; it changes no
setting. Install QExcel.cmd runs the same with `-Gui`, for the windows.

### Start, stop and check

```bat
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" status
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" start
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" stop
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" restart
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" log
```

`stop` holds until you log on again, or until `start`.

```powershell
& "$env:LOCALAPPDATA\QExcel\bin\ofexceld.exe" doctor
```

prints one line per check and says what to do about any that failed. The service's log is
`%LOCALAPPDATA%\QExcel\Logs\ofexceld.log`. Each day's log is kept next to it as
`ofexceld-YYYY-MM-DD.log` (a log past 10 MB starts another the same day), and logs older than
30 days are deleted.

### Uninstall from PowerShell

```powershell
powershell -ExecutionPolicy Bypass -File .\uninstall.ps1
```

removes the service, its scheduled task, the certificates (Windows asks before it removes the
authority) and the add-in. The database and its key stay; `-Purge` removes them too, with the
saved passwords.

### What installing does

1. Copies `ofexceld.exe` to `%LOCALAPPDATA%\QExcel\bin` and unblocks it: Windows marks files
   from the internet, and SmartScreen would stop an unsigned one ("Windows protected your PC").
2. Writes `%LOCALAPPDATA%\QExcel\init.sql` with the local profile, unless you already have one.
3. Creates a local certificate authority, "QExcel Local CA", and the service's certificate, and
   adds the authority to your trusted roots. The authority can only vouch for `localhost` and
   `127.0.0.1`.
4. Registers the add-in's manifest where Excel loads developer add-ins.
5. Adds a scheduled task, "QExcel ofexceld", that starts the service when you log on, without a
   window, and starts it now. Where Windows lets only administrators create tasks, it uses your
   Run key instead. It also adds the **QExcel icon** in the notification area, started when you
   log on (your Run key); `qexcel.cmd`, for a command prompt; and a Start menu folder
   **QExcel** with **QExcel Service** (the icon) and **Uninstall QExcel**.
6. Runs `ofexceld doctor`, which checks the certificates, the service and the connections.

The service keeps its database encrypted, with the key in Windows Credential Manager under
`QExcel/data.duckdb`.
