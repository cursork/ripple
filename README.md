# ripple

RIDE + Perl = Ripple. A minimal client for Dyalog APL's [RIDE](https://github.com/Dyalog/ride) protocol.

Connects to a running Dyalog session, executes expressions, disconnects. One file, no dependencies beyond core Perl.

## Usage

```
ripple [-addr host:port] [-e 'expr'] ... [script.apl]
```

Default address is `localhost:4502`.

### Inline expressions

```sh
ripple -e "⎕←2+2"
ripple -e "⎕SE.Link.Create '#' '/app/src'" -e "#.Run 'Multi'"
```

### From a script file

```sh
ripple startup.apl
```

Where `startup.apl` contains one expression per line:

```apl
⎕SE.Link.Create 'owum' '/app/owum/APLSource'
'DRC' ⎕CY 'conga' ⋄ DRC.Init ''
#.owum.app.Run 'Multi'
```

### Remote session via SSH

```sh
ssh -L 14502:localhost:4502 user@server
ripple -addr localhost:14502 -e "⎕←⎕WA"
```

### Docker bootstrap

```sh
RIDE_INIT=SERVE:*:4502 $DYALOG/dyalog +s -q &
sleep 3
ripple startup.apl
wait
```

## Requirements

Perl 5.32+ (ships with RHEL 9, Debian 11, Ubuntu 22.04, and anything newer). Uses only `IO::Socket::INET` from core.

## Install

Copy `ripple` somewhere on your `$PATH`:

```sh
cp ripple /usr/local/bin/
```

## How it works

The RIDE protocol runs over raw TCP with a simple framing format: `[4-byte big-endian length][RIDE + payload]`. Ripple performs the protocol handshake (`SupportedProtocols`, `UsingProtocol`, `Identify`, `Connect`), waits for the interpreter to be ready, then sends `Execute` commands sequentially — waiting for each to complete before sending the next. Then it closes the connection, leaving the Dyalog session running.

## Licence

MIT
