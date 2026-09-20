# Stage maths

A hit factor tool for IPSC. Work out what it takes to beat a competitor,
keep live stage standings your match director isn't publishing, and build a
match leaderboard as you go.

Works offline once loaded, and installs to the home screen like an app.

## Install

Open the published URL on your phone.

- **Android** — open in Chrome and tap Install when prompted
- **iOS** — open in Safari, tap Share, then Add to Home Screen

After the first load it runs entirely from the phone, so it works at a range
with no signal.

## Stage tab

Enter any two of time, hit factor and points and the third is calculated.
Whichever you touched last is treated as known. The tap pad starts from a
clean run, so you only tap what was dropped.

Save each shooter and they join the stage standings, ranked by hit factor
with each shooter's percentage of the leader. Tap a row to edit it. Names
autocomplete from everyone entered across the match.

The target times underneath aim at the current stage leader — clean, one
charlie, one delta, one no-shoot, one mike, and a no-shoot plus mike — with
the time cost of each and a seconds-per-alpha pace figure on the clean row.

Stages are the chips along the top. Each keeps its own name and round count.

## Overall tab

Match points the way IPSC scores it: on each stage the best hit factor takes
all the points on offer and everyone else gets their share, summed across
stages. CSV export lives here.

## Timer tab

Connects to a shot timer over Web Bluetooth — Chrome on Android only, since
Safari has never supported it. Defaults to the Nordic UART Service, which is
what the AMG Lab Commander uses.

On the Commander, turn on **BLE push** in the advanced menu — hold "M" while
switching the timer on. Without it the timer won't send shot data. Only one
app can hold the connection, so disconnect PractiScore or nRF Connect first.

Shots appear as they're fired, with split and running total. The raw hex log
is still there underneath for working out a different timer.

### Using it from the Stage tab

You don't need to leave the Stage tab while a squad shoots. Under **Time
called**:

- **Auto** follows the timer, so Time called holds the string's total as it
  builds and is already right when the shooter finishes.
- **‹** and **›** step back and forward through the string. Use these when
  the timer picks up a neighbour's shot after the last round — step back to
  your real last shot and the time stops following. The next start beep
  releases that hold, so the following shooter auto-fills without re-arming.
  Typing a time by hand holds it the same way.
- The chip shows whether a timer is connected, and beside it the shot count
  and first shot time.

### AMG Lab Commander frame format

Worked out from captures, not from a published spec — AMG don't document
this. Fourteen bytes per notification on the TX characteristic:

```
01 TT SS SS CCCC PPPP FFFF MMMM 00 ??
 0  1  2  3  4 5  6 7  8 9 10 11 12 13
```

| Byte | Meaning | Confidence |
|------|---------|------------|
| 0 | `0x01`, constant | confirmed |
| 1 | `0x05` start beep, `0x03` shot | confirmed |
| 2 | shot number, from 1 | confirmed |
| 3 | shot number again on shot frames | see below |
| 4–5 | time from the beep, big-endian centiseconds | confirmed |
| 6–7 | split from the previous shot, big-endian centiseconds | confirmed |
| 8–9 | first shot time, constant across a string | confirmed |
| 10–11 | mirrored bytes 4–5 in every capture | unconfirmed |
| 12 | `0x00` in every capture | unconfirmed |
| 13 | varies per string | **not a footer — see below** |

So `01 03 02 02 01 6d 00 6a 01 03 01 6d 00 0b` is shot 2 at 3.65 s, a 1.06 s
split, first shot 2.59 s.

Only bytes 0, 1, 2 and 4–7 are parsed. The rest are read but not trusted.

**Two things that will catch you out.**

Byte 13 is *not* a footer. It was `0x03` throughout the first capture, so it
was validated as one — and the next string came through with `0x06`, so every
packet was rejected and the app sat waiting for a beep that had already
happened. It's constant within a string and changes between strings, so it
looks like a string counter, but three captures isn't proof. Don't constrain
it without evidence.

Byte 3 can't be told apart from byte 2 while shots arrive live, because the
current shot is also the last shot so far. It may be a total in a downloaded
string. Unused either way.

The start beep frame carries the *previous* string's summary in bytes 3 and
8–11 rather than zeros — byte 3 held the last string's shot count and bytes
8–9 its first shot time. Not relied on, but that's where a "last run" would
live if it's ever wanted.

**How this was verified**, in case a future capture disagrees. Two
independent checks, on every string: each shot's bytes 4–5 minus the previous
shot's bytes 4–5 equals its bytes 6–7 exactly; and every decoded time lands
20–70 ms before that packet's own arrival timestamp in the log, which is
consistent transmission latency and is what pins the units as centiseconds
and the origin as the beep.

Remote start is still unknown. The hex box writes arbitrary bytes to the RX
characteristic if you want to hunt for it.

## Scoring assumptions

Alpha 5. Minor: charlie 3, delta 1. Major: charlie 4, delta 2. A mike is the
10 point penalty plus the 5 point alpha you didn't get, so 15 down. A
no-shoot hit is 10 down on its own, or 25 if the round missed the target too.

Check these against your region's current rules before trusting them.

## Privacy

Everything is stored in the browser on your own device. No server, no
accounts, nothing leaves the phone.

## Publishing

Static files, no build step. GitHub Pages: Settings → Pages, deploy from
`main`, folder `/ (root)`.

Bump `CACHE` in `sw.js` and `BUILD` in `index.html` together on every change.
They're shown at the foot of the page, so you can tell at a glance whether a
phone is running the code you just shipped — worth checking before debugging
anything a phone reports.
