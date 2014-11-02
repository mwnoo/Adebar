# Miscellaneous hints for Adebar usage

## General
*Adebar* only creates the scripts, it doesn't backup stuff itself. There are two
exceptions to this rule currently:

* it pulls the `packages.xml` (as it requires this to obtain additional details)
* it pulls the `wpa_supplicant.conf` directly.

Both actions are relative fast. But once you run the scripts to create the real
backups, things are different: especially pulling the "Shared Storage" (see
below) might take quite a while, as your SD cards might hold gigabytes of data.
At the same time, it puts quite a strain on your battery, as it needs to transfer
all those data. So make sure to have a power source connected to your device
during the backup process (you probably will have that, as ADB is mostly used
via USB – but you could also use "ADB wireless", and the *TiBu* stuff is using
WiFi as well – so I'd thought to better point this out).

### When running the real backup
* as pointed out above: have a power source connected
* with the "ADB Backups": always wait for the "complete" toast before confirming
  the next backup (or ADB might choke, backup won't continue)
* if ADB "chokes" (and the backup stalls): Check your backup directory for how
  far the process already went, adjust the script accordingly to start there,
  disconnect and reconnect your device, and re-start the modified script.


## Installation
Simply unpack into an empty directory, and run the script from there. No special
installation-steps required.


## Configuration
When running, *Adebar* first checks for an existing config file:

* if `config/` is a directory, and contains a file with the same name
  as specified by the first command-line parameter, this is used. This
  allows for device-specific config files if you have more than one
  device.
* otherwise, if `config/` is a directory, and contains a file named
  `default`, this will be used. You might want to have that for e.g.
  "guest devices" or as a default for possible "new ones"
* otherwise, if `config` is a file, that will be used. Makes it easier
  if you only have one device, and don't plan for more.

A sample configuration file is included as `doc/config.sample`, which you
simply can copy and adjust. Your config file has only to contain the
settings you wish to change; this will usually be the DEVICE_IP (which is not
set by default) and/or the STORAGE_BASE.

What the settings are standing for is:

* directory settings:
  * `STORAGE_BASE=`: that's the "base directory" everything goes below. Here
    the command-line provided `OUTDIR` will be appended to. By default, this
    variable is empty, so `OUTDIR` is relative to the execution path. If you
    e.g. set it as `STORAGE_PATH=/home/me/backups`, and pass `moto_x` as
    parameter to `adebar_cli`, your files should end up below
    `/home/me/backups/moto_x`.
  * `USERDIR="userApps"`: sub-directory where the backup scripts will place the
    ADB backups of your user-apps into (relative to where they're run)
  * `SYSDIR="sysApps"`: Similar, for the data backups of your system apps
* TiBu specific stuff:
  * `DEVICE_IP="192.168.101.111"`: IP address of your device
  * `TIBU_PORT="8080"`: port the *TiBu* web server listens on
  * `TIBU_SDINT="/storage/INTERNAL/Storage-ALL.zip"`: URL path of the internal SD
  * `TIBU_SDEXT="/storage/SAMSUNG_EXT_SD_CARD/Storage-ALL.zip":` URL path of the
    external SD
  * `TIBU_BACKUPS="/TitaniumBackup-ALL.zip"`: URL path to the *TiBu* backups
* Disable features (optional, by default they're all enabled; set a value to "0"
  in order to disable a feature
  * `MK_APPDISABLE`: the script to "freeze/disable" apps
  * `MK_USERBACKUP`/`MK_SYSBACKUP`: create the script to backup user apps+data /
    system app-data
  * `MK_COMPONENTS`: create simple list for "disabled components" (for scripts &
    docu see `MK_PKG_DATA`)
  * `PULL_SETTINGS`: pull settings/configs from the device (currently just
    `wpa_supplicant.conf`, but there might be more in the future)
  * `MK_TIBU`: create the script to pull stuff from the TiBu web server
  * `MK_PKG_DATA`: process packages.xml for further details (requires PHP-cli)
    This creates the *script* to deal with "disabled components", plus the
    corresponding markup document, plus the markup document holding details
    on installed apps and their sources
  * `MK_INSTALLLOC`: Deal with the default-install-location (where apps should
    be installed by default: 0=auto (system decides), 1=device, 2=sdcard).
    Creates a 1-liner script to set that again.
  * `MK_DEVICEINFO`: Create a (Markdown) document containing device information.
    Currently it lists the "device features" as returned by `pm list features`,
    plus some selected details from the `build.prop`; more might be added in
    the future.


## Shared Storage
This term refers to your SD card(s). *Adebar* offers two ways to retrieve data
from those:

* `com.android.sharedstoragebackup.ab` (for abbreviation issues, I'll refer to
  this as *SSB* from now on)
* [Titanium Backup](http://play.google.com/store/apps/details?id=com.keramidas.TitaniumBackup)
  (abbreviated as *TiBu* here)

Scripts generated by *Adebar* cover both, so it's up to you which to use.

### SSB
As you might have already guessed from the above, *SSB* is offered via ADB
without the need of additional apps. It is simply triggered when backing up
data from its package. These data include everything from your device's
SD cards – both internal (`/0`) and external (`/1`). Timestamps of files
and directories are kept, but most files seem to come as "0775" (i.e. the
execute flag is set for user and group) – a condition you might have seen
when accessing FAT formatted drives in general.

While I'd prefer this backup to the one *TiBu* creates, specifically for
the timestamps, on some devices it has a little draw-back: My *LG Optimus 4X HD*
e.g. mounts the external SD card *inside* the internal one (`/sdcard/external_sd`),
which results in its contents backed up twice: once as part of the internal SD
(in the `/0` subdirectory), and another time as external SD in the `/1`
subdirectory. This not only makes the resulting backup file unnecessary huge,
but also takes much longer to backup.

### TiBu
*TiBu* offers an internal web server, which you can start manually via its options
menu. If running, you can pull 3 kinds of backups via HTTP:

* contents of the internal SD card (http://<DEVICE_IP>:8080/storage/INTERNAL/Storage-ALL.zip)
* contents of the external SD card (http://<DEVICE_IP>:8080/storage/SAMSUNG_EXT_SD_CARD/Storage-ALL.zip)
* all your *TiBu* created backups (http://<DEVICE_IP>:8080/TitaniumBackup-ALL.zip).

In case of my above mentioned *LG Optimus 4X HD* I just had to pull the contents
of the internal card, and had them all: As noted, the external card is mounted
inside – and of course the backups reside on one of the two.

Disadvantage here is that *TiBu* touches the timestamps of all files, so the
original timestamps are lost: All files appear as if they would have been created
at the time the backup was made.


## Usage
`adebar-cli` requires at least one parameter, which will be used to

* detect a matching (device-specific) configuration file (see the "Configuration"
  section above)
* define the desired output-directory (residing below the configured
  `STORAGE_BASE`)

You can pass it an optional second parameter, which will then be appended
to the output directory name. This is especially useful if you want to keep
"historic scripts", e.g. to later detemine which apps where installed on your
device at a given time.

As this might be easier to understand with an example: Let's say you've got two devices, one Motorola and one HTC. For each of them, you wish different settings
to be applied. So for a "shortcut", you name the first "moto" and the second "htc".
For easier handling, you could do the following:

* create the `config` directory below the directory `adebar-cli` resides in
* copy `doc/config.sample` to `config/moto` resp. `config/htc`
* adjust the two files to your needs
* call `adebar-cli` with...
  * `./adebar-cli moto` to create the scripts in `$STORAGE_BASE/moto`
  * `./adebar-cli htc _20141101` to create them in `$STORAGE_BASE/htc_20141101`