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


### Attributes
#### User Attributes

| Name | Description |
| --- | --- |
| `<appname>:<profile>.dbus-system` | Configure the system bus mode. There are three modes:<br><br>1. `filter`: Enables filtering.<br>2. `unfiltered`: Disables filtering. This mode is insecure and is recommended only for development.<br>3. `none`: Completely disables the session bus. This is the default mode unless explicitly stated otherwise.<br>|
| `<appname>:<profile>.dbus-system.<policy>.*` | Apply a policy or rule to the system bus, where `<type>` can be `own`, `call`, `talk`, `see`, and `broadcast`. See `xdg-dbus-proxy(1)` for details.|
| `<appname>:<profile>.dbus-user` | Configure the session bus mode. There are three modes:<br><br>1. `filter`: Enables filtering. This is the default mode unless explicitly stated otherwise.<br>2. `unfiltered`: Disables filtering. This mode is insecure and is recommended only for development.<br>3. `none`: Completely disables the session bus.<br>|
| `<appname>:<profile>.dbus-user.<policy>.*` | Apply a policy or rule to the session bus, where `<type>` can be `own`, `call`, `talk`, `see`, and `broadcast`. See `xdg-dbus-proxy(1)` for details.|
| `<appname>:<profile>.jail.ephemeral` | Mark the jail as ephemeral. See `ephemeral` option in `appjail-quick(1)` for details.<br><br>Although the jail may be destroyed, its data is preserved in the user directory (see `${X11APPJAIL_USERDIR}` in `x11appjail-spec(5)`).<br>|
| `<appname>:<profile>.dbus-user.ignore.broadcast.portal` | If this attribute exists, `org.freedesktop.portal.*=@/org/freedesktop/portal/*` will not be applied in the `broadcast` rule.|
| `<appname>:<profile>.dbus-user.ignore.call.portal` | If this attribute exists, `org.freedesktop.portal.*=*` will not be applied in the `call` rule.|

#### System Attributes

| Name | Description |
| --- | --- |
| `oci.from` | Location of OCI image.|
| `oci.pull-policy` | Pull policy. Default is `newer`.<br><br>See `--pull` in `buildah-from(1)` for details.<br>|
| `oci.tag` | OCI image tag.|
| `<appname>:<profile>.oci.from` | Same as `oci.from`, but by application. It takes precedence when defined.|
| `<appname>:<profile>.oci.pull-policy` | Same as `oci.pull-policy`, but by application. It takes precedence when defined.|
| `<appname>:<profile>.oci.tag` | Same as `oci.tag`, but by application. It takes precedence when defined.|
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
