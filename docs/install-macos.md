# Install QExcel on macOS

QExcel runs on your Mac: a pane in Excel, and the QExcel service (`ofexceld`) that runs in the
background, signs in to Oracle Fusion and keeps the fetched data. It installs for your user
account only and needs no administrator rights.

**You need:**

- a Mac with Apple Silicon, macOS 13 or later;
- Excel for Mac;
- Google Chrome, Microsoft Edge or Chromium to sign in to Oracle Fusion with single sign-on
  (not needed for a connection that signs in with a username and password).

Nothing else: DuckDB and everything else the service needs is inside it.

## Install

1. Download `qexcel-<version>-macos-arm64.dmg` from
   [Releases](https://github.com/krokozyab/qexcel/releases), open it and double-click
   **Install QExcel**.
2. QExcel is not signed by Apple, so macOS stops it the first time ("Apple could not verify
   “Install QExcel” is free of malware"):
   - **macOS 15 or later:** choose **Done**, open **System Settings > Privacy & Security**,
     scroll down to "“Install QExcel” was blocked", choose **Open Anyway** and enter your
     password;
   - **macOS 13 or 14:** Control-click **Install QExcel**, choose **Open**, then **Open** again.
3. Install QExcel asks whether to install, then shows its progress. **macOS asks for your
   password** once more: Excel reaches QExcel over HTTPS, and this trusts QExcel's own
   certificate, which is good only for your Mac (`localhost`).
4. A window says when it is done. Quit Excel (`⌘Q`) and open it again.

macOS may tell you that "ofexceld" and "QExcel Service" can run in the background. Leave them
on: they start the service and its menu bar icon when you log in.

## Open QExcel in Excel

**Home > Add-ins > QExcel.** The pane opens on the right; the
[user guide](https://github.com/krokozyab/qexcel/blob/main/docs/user-guide.md) takes it from there.

## Update

Open **Install QExcel** from the newer disk image. Your connections, queries and data stay.

After an update, macOS asks once whether `ofexceld` may use the "QExcel" item in your login
keychain — the key of QExcel's encrypted database. Enter your password and choose **Always
Allow**. Until you answer, the service waits and the Excel pane cannot reach it.

## The QExcel service and its icon

The **QExcel icon** in the menu bar shows whether the service runs: a filled circle with a Q
while it runs, an empty grey one while it does not. Its menu says since when it runs and which
version, and has **Start**, **Stop**, **Restart**, **Open Log** and **Uninstall QExcel…**.
**Hide Until Next Login** takes only the icon away; the service keeps running.

- **The pane says "QExcel service stopped", or does not open:** click the QExcel icon and
  choose **Start**. The pane comes back by itself within a few seconds.
- **No icon?** Open **QExcel Service** from the Applications folder of your home folder
  (`~/Applications`), or find it with Spotlight.

## Uninstall

Open **Uninstall QExcel** in `~/Applications`, or choose **Uninstall QExcel…** in the icon's
menu. It asks whether to delete your data too. If you keep it, installing again finds your
connections and data. The data is a copy of what Oracle Fusion holds: nothing is lost there.

## For IT and the command line

### Check the download

Each file of a release has its SHA-256 next to it. QExcel is not signed by Apple; the checksum
is how you know a file is the one that was published. In Terminal, in the folder you
downloaded to:

```sh
shasum -a 256 -c qexcel-<version>-macos-arm64.dmg.sha256
```

It must print `OK`.

### Install from Terminal

With the `.tar.gz` of the same release:

```sh
tar -xzf qexcel-<version>-macos-arm64.tar.gz
cd qexcel-<version>-macos-arm64
./install.sh
```

### Start, stop and check

```sh
qexcel status     # whether it runs, since when, which version
qexcel start
qexcel stop       # until you log in again, or qexcel start
qexcel restart
qexcel log        # the last lines of its log
```

`qexcel` is in `~/.local/bin`; if that is not on your `PATH`, use
`~/Library/Application\ Support/QExcel/bin/qexcel`.

```sh
~/Library/Application\ Support/QExcel/bin/ofexceld doctor
```

prints one line per check and says what to do about any that failed. The service's log is
`~/Library/Logs/QExcel/ofexceld.log`. Each day's log is kept next to it as
`ofexceld-YYYY-MM-DD.log` (a log past 10 MB starts another the same day), and logs older than
30 days are deleted.

### Uninstall from Terminal

```sh
~/Library/Application\ Support/QExcel/bin/uninstall.sh
```

removes the service, its LaunchAgents, the certificates (macOS asks for your password to stop
trusting the authority) and the add-in manifest. The database and its key stay;
`uninstall.sh --purge` removes them too, with the saved passwords.

### What installing does

1. Copies `ofexceld` to `~/Library/Application Support/QExcel/bin` and removes the quarantine
   mark the browser put on it. QExcel is not notarized, so without this macOS would refuse to
   run it ("cannot be opened because the developer cannot be verified").
2. Writes `~/Library/Application Support/QExcel/init.sql` with the local profile, unless you
   already have one.
3. Creates a local certificate authority and the service's certificate, and asks macOS to trust
   that authority for your account. The authority can only vouch for `localhost` and
   `127.0.0.1`.
4. Puts the add-in's manifest where Excel for Mac looks for add-ins.
5. Adds a LaunchAgent, so the service starts when you log in, and starts it now; the **QExcel
   Service** app, which puts the icon in the menu bar, with a LaunchAgent of its own; `qexcel`,
   for Terminal; and the **Uninstall QExcel** app.
6. Runs `ofexceld doctor`, which checks all of the above.

The two LaunchAgents are what macOS lists in **System Settings > General > Login Items &
Extensions**. Turned off there, the service does not start when you log in, or the icon is not
in the menu bar.

The first time the service starts, it creates its database, encrypted, and keeps the key in
your login keychain.
