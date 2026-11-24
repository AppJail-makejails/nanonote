# nanonote

Nanonote is a minimalist note taking application.
It automatically saves anything you type. Being minimalist means it has no synchronisation, does not support multiple documents, images or any advanced formatting (the only formatting is highlighting URLs and Markdown-like headings).

github.com/agateau/nanonote

<img src="https://raw.githubusercontent.com/AppJail-makejails/nanonote/refs/heads/main/nanonote/nanonote.png" width="30%" height="auto" alt="nanonote logo">

## How to use this AppJail

### General usage

You can obtain the AppJail from the releases section of this repository. However, to go beyond a simple download and be able to update it conveniently from the console, the simplest complementary tool for our purposes is [sysutils/bin](https://freshports.org/sysutils/bin), a binary manager:

```console
$ doas pkg install -y bin
```

Install the latest version of this AppJail by running the following command:

```console
$ mkdir -p ~/bin
$ bin install https://github.com/appjail-makejails/nanonote
```

Or update it if it is already installed:

```console
$ bin update nanonote.appjail
```

Assuming `~/bin` is in your `PATH`, you can run the AppJail simply by using the following command:

```console
$ nanonote.appjail
```

Remember that when running an AppJail in portable mode, you must install the key used to verify the binary:

```console
$ cat << "EOF" | doas x11appjail trust dtxdf@disroot.org -
untrusted comment: dtxdf@disroot.org (x11appjail) public key
RWSZbdqRaZVSgICvhui+nrVbXbWw25jyZx/3lhaPzSmVi1Pgvk2DAB1h
EOF
$ x11appjail trusted
KEY                                                                   COMMENT
37e1a7da5478a29ec3d38ecb14919b107beab67cc0de0b473b5b018f018e1ccb.pub  dtxdf@disroot.org (x11appjail) public key
```

A system-wide installation requires only root access; the key is not necessary. However, it is strongly recommended to verify the binary before installation, making it necessary to install the key anyway.

```console
$ x11appjail verify ~/bin/nanonote.appjail
Signature Verified
$ doas ~/bin/nanonote.appjail --install
$ x11appjail run nanonote
```

An AppJail creates the jail only if it does not already exist or if the AppJail detects a valid change in its checksum (e.g.: after an update). This means that updates to the OCI image used by the AppJail are only checked at the creation time. If you need to update the OCI image, simply destroy the jail:

```console
$ x11appjail destroy-jail nanonote
```

Once you run the AppJail again, the OCI image is pulled again only if it is newer than the one on your system.

### D-Bus

To achieve better integration with the desktop environment, the host's D-Bus is shared via `xdg-dbus-proxy(1)`. By default, the system bus is not shared and the session bus is filtered. This happens transparently and you don't need to do anything. However, you can modify the behavior by creating user attributes that affect how AppJail interprets them when starting the D-Bus proxy.

```console
$ x11appjail attr put nanonote:default.dbus-user.talk.notifications "org.freedesktop.Notifications"
```

By defining this attribute, notifications will work as if they were running on your host. Note that this is just an example, and the application running within the jail may or may not use notifications.

Both the session bus and the system bus can be completely disabled by simply setting `<appname>:<profile>.dbus-user` or `<appname>:<profile>.dbus-system` to `none`. The system bus is usually disabled by default, as in most cases only the session bus is needed.

```console
$ x11appjail attr put nanonote:default.dbus-user none
```

Note that if you have already started the AppJail, in order for the changes to take effect, you must destroy the jail and then run the AppJail again.

```console
$ x11appjail destroy-jail nanonote
```
### Keyboard and mouse reference (Openbox)

The Openbox environment uses the following key and mouse bindings. They are
defined in Openbox's `rc.xml`.

> **Note:** `W` means the **Super** key (usually the Windows key), and `S`
> means **Shift**.

#### Keyboard bindings

| Binding | Action |
|---------|--------|
| `W-q` | Close the active window (`Close`) |
| `W-S-q` | Exit Openbox with confirmation (`Exit` with `prompt=yes`) |
| `W-f` | Toggle fullscreen (`ToggleFullscreen`) |
| `W-w` | Go to the next window (`NextWindow`) |
| `W-S-w` | Go to the previous window (`PreviousWindow`) |
| `W-l` | Show the combined client list menu (`ShowMenu > client-list-combined-menu`) |

#### Mouse bindings

| Context | Button / action | Resulting action |
|---------|-----------------|------------------|
| **Frame** (window frame) | `W-Left` (Press) | Focus and raise the window (`Focus`, `Raise`) |
| | `W-Left` (Click) | Unshade the window (`Unshade`) |
| | `W-Left` (Drag) | Move the window (`Move`) |
| | `W-Right` (Press) | Focus, raise, and unshade (`Focus`, `Raise`, `Unshade`) |
| | `W-Right` (Drag) | Resize the window (`Resize`) |
| | `W-Middle` (Press) | Lower the window to the bottom (`Lower`, `FocusToBottom`, `Unfocus`) |
| **Titlebar** | `Left` (Drag) | Move the window (`Move`) |
| | `Left` (DoubleClick) | Toggle maximize (`ToggleMaximize`) |
| | `Up` (Click) | Shade the window if it is not shaded (`Shade` conditional) |
| | `Down` (Click) | Unshade the window if it is shaded (`Unshade` conditional) |
| **Titlebar Top Right Bottom Left TLCorner TRCorner BRCorner BLCorner** | `Left` (Press) | Focus, raise, and unshade (`Focus`, `Raise`, `Unshade`) |
| | `Middle` (Press) | Lower to the bottom (`Lower`, `FocusToBottom`, `Unfocus`) |
| | `Right` (Press) | Focus, raise, and show the client menu (`ShowMenu > client-menu`) |
| **Top** (top edge) | `Left` (Drag) | Resize from the top edge (`Resize` edge `top`) |
| **Left** (left edge) | `Left` (Drag) | Resize from the left edge (`Resize` edge `left`) |
| **Right** (right edge) | `Left` (Drag) | Resize from the right edge (`Resize` edge `right`) |
| **Bottom** (bottom edge) | `Left` (Drag) | Resize from the bottom edge (`Resize` edge `bottom`) |
| | `Right` (Press) | Focus, raise, and show the client menu (`ShowMenu > client-menu`) |
| **TRCorner BRCorner TLCorner BLCorner** (corners) | `Left` (Press) | Focus, raise, and unshade (`Focus`, `Raise`, `Unshade`) |
| | `Left` (Drag) | Resize (`Resize`) |

> **Note:** `W-Left`, `W-Right`, and `W-Middle` mean that you must hold the
> **Super** key while clicking the left, right, or middle mouse button,
> respectively.


### Attributes
#### User Attributes

| Name | Description |
| --- | --- |
| `<appspec>.dbus-system` | Configure the system bus mode. There are three modes:<br><br>1. `filter`: Enables filtering.<br>2. `unfiltered`: Disables filtering. This mode is insecure and is recommended only for development.<br>3. `none`: Completely disables the session bus. This is the default mode unless explicitly stated otherwise.<br>|
| `<appspec>.dbus-system.<policy>.*` | Apply a policy or rule to the system bus, where `<type>` can be `own`, `call`, `talk`, `see`, and `broadcast`. See `xdg-dbus-proxy(1)` for details.|
| `<appspec>.dbus-user` | Configure the session bus mode. There are three modes:<br><br>1. `filter`: Enables filtering. This is the default mode unless explicitly stated otherwise.<br>2. `unfiltered`: Disables filtering. This mode is insecure and is recommended only for development.<br>3. `none`: Completely disables the session bus.<br>|
| `<appspec>.dbus-user.<policy>.*` | Apply a policy or rule to the session bus, where `<type>` can be `own`, `call`, `talk`, `see`, and `broadcast`. See `xdg-dbus-proxy(1)` for details.|
| `<appspec>.dpi` | Sets the resolution for all screens, in dots per inch. By default, no DPI is set.|
| `<appspec>.env.<label>.*` | User-defined custom environment variables. `<label>` is an arbitrary name, as it is not part of the environment. The value of the attribute will be the environment variable with its value in the format `<name>=<value>`.|
| `<appspec>.jail.ephemeral` | Mark the jail as ephemeral. See `ephemeral` option in `appjail-quick(1)` for details.<br><br>Although the jail may be destroyed, its data is preserved in the user directory (see `${X11APPJAIL_USERDIR}` in `x11appjail-spec(5)`).<br>|
| `<appspec>.nofullscreen` | Prevents devilspie2 from setting windows to full screen when they can be maximized.|
| `<appspec>.screen` | Screen size. If this is not set, allow the window to be resized.|

#### System Attributes

| Name | Description |
| --- | --- |
| `oci.from` | Location of OCI image.|
| `oci.pull-policy` | Pull policy. Default is `newer`.<br><br>See `--pull` in `buildah-from(1)` for details.<br>|
| `oci.tag` | OCI image tag.|
| `<appspec>.oci.from` | Same as `oci.from`, but by application. It takes precedence when defined.|
| `<appspec>.oci.pull-policy` | Same as `oci.pull-policy`, but by application. It takes precedence when defined.|
| `<appspec>.oci.tag` | Same as `oci.tag`, but by application. It takes precedence when defined.|
| `mount.system-fonts` | Read-only mounts the fonts system inside the jail, configure Fontconfig, and rebuild the font cache.|

## OCI Configuration

```yaml
build:
  variants:
    - tag: 15.1
      containerfile: Containerfile
      aliases: ["latest"]
      default: true
      args:
        FREEBSD_RELEASE: "15.1"
        NO_PKGCLEAN: "1"
      cache_dirs: ["pkgcache0:/var/cache/pkg"]
```

## Notes

1. Session bus is disabled by default.
