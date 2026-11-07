# QExcel for Windows

QExcel brings Oracle Fusion Cloud data into Excel tables and formulas through
`ofexceld`, a small program that runs in the background on your PC. This package
installs it for your user account only; it needs no administrator rights.

Requirements: Windows 10 or 11 (64-bit), Excel from Microsoft 365, and Microsoft
Edge or Google Chrome to sign in to Oracle Fusion with single sign-on (not
needed for a connection that signs in with a username and password). Nothing
else: DuckDB and everything else ofexceld needs is inside it.

## Check the download

The release lists a SHA-256 for the archive. In PowerShell, in the folder you
downloaded to:

```powershell
Get-FileHash -Algorithm SHA256 .\qexcel-<version>-windows-x64.zip
Get-Content .\qexcel-<version>-windows-x64.zip.sha256
```

The two hashes must be the same. QExcel is not signed; this checksum is how you
know the file is the one that was published.

## Install

1. Extract the archive: right-click it, **Extract All**.
2. In the folder it makes, double-click **Install QExcel.cmd**.
3. QExcel is not signed, so Windows may warn first. On **Windows protected your
   PC**, choose **More info**, then **Run anyway**; on **Open File - Security
   Warning**, choose **Run**.
4. QExcel asks whether to install, then shows its steps in a console window.
   **Windows asks whether to install the QExcel certificate: answer Yes.**
5. A window says when it is done.

Then quit and reopen Excel; QExcel is under **Home > Add-ins** (or **Insert >
My Add-ins > Developer Add-ins**).

The same from PowerShell, in the extracted folder:

```powershell
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

`-ExecutionPolicy Bypass` lets this one script run although it is not signed;
it changes no setting; Install QExcel.cmd runs the same with `-Gui`, for the
windows. What the script does:

1. Copies `ofexceld.exe` to `%LOCALAPPDATA%\QExcel\bin` and unblocks it: Windows
   marks files from the internet, and SmartScreen would stop an unsigned one
   ("Windows protected your PC").
2. Writes `%LOCALAPPDATA%\QExcel\init.sql` with the local profile, unless you
   already have one.
3. Creates a local certificate authority and the daemon's certificate, and adds
   the authority to your trusted roots: **Windows asks whether to install it;
   answer Yes**. Excel talks to the daemon over HTTPS and refuses it otherwise.
   The authority can only vouch for `localhost` and `127.0.0.1`.
4. Registers the add-in's manifest where Excel loads developer add-ins.
5. Adds a scheduled task, "QExcel ofexceld", that starts the daemon when you log
   on, without a window, and starts it now. Where Windows lets only
   administrators create tasks, it uses your Run key instead.
   It also adds the **QExcel icon** in the notification area, started when you
   log on (your Run key), to see whether the daemon runs and start or stop it;
   `qexcel.cmd`, to do the same from a command prompt; and a Start menu folder
   **QExcel** with **QExcel Service** (the icon) and **Uninstall QExcel**.
6. Runs `ofexceld doctor`, which checks the certificates, the daemon and the
   connections.

The daemon keeps its database encrypted, with the key in Windows Credential
Manager under `QExcel/data.duckdb`.

To update, run Install QExcel.cmd from the newer package. Your connections and
data stay.

## Start, stop and check the QExcel service

The QExcel service (`ofexceld`) runs in the background: Excel's QExcel pane is
served by it and reads all its data from it. The pane's top line says whether
it runs ("QExcel service · running since 09:12"); click it to see more.

**If the QExcel pane does not open, or says "QExcel service stopped":** click
the QExcel icon in the notification area and choose **Start**. The pane comes
back by itself within a few seconds.

The icon is a green circle with a Q while the service runs, and a grey one while
it does not. Its menu (left or right click) says since when it runs and which
version, and has **Start**, **Stop**, **Restart**, **Open log** and **Uninstall
QExcel...**. **Hide until next sign-in** takes only the icon away; the service
keeps running. Windows may put a new icon under the arrow next to the clock:
drag it out to keep it in sight. No icon? **Start menu > QExcel > QExcel
Service**. The same from a command prompt:

```bat
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" status
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" start
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" stop
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" restart
"%LOCALAPPDATA%\QExcel\bin\qexcel.cmd" log
```

`stop` holds until you log on again, or until `start`.

## Check the installation

```powershell
& "$env:LOCALAPPDATA\QExcel\bin\ofexceld.exe" doctor
```

prints one line per check and says what to do about any that failed. The
daemon's log is `%LOCALAPPDATA%\QExcel\Logs\ofexceld.log`. Each day's log is
kept next to it as `ofexceld-YYYY-MM-DD.log` (a log past 10 MB starts another
the same day), and logs older than 30 days are deleted.

## Uninstall

**Start menu > QExcel > Uninstall QExcel**, or double-click **Uninstall
QExcel.cmd** in the extracted folder. It asks whether to delete your data too.
The same from PowerShell:

```powershell
powershell -ExecutionPolicy Bypass -File .\uninstall.ps1
```

removes the daemon, its scheduled task, the certificates (Windows asks before it
removes the authority) and the add-in. Your database and its key stay, so
installing again finds your connections and data; `-Purge` removes them too,
with the saved passwords. The data is a copy of what Oracle Fusion holds:
nothing is lost there.
