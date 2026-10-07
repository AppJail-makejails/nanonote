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


### Attributes
#### User Attributes

| Name | Description |
| --- | --- |
| `<appname>:<profile>.jail.ephemeral` | Mark the jail as ephemeral. See `ephemeral` option in `appjail-quick(1)` for details.<br><br>Although the jail may be destroyed, its data is preserved in the user directory (see `${X11APPJAIL_USERDIR}` in `x11appjail-spec(5)`).<br>|

#### System Attributes

| Name | Description |
| --- | --- |
| `oci.from` | Location of OCI image.|
| `oci.tag` | OCI image tag.|
| `<appname>:<profile>.oci.from` | Same as `oci.from`, but by application. It takes precedence when defined.|
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
