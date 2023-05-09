# Podman snap package (WIP)

This repository contains a PoC (Prof of Concept) of a snap package for Podman.
Some Podman contributors already tried to kick that off but right now there is
no working solution.

The scope of this PoC at the moment is supporting only privileged mode
containers. In order to support unprivileged mode containers we would
need to run SUID binaries, such as the ones provided by `uidmap`, to
create user namespaces, and at the moment this does not seem to be
supported by snapd. It requires further investigation.

## Build

In a Ubuntu system, you have to have installed the snapcraft snap package:

```
$ sudo apt install snapd
$ sudo snap install snapcraft
```

Then run:

```
$ snapcraft
```

from the root of this repository. If you want to see all the steps executed by
`snapcraft` while building the package you can pass `-v` option. In case of a
build failure, you might want to get in the build environment to inspect what
is happening, to do that you can pass `--debug` option.

At the end, you will have a `podman_*.snap` file.

## Testing

In a Ubuntu virtual machine, you can install the generated
`podman_*.snap` file by running:

```
$ sudo snap install --devmode <path_to_snap_file>
```

You will then be able to check if it is working:

```
$ sudo podman help
```

Now, let's make the config files in `config` directory available to the snap
package:

```
$ sudo cp config/* /var/snap/podman/current/etc/containers/
```

## Apparmor issue

There is a Apparmor issue when trying to stop or remove containers:

```
$ sudo podman stop great_chebyshev
2023-05-08T21:14:28.693554Z: send signal to pidfd: Permission denied
Error: timed out waiting for file /run/libpod/exits/0cbe55bfa4102fc38260f60a504b6e7cd79eebc3d392e85ce3a2eaf048b585f4: internal libpod error
```

This is happening because the apparmor template from the `containers-common`
library is not adding the needed part.

From https://github.com/containers/common/blob/main/pkg/apparmor/apparmor_linux_template.go:

```
...

{{if ge .Version 208096}}
  # Allow signals from privileged profiles and from within the same profile
  signal (receive) peer=unconfined,
  signal (send,receive) peer={{.Name}},
{{end}}

...
```

The current workaround is adding the missing part to the apparmor profile:

```
# cat > /etc/apparmor.d/local/containers-default-0.51.1 << __EOF__
profile containers-default-0.51.1 flags=(attach_disconnected,mediate_deleted) {
  signal (receive) peer=unconfined,
  signal (send,receive) peer=containers-default-0.51.1,
}
__EOF__
# apparmor_parser -K -r -T /etc/apparmor.d/local/containers-default-0.51.1
```
