> [!NOTE]  
> As of [Telegraf 1.30](https://github.com/influxdata/telegraf/releases/tag/v1.30.0) PSI (Pressure Stall Information) metrics will be collected without needing this plugin.

# linux-psi-telegraf-plugin

This telegraf plugin will push the pressure stall information (PSI) from the Linux Kernel to influx.

For more information about the PSI metrics read [facebooks documentation](https://facebookmicrosites.github.io/psi/).

The following metrics will be pushed to telegraf:

- avg10, avg60, avg300, total for
  - `/proc/pressure/cpu` (some)
  - `/proc/pressure/memory` (some and full)
  - `/proc/pressure/io` (some and full)

## Configuration

```
[[inputs.execd]]
  command = ["/usr/local/bin/psi"]
  signal = "none"
```

## Measurements & Fields

- pressureTotal
  - total
- pressure
  - avg10
  - avg60
  - avg300

## Tags

- All measurement have the following tags:
  - resource: one of `cpu`, `memory` or `io`
  - type: either `some` or `full`


## Example Output

```
pressureTotal,resource=cpu,type=some total=2302995431i 1642080601757211558
pressureTotal,resource=memory,type=some total=7459296i 1642080601757229544
pressureTotal,resource=io,type=some total=113202241i 1642080601757233510
pressureTotal,resource=memory,type=full total=3378729i 1642080601757326002
pressureTotal,resource=io,type=full total=90569049i 1642080601757329299
pressure,resource=cpu,type=some avg60=1.69,avg300=1.56,avg10=1.38 1642080601757332616
pressure,resource=memory,type=some avg10=0,avg60=0,avg300=0 1642080601757545386
pressure,resource=io,type=some avg10=0,avg60=0,avg300=0 1642080601757564081
pressure,resource=memory,type=full avg60=0,avg300=0,avg10=0 1642080601757567428
pressure,resource=io,type=full avg60=0,avg300=0,avg10=0 1642080601757625827
```

## Building the binary

1. Build the binary

```
go build -o psi cmd/main.go
```

2. Copy the binary to desired location

3. Adjust your telegraf.conf

```
[[inputs.execd]]
  command = ["/usr/local/bin/psi"]
  signal = "none"
```

4. Testing the output

On a linux machine the binary can be called and should output something like the [example output](#example-output).

## Usage with docker

If you use telegraf with docker you can build your own telegraf image with a multi stage build and just add the binray to keep the image size small:


```Dockerfile
# Get repo
FROM bitnami/git AS git-psi
WORKDIR /app
RUN git clone --depth 1 --branch v0.1.0 https://github.com/gridscale/linux-psi-telegraf-plugin.git linux-psi-telegraf-plugin

# Build psi binary
FROM golang:1.16-alpine AS binary-psi
WORKDIR /go/src/app/
COPY --from=git-psi /app/ ./
WORKDIR /go/src/app/linux-psi-telegraf-plugin
RUN go build -o psi cmd/main.go

# Build telegraf container
FROM telegraf:alpine

COPY --from=binary-psi /go/src/app/linux-psi-telegraf-plugin/psi /usr/local/bin/psi
```
