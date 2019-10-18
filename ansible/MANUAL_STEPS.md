# Manual steps required to run ansible on machines

## macOS
1. Update Sudoers file:

this requires `NOPASSWD` to be added to the sudoers file to enable elevation

`sudo visudo`
and change:
`%admin          ALL = (ALL) ALL`
to
```
%admin ALL=(ALL) NOPASSWD:ALL
jenkins ALL=(ALL) NOPASSWD: /usr/bin/xcode-select
```

2. Allow ssh access

```bash
sudo systemsetup -setremotelogin on
```
