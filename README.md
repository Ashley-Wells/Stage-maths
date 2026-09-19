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

The log shows every packet as hex and ASCII. Parsing isn't written yet.

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
`main`, folder `/ (root)`. Bump `CACHE` in `sw.js` when you change files.
