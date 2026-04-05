# ripple

My view:
_My attempt to upset every programming language community I belong to_

Claude's view:
_What I will say: the audience for it is maybe 50 people on earth. And half of them will dismiss it because it's Perl._

Claude is probably being kind on the number of interested...

RIDE + Perl = Ripple. A minimal <3KB (before minification and compression)
client for Dyalog APL's [RIDE](https://github.com/Dyalog/ride) protocol.

Connects to a running Dyalog session, executes expressions, disconnects. One file, no dependencies beyond core Perl.

**Why?! No seriously... WHY?!**

It may be the case that you want to administer a Dyalog session exposing the
RIDE port, but all you have is an impoverished and simple machine. That is, one
without a RIDE client of any sort.

On POSIX and POSIX-ish systems Perl is _ubiquitous_. A small review showed that
even RHEL 9 comes with Perl 5.32, which is nice and very modern.

The solution: a tiny RIDE implementation in Perl. As Perl is guaranteed on
everything but Windows (WSL exists though), and the author knows Perl, it was an
obvious choice. No installing an untrusted binary; no installation at all. One
small script.

With minification, it gets to <1KB. At this point it can be embedded or inlined
in to scripts.

## Usage

```
ripple [-addr host:port] [-e 'expr'] ...
```

Default address is `localhost:4502`.

### Inline expressions

```sh
$ ripple -e "⎕←2+2"
4
$ ripple -e "⎕SE.Link.Create '#' '/app/src'" -e "#.Run 'Multi'"
```

Output is printed to stdout. Errors (including APL errors) cause a non-zero exit code:

```sh
$ ripple -e "1÷0"; echo "exit: $?"
DOMAIN ERROR: Divide by zero
      1÷0
       ∧
exit: 1
```

### Remote session via SSH

Just an example from how I've used it:

```sh
ssh -L 14502:localhost:4502 user@server
ripple -addr localhost:14502 -e "⎕←⎕WA"
```

### From a machine without Dyalog

Ripple doesn't need Dyalog installed — just network access to a RIDE port. Copy
one file to a monitoring box, a CI runner, or a jump host and talk to Dyalog
remotely. Non-zero exit on APL errors means ripple works as a health probe:

```sh
ripple -e "Health.Check" || alert "Dyalog down"
```

### Bootstrap a headless Dyalog

Start Dyalog with RIDE, then send startup commands from a script:

```sh
RIDE_INIT=SERVE:*:4502 $DYALOG/dyalog +s -q &
sleep 3
ripple -e "⎕SE.Link.Create '#' '/app/src'" -e "#.Run 'Multi'"
```

Ripple disconnects. Dyalog keeps running. RIDE stays open for debugging.

## It fits in a QR code

The minified ripple is small enough to encode as a QR code. `minis/ripple-qr.png`
contains the entire RIDE client:

![ripple QR code](minis/ripple-qr.png)

```sh
curl -s https://raw.githubusercontent.com/cursork/ripple/main/minis/ripple-qr.png | zbarimg --raw -q - | perl - -e "⎕←747753"
```

Yes. This works. I am not sorry.

## Requirements

Perl 5.32+ (ships with RHEL 9, Debian 11, Ubuntu 22.04, and anything newer).
Uses only `IO::Socket::INET` from core.

## Install

There is no install. `ripple` just works. You may need to `chmod +x ripple`.

## How it works

The RIDE protocol runs over raw TCP with a simple framing format: `[4-byte big-endian length][RIDE + payload]`. Ripple performs the protocol handshake (`SupportedProtocols`, `UsingProtocol`, `Identify`, `Connect`), waits for the interpreter to be ready, then sends `Execute` commands sequentially — waiting for each to complete before sending the next. Then it closes the connection, leaving the Dyalog session running and further clients able to connect.

## What are these other files in `minis`?

Inside `minis`: `ripple-min`, `ripple-packed`, etc are experiments in getting silly about
minification - **ignore them**. I will decide how best to minify. `minify.pl` is the script used to create them.

If you refuse to ignore, ripple-min.gz is nice. Under 1KB and can be piped in
to perl (gzip is also POSIX standard):

```
gzip -dc ripple-min.gz | perl - -e "⎕←747753"
```

## Licence

MIT
