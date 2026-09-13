# Aurora SEP userspace integration

These files connect the Apple SEP kernel driver to the shared APFS xART
gigalocker and the desktop fingerprint stack.

Install the xART mapper and driver service:

```sh
install -Dm755 aurora-sep-xart-map /usr/local/sbin/aurora-sep-xart-map
install -Dm755 load-driver /usr/local/sbin/aurora-sep-load
install -Dm644 aurora-sep-xart-map.service \
    /etc/systemd/system/aurora-sep-xart-map.service
install -Dm644 aurora-sep.service /etc/systemd/system/aurora-sep.service
systemctl daemon-reload
systemctl enable aurora-sep-xart-map.service aurora-sep.service
```

`aurora-sep-xart-map` requires `apfuse` at `/usr/local/libexec/apfuse`.
It validates the iBoot System Container and produces a writable device-mapper
view containing only the existing gigalocker extents. The kernel driver is not
loaded unless that shared mapping succeeds.

For fingerprint support, apply
`patches/libfprint-1.94.100-apple-sep.patch` to libfprint 1.94.100, build and
install libfprint, then install the fprintd device policy:

```sh
install -Dm644 fprintd-aurora.conf \
    /etc/systemd/system/fprintd.service.d/aurora.conf
systemctl daemon-reload
systemctl restart fprintd.service
```

On Omarchy, run `omarchy-apply-lock` once after enrollment. The Quickshell
lock screen then selects `omarchy-lock-fingerprint` automatically and accepts
Touch ID through fprintd. Hyprlock's separate fingerprint switch is not used
by the Quickshell lock screen.
