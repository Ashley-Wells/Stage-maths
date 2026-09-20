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

Shooters are picked from chips above the form, ordered by who has shot most
recently. **+ Shooter** adds someone new. Saving keeps the shooter selected
and resets the score to clean, so a practice squad doing run after run can
just keep tapping Save.

Standings read two ways. **All runs** lists every run, which is what you want
when the same people are shooting repeatedly and you're watching for
improvement. **Best each** collapses to one row per shooter — their best hit
factor, with run count and average underneath.

The target times underneath aim at the current stage leader — clean, one
charlie, one delta, one no-shoot, one mike, and a no-shoot plus mike — with
the time cost of each and a seconds-per-alpha pace figure on the clean row.

Stages are the chips along the top. Each keeps its own name and round count.

## Overall tab

Match points the way IPSC scores it: on each stage the best hit factor takes
all the points on offer and everyone else gets their share, summed across
stages. CSV export lives here.

A shooter scores once per stage, on their best run — several practice runs
don't earn several shares.

## Log tab

Saves a day's shooting as a session and keeps it to look back on.

**Save session** snapshots the whole board — every stage, its round count and
every run — under a name, defaulting to today's date. It then offers to clear
the runs so you can start fresh, keeping stage names and round counts since
those usually get reused. Tap a session to see each stage's runs ranked, plus
the session total. Sessions are stored under their own key, separate from the
live board.

### Sharing with training partners

**Share** on a saved session builds a link and hands it to whatever you send
things with. Open it on another phone and that app offers to import the
session; from then on it counts in their head to head too.

The session is packed into the part of the link after the `#`. Browsers never
send that to a web server, so the data travels through whichever app you sent
it with and never touches GitHub. There are no accounts and no server —
"friends" here just means people you send links to.

One person scorekeeps a day and shares it. Sessions carry an ID, so the same
link can be forwarded around and imported twice without anyone collecting
duplicates. An imported session is treated as untrusted: every field is
type-checked and capped, and names are only ever rendered as text.

Links are typically a few hundred characters. A very large session is refused
rather than sent as a link too long to survive a messaging app.

### Head to head

The long-term comparison, across every saved session.

Hit factor does not compare between different stages — a 24 round field
course and a 12 round short course aren't the same scale — so an average hit
factor over time would be meaningless if the stages change. Instead each
shooter is scored as a percentage of whoever won that stage, and those
percentages are averaged. Stage difficulty cancels out, so the number answers
"who is ahead, and am I closing the gap".

Two things it deliberately does: stages only one person shot are left out,
because being 100% of yourself says nothing; and where someone shot a stage
several times, their best run is used, which is how a match would score them.

Shooters are matched by roster ID, so renaming someone does not split their
history.

### Trend

The same percentage, session by session, one line per shooter. Shows whether
the gap is opening or closing. Tap or drag across it for a session's numbers.

Six shooters at most, taken in order of sessions attended. Past that, people
are left off and the note says how many — hues are assigned in a fixed order
and never recycled, because two shooters sharing a colour is worse than one
being absent.

### Backup

Sessions live in this browser and nowhere else, so clearing site data loses
them and a new phone starts empty. **Export file** writes the lot to a dated
`.json`; **Import file** merges one back, skipping anything already present by
ID. The same ID rule as share links, so a backup and a shared link can't
fight each other. Imported files are validated exactly like share links.

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

## Shooters

Every shooter is a roster record with a stable ID, and every run points at one.
The **Shooters** card on the Log tab lists them with their run counts.

- **Rename** changes the name everywhere at once, current board and saved
  sessions alike. Runs store the ID, not the name, so nothing is rewritten and
  history cannot split.
- **Merge** is for the same person recorded twice — "Dave" and "Dave S". Every
  run moves across and the duplicate is removed. This rewrites saved sessions,
  and cannot be undone.

Existing data was migrated automatically on first launch: every distinct name
across the board and all saved sessions became a roster record, and every run
was stamped with its ID. The match is now stored under `stage-maths:match:v2`
and the roster under `stage-maths:roster:v1`; the old `:v1` match key is still
read but never written again, so it survives as a fallback copy.

A session shared from another phone carries that phone's IDs, which mean
nothing here, so importing re-resolves every run against the local roster by
name — matching people you already have and adding the ones you don't.

## Colours

"Midnight": deep navy surfaces with a cyan accent, chosen over three other
candidates. Both modes are stepped separately rather than one being a flip of
the other, and the values were checked rather than eyeballed — body text
clears 4.5:1 on its surface and the accent clears 3:1.

The accent and the penalty colour sit on screen together (the leader row and
the primary button against the M and NS keys), so they have to be tellable
apart by someone colourblind, not just by hue. Cyan against red separates by
ΔE 17 dark and 18 light, against a target of 8. A warm accent fails this
badly — orange or brass beside red comes out at ΔE 2–3, effectively identical
under deuteranopia. Keep that in mind before making the accent warm.

The chart's series colours were re-validated against the new card surfaces;
on the dark card they now clear 3:1 outright.

App icons were not changed and still carry the old charcoal background. Per
the note above, rename to `-v3` rather than overwriting if they are ever
redrawn.

## Scoring assumptions

Alpha 5. Minor: charlie 3, delta 1. Major: charlie 4, delta 2. A mike is the
10 point penalty plus the 5 point alpha you didn't get, so 15 down. A
no-shoot hit is 10 down on its own, or 25 if the round missed the target too.

Check these against your region's current rules before trusting them.

## Privacy

Everything is stored in the browser on your own device. No server, no
accounts, nothing leaves the phone. That includes saved sessions — they live
in this browser only, so clearing site data loses them and they don't follow
you to another phone.

## Publishing

Static files, no build step. GitHub Pages: Settings → Pages, deploy from
`main`, folder `/ (root)`.

Bump `CACHE` in `sw.js` and `BUILD` in `index.html` together on every change.
They're shown at the foot of the page, so you can tell at a glance whether a
phone is running the code you just shipped — worth checking before debugging
anything a phone reports.
