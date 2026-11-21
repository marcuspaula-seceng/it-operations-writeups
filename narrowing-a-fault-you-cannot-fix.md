---
title: Narrowing a fault you cannot fix
description: Six attempts to screenshot a Jenkins build console. One cause discarded, two hypotheses weakened, no root cause found.
---

# Narrowing a fault you cannot fix

*Home lab, 20-21 November 2026. Jenkins controller in a container, one Windows
build agent. Measured on my own machine; nothing simulated.*

I set out to screenshot a Jenkins build console. Six attempts later I had no
screenshot and one reproducible fault. This is what I did with it, because
**the useful skill is not fixing every fault - it is knowing what you have
ruled out when you stop.**

## The symptom

Every attempt to capture the console page ended the same way:

```
Page.captureScreenshot timed out after 30000ms on tab <id>.
The renderer may be frozen or unresponsive.
```

Six attempts. Three browser tabs opened fresh for the purpose. Two different
jobs. Two endpoints: the normal `/console` page and the plain-text
`/consoleText`. Waits of up to 13 seconds before capturing. Always 30 seconds,
always a timeout, always a tab that had to be thrown away.

## What I tested, and what each test actually settles

A suspicion is worth nothing until there is a test that could have proved it
right. It is worth less than nothing if the result is then overstated. Three
labels are used below, and they are not interchangeable:

- **Observation** - something recorded, with no claim attached.
- **Hypothesis weakened** - evidence points against it, but it is not excluded.
- **Cause discarded** - ruled out, within a scope that is stated.

| Suspicion | Test | Observation | What it settles |
|---|---|---|---|
| Jenkins is unwell | Load other pages in the same session | Dashboard, two job pages and the node page all rendered | **Hypothesis weakened, not excluded.** Other pages working says nothing about a fault specific to one endpoint. A per-endpoint fault inside Jenkins remains entirely possible. |
| The page must be heavy to trigger it | Measure both logs that froze | 31 lines each; 6,552 and 7,949 bytes | **One form discarded:** a large page is not *required*, since the fault appears at about the floor for a pipeline log. Whether weight contributes at all is untested. |
| Jenkins' own console JavaScript causes it | Capture `/consoleText`, served as `text/plain` | Froze the same way | **Cause discarded, scoped:** Jenkins' console scripts are not necessary for the freeze. This says nothing about browser-side code - the extension, the capture path, or the renderer itself. |
| A single poisoned tab | Open a fresh tab for each attempt | Reproduced on three | **Weakened at tab level only.** All three tabs shared one browser process, one profile and one extension, so browser- or profile-level state is not excluded at all. |

Four tests: one cause discarded inside a stated scope, one specific form of a
second discarded, and two hypotheses weakened but alive. **None of this
identifies a cause**, and counting the rows as if it did would be the same
mistake as the one corrected further down.

## Where I stopped

What is left is that the console text and the *Manage Jenkins > Nodes* page -
which failed the same way three times the day before - share something I have
not identified.

**I did not find the root cause, and I am not going to imply that I did.** What
I reached is a fault anyone can reproduce in about three minutes, with the
field narrowed and the narrowing written down honestly, so the next person does
not repeat my first hour.

Known limits of this account:

- One machine, one browser, one browser profile, one Jenkins version. The
  repeated attempts shared all of them, so nothing here separates a
  Jenkins-side cause from a browser-side one.
- The two affected pages may share a cause or may be two faults that look
  alike. I have no evidence either way.
- "About three minutes to reproduce" is my own timing on this machine, not a
  benchmark.

## The number that made a correction necessary

The size row above exists because of an error I had already published.

My first write-up claimed the fault was independent of page size because "an
11-line log froze like a 900-line one". **That comparison never happened.** When
I went back and measured, both logs were 31 lines and about 7 KB. I had
compared a cleaned-up excerpt against an impression of the other file.

The conclusion survived. The reasoning behind it did not, and the two are not
the same thing. The original now carries a dated correction beside the table.

An unmeasured number that happens to point the right way is still an unmeasured
number. It was wrong when it was convenient, which is exactly when it is
hardest to catch.

## What shipped instead of the screenshots

The console logs went out as text, with the monitored site address and the
storage host name replaced by placeholders. The evidence table marks those rows
as partial rather than done: the content is there, the proof that it came from
the Jenkins interface is not. A reader can tell the difference at a glance,
which is the point.

Two other measurements from the same session:

- **79 builds, one per hour, unattended since 17 November.** A schedule in a
  config file is a claim; 79 timestamps on the hour are evidence.
- **Pester 6.2.0: 20 tests, 20 passed, 0 failed, 1.14 s.**

## The transferable part

Three habits did the work, and none of them are about Jenkins.

1. **Make each suspicion falsifiable before testing it.** "The page is too
   heavy" becomes useful only when it turns into "then a 7 KB page should
   render" - a sentence that can come back false.
2. **Record the stopping point as a finding.** An undiagnosed fault with a
   narrowed field and honest labels on the narrowing is a handover. An undiagnosed fault with no notes is an hour
   the next person spends again.
3. **Re-measure the number that supports you.** The claim that was wrong here
   was the one I had least reason to question, because it agreed with me.

---

Marcus Paula. Home lab write-up. No employer data is published.
