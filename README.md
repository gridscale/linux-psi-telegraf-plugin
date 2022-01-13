# linux-psi-telegraf-plugin

## Usage

Copy the binary to some location telegraf can access it and adjust your telegraf.conf

```
[[inputs.execd]]
  command = ["/usr/local/bin/psi"]
  signal = "none"
```