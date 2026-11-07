# QExcel for macOS

QExcel brings Oracle Fusion Cloud data into Excel tables and formulas through
`ofexceld`, a small daemon that runs on your Mac. This package installs it for
your user account only; it needs no administrator rights.

Requirements: a Mac with Apple Silicon, macOS 13 or later, Excel for Mac, and
Google Chrome, Microsoft Edge or Chromium to sign in to Oracle Fusion with single
sign-on (not needed for a connection that signs in with a username and
password). Nothing else: DuckDB and everything else ofexceld needs is inside it.

## Check the download

The release lists a SHA-256 for each file. In Terminal, in the folder you
downloaded to:

```sh
shasum -a 256 -c qexcel-<version>-macos-arm64.dmg.sha256
```

It must print `OK`. QExcel is not signed by Apple; this checksum is how you know
the file is the one that was published.

## Install

1. Open `qexcel-<version>-macos-arm64.dmg` and double-click **Install QExcel**.
2. QExcel is not signed, so macOS stops it the first time: "Apple could not
   verify “Install QExcel” is free of malware". Choose **Done**, then open
   **System Settings > Privacy & Security**, scroll down to "“Install QExcel”
   was blocked", choose **Open Anyway** and enter your password. (On macOS 13
   and 14: Control-click Install QExcel, choose **Open**, then **Open**.)
3. Install QExcel asks whether to install, then shows its progress. **macOS asks
   for your password**, to trust the certificate Excel reaches QExcel with.
4. A window says when it is done.

Then quit and reopen Excel, and open **Home > Add-ins > QExcel**.

The **QExcel icon** is now in the menu bar: it shows whether the QExcel service
runs, and has Start, Stop and Uninstall QExcel. **QExcel Service** (the icon's
app) and **Uninstall QExcel** are in the Applications folder of your home folder
(`~/Applications`).

The same from Terminal, with the `.tar.gz` of the same release:

```sh
tar -xzf qexcel-<version>-macos-arm64.tar.gz
cd qexcel-<version>-macos-arm64
./install.sh
```

What installing does:

1. Copies `ofexceld` to `~/Library/Application Support/QExcel/bin` and removes
   the quarantine mark the browser put on it. QExcel is not notarized, so
   without this macOS would refuse to run it ("cannot be opened because the
   developer cannot be verified").
2. Writes `~/Library/Application Support/QExcel/init.sql` with the local
   profile, unless you already have one.
3. Creates a local certificate authority and the daemon's certificate, and asks
   macOS to trust that authority for your account: **macOS asks for your
   password**. Excel talks to the daemon over HTTPS and refuses it otherwise.
   The authority can only vouch for `localhost` and `127.0.0.1`.
4. Puts the add-in's manifest where Excel for Mac looks for add-ins.
5. Adds a LaunchAgent, so the daemon starts when you log in, and starts it now;
   the **QExcel Service** app, which puts the QExcel icon in the menu bar, with
   a LaunchAgent of its own so it is there after every login; `qexcel`, to do
   the same from Terminal; and the **Uninstall QExcel** app.
6. Runs `ofexceld doctor`, which checks all of the above.

macOS tells you that "ofexceld" and "QExcel Service" can run in the background:
those are the two LaunchAgents. You can turn them off in System Settings >
General > Login Items & Extensions, but then the daemon does not start when you
log in, or the icon is not in the menu bar.

The first time the daemon starts, it creates its database and keeps the
database's encryption key in your login keychain. After an update, macOS asks
whether `ofexceld` may use that key: enter your password and choose **Always
Allow**. Until you answer, the daemon waits and Excel cannot reach it.

To update, open Install QExcel from the newer disk image (or run its
`./install.sh`). Your connections and data stay.

## Start, stop and check the QExcel service

The QExcel service (`ofexceld`) runs in the background: Excel's QExcel pane is
served by it and reads all its data from it. The pane's top line says whether
it runs ("QExcel service · running since 09:12"); click it to see more.

**If the QExcel pane does not open, or says "QExcel service stopped":** click
the QExcel icon in the menu bar and choose **Start**. The pane comes back by
itself within a few seconds.

The icon is a filled circle with a Q while the service runs, and an empty, grey
one while it does not. Its menu says since when it runs and which version, and
has **Start**, **Stop**, **Restart**, **Open Log** and **Uninstall QExcel…**.
**Hide Until Next Login** takes only the icon away; the service keeps running.
No icon? Open **QExcel Service** from your Applications folder, or search for it
with Spotlight. The same from Terminal:

```sh
qexcel status     # whether it runs, since when, which version
qexcel start
qexcel stop       # until you log in again, or qexcel start
qexcel restart
qexcel log        # the last lines of its log
```

`qexcel` is in `~/.local/bin`; if that is not on your `PATH`, use
`~/Library/Application\ Support/QExcel/bin/qexcel`.

## Check the installation

```sh
~/Library/Application\ Support/QExcel/bin/ofexceld doctor
```

prints one line per check and says what to do about any that failed. The
daemon's log is `~/Library/Logs/QExcel/ofexceld.log`. Each day's log is kept
next to it as `ofexceld-YYYY-MM-DD.log` (a log past 10 MB starts another the
same day), and logs older than 30 days are deleted.

## Uninstall

Open **Uninstall QExcel** in `~/Applications`. It asks whether to delete your
data too. The same from Terminal:

```sh
~/Library/Application\ Support/QExcel/bin/uninstall.sh
```

removes the daemon, the LaunchAgent, the certificates (macOS asks for your
password to stop trusting the authority) and the add-in manifest. Your database
and its key stay, so installing again finds your connections and data;
`uninstall.sh --purge` removes them too, with the saved passwords. The data is
a copy of what Oracle Fusion holds: nothing is lost there.
