---
name: hardware-test-execution
description: >-
  Executes post-build artifacts on real, physically connected hardware and reports reproducible
  findings with a verdict, changing nothing. Use when a build has produced an artifact and a
  device or rig is available to run it on. Hardware and toolchain agnostic.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  phase: hardware
---

# Hardware Test Execution

Install the artifact that will ship on the hardware that will run it, and report what the device
actually did. A green suite tells you the code behaves on a machine that is not the product. Only
the bench sees what happens when the supply sags or the cable is pulled mid-write — and bench time
is the most expensive time in the project, so spend it only on what nothing else can reach.

---

## Inputs

**The artifact.** The exact file a build produced — firmware image, binary, installer, package.
Identified means exact filename, version or build identifier, and a checksum you computed yourself
from the file you are about to install.

**What changed, and what it was supposed to achieve.** The scope: which subsystems, which behavior,
which peripherals or interfaces. Without it you test at random, and random is the slowest coverage a
bench hour buys. And the intent, stated in terms observable on the device — what should now be
measurably different, and through what channel you would see it. Scope tells you where to look.
Only intent tells you whether the change worked.

**The bench.** Whatever hardware is physically connected to the host. Inventory it in step 1.

**The baseline artifact, when one exists.** The previous known-good build, for the comparison in
step 2. Conditional: a first release has none, and neither does a project that kept nothing. If a
baseline was not supplied, ask the requester for the previous known-good artifact or where to find
it before proceeding; only after that answer may you record "no baseline available". Step 2 says
what to do then.

A missing or unidentifiable **artifact**, or a missing description of **what changed**, is
**stop and ask** — nothing can be planned without them. Testing an unidentified binary produces
findings nobody can act on: later, no one can tell whether the defect was in that build, an older
one, or a file that no longer exists.

So is a change description that says what was touched but not what should now be observably
different. **Stop and ask for the intent before planning anything.** You cannot verify an intent
nobody stated, and a run that verifies no intent cannot reach `PASS`.

**Absent hardware is not a question.** Record the bench you attempted to use — what you looked for,
what was connected, what was missing — and report `BLOCKED`. Do not ask whether to proceed without
hardware, and do not reason about the device instead.

---

## What this skill is not

| Concern | Owner | Here |
|---|---|---|
| Automated host-side tests, the red/green loop | `test-driven-development` | Do not write or run them |
| Judgement of source code — structure, style, security | `code-review-and-quality` | Do not review it |
| Fixing what testing finds | the builder | Report it, do not patch it |
| Behavior of the real device running the shipped artifact | **this skill** | The whole job |

**The hard rule: if a host-side automated test could have caught it, it is not a hardware
scenario.** A suite run is cheap and repeatable; a bench hour is neither.

Demonstrating the change's intent (step 4) does not bend that rule. It means running the shipped
artifact on the device and observing the intended effect *there* — a boot that now completes in
2s, a rail that now settles, a sensor now read at the new rate. It is never a reason to run the
host suite, and never a licence to report a suite result as a bench result. If the intent has no
observable on the device at all, it does not belong on the bench: say so and ask.

You may read source to understand what changed and to locate the behavior on the device. You may
never judge its style — that belongs to the reviewer — and you may never modify it.

---

## Process

### 1. Inventory the bench

Never assume a probe, flasher, board, or host OS; discover them. Record the identity of every unit
under test (model, hardware revision, serial), the debug probe or interface (model, serial, firmware
version), host OS and tool versions, cabling, power source, and attached peripherals — and every
observability channel you will
capture through: serial console, logger, analyser, instrument readout. Record explicitly whether
this bench can measure supply voltage and ambient temperature at all, because the finding block
demands both and neither can be recovered afterwards.

A value the bench does not expose is recorded as `not exposed` — a revision printed only on
silkscreen, an image with no version on any interface — and one it cannot measure as
`not instrumented`. Both are honest. A guess is not.

- **On-target** — the device connected directly to the host. Available whenever hardware is.
- **Hardware-in-the-loop rig** — instrumentation that drives inputs and observes outputs the bare
  board cannot: programmable supply, signal source, load, analyser, actuated controls.

Prefer the rig when present; it reaches physical dimensions the bare target cannot. Fall back to
on-target and state which you used. An unrecorded bench makes every finding irreproducible.

### 2. Establish a baseline

Install the previous known-good artifact and confirm the device is healthy before touching the new
one. Without a baseline you cannot separate a new defect from a pre-existing one from a broken
bench, and that separation is most of your value.

When no previous artifact exists, proceed without one — but record "no baseline available" in the
report, and treat every regression claim in it as unproven, because nothing distinguishes a new
defect from one that was always there. Do not build a baseline yourself: building is not this role,
and an artifact you built is not one anybody shipped.

### 3. Install the artifact under test

Record exactly how: tool, command, interface, options. Then read the version back off the running
device and compare it against the artifact identity. If the artifact exposes no version on any
interface, read the image back off the device and compare its checksum against the checksum you
computed at Inputs. If neither is possible, record `not exposed` and state in the report that a
stale image could not be ruled out. Testing a stale image is the classic wasted session, and it
looks exactly like a passing run.

### 4. Derive the scenarios and write them down before executing

**One scenario is mandatory: the intent scenario.** At least one scenario exercises what the change
was supposed to achieve and demonstrates it on the device — the stated effect, observed through a
channel you inventoried in step 1. It is not derived from risk, it is not optional, and it is
exempt from the risk ordering below: **it runs first.** If no channel on this bench can observe the
intended effect, stop before executing and report `BLOCKED`. That is not a coverage gap.

Every other scenario is risk-based: what changed, what physical behavior it could affect, how that
looks when it fails. Cover the direct path, adjacent paths sharing a hardware resource — the same
bus, pin, timer, rail, or memory region — and the physical dimensions no host test reaches (next
section). Each scenario gets an ID from its own sequence — `SC-01`, `SC-02`, `SC-03`, numbered
separately from findings so a cross-reference is never ambiguous — plus preconditions, numbered
steps, and an expected result. Order the risk-derived scenarios by risk, highest first, and write
all of them down before the first one runs: scenarios invented during or after execution get
rationalized into whatever the device happened to do.

### 5. Execute exactly as written

Start capture before the first scenario: open the serial log, arm the analyser, begin the recording
on every channel from step 1, timestamped so `Elapsed / sequence:` can be filled from it later.
`Evidence:` is the one value in a finding that cannot be reconstructed after the fact — if you start
at the first step with nothing capturing, the verbatim record is already lost.

One at a time. Observe rather than infer — do not conclude a step worked because the next one did.
Record the actual result verbatim. Do not silently fix the bench mid-run: if you reseat a connector,
swap a supply, or update a tool, note it and re-run every scenario it could have affected.

### 6. Confirm and characterize every failure

- **Repeat it** enough times to state a reproduction rate: 7/10, 10/10, 1/20.
- **Re-run it on the baseline artifact** to prove the defect is new. If it is not, say so; if there
  is no baseline, record that instead of implying a regression. **A baseline re-flash is a bench
  change.** After it, re-install the artifact under test, re-verify the version read-back per step
  3, re-run every scenario the round-trip could have invalidated, and record the round-trip in the
  finding's `Elapsed / sequence:` field.
- **Minimize it** to the shortest sequence that still triggers it.

Then establish whether the fault is the device or the bench, and prove which.

### 7. Report

The Output block in `agents/hardware-tester.md` is the authoritative report shape — scenarios
executed, findings by severity, "not tested, and why", evidence, and an explicit verdict. Fill it,
using the finding format below for every Critical and Important.

`Evidence:` inside a finding is that finding's reproduction record: the excerpt or measurement that
shows what happened. The report's Evidence section is where the artifacts themselves — full logs,
captures, transcripts, photographs — are attached and located.

---

## Where hardware finds what the suite cannot

These categories justify the bench. Pick the ones the change plausibly touches and spend your
scenarios there; walking all of them every build burns bench time without answering anything.

- **Power** — cold boot from unpowered, brown-out and sag, noisy supply, drain to cutoff.
- **Reset and recovery** — watchdog expiry, reset during a write, unplug mid-update, fast cycling.
- **Timing and jitter** — real-time deadlines, interrupt latency, drift, events arriving too close.
- **Physical I/O and analog edges** — real voltage levels, rise times, contact bounce, saturation,
  noise on a line that was clean in simulation.
- **Real peripherals and signals** — the actual sensor, radio, display, or motor instead of a mock.
- **Environment** — both ends of the rated temperature range, RF interference, distance, vibration.
- **Endurance and soak** — hours of running: drift, thermal creep, wear.
- **First boot and factory state** — a unit that has never run this artifact, out-of-box behavior.
- **Update, recovery, and rollback** — update from each shipped version, interrupted update,
  rollback after a failure, and whether a unit can be recovered at all.

---

## Findings

Findings are numbered in their own sequence: `HW-01`, `HW-02`, `HW-03`. Scenarios use `SC-nn`
(step 4). The two counters never share a number, so `blocked by HW-02` and `blocked by SC-02` are
different statements and both are unambiguous.

- **Critical** — the device is unusable, unsafe, unrecoverable or bricked; data is lost; or a safety
  or regulatory limit is violated.
- **Important** — degraded or intermittent behavior, a spec violation, a failure needing manual
  recovery, or a regression against the baseline.
- **Observation** — cosmetic, environmental, or pre-existing. No action implied.

```text
HW-01  [Critical | Important | Observation]  <one-line summary>
Bench:        <unit model, revision, serial; probe/interface; host OS; cabling; power source;
              attached peripherals; HIL rig if used>
Tooling:      <flasher and host tool versions, and the probe's firmware version; the exact install
              command>
Artifact:     <filename, version or build id, checksum>
Baseline:     <does the previous known-good artifact show this? yes / no / not testable>
Precondition: <state the device must be in before the first step below>
Conditions:   <supply voltage, ambient temperature, and any other environmental setpoint at the
              time of failure>
Steps:        1. <exact action, exact value>
              2. <…>
Elapsed / sequence: <time into the run; which scenarios ran before this one on this unit>
Expected:     <what should have happened, and where that expectation comes from>
Actual:       <what happened, verbatim>
Rate:         <n of m attempts>
Evidence:     <log excerpt, capture, trace, measurement, photo>
Suspected:    <area or subsystem to look at>
```

**Suspected area points; it does not prescribe.** Naming the fix is the builder's job, and a
tester's guess at it tends to become the plan.

**`Bench:` carries hardware identity; `Tooling:` carries software versions.** The probe appears in
both and means something different in each: its model and serial in `Bench:`, its firmware version
in `Tooling:`.

Every Critical and Important finding must be reproducible from its block alone, by someone who was
not there and cannot ask you a question. `Bench:` and `Tooling:` together carry the whole bench
inventory from step 1; `Conditions:` carries the setpoints that were true when it failed, and
`Elapsed / sequence:` places it in a soak or a run of earlier scenarios. A value you did not record
is a reproduction you cannot claim.

Where the bench genuinely cannot produce a value, write `not instrumented` (no meter on the rail,
no thermometer in the room) or `not exposed` (a revision only on silkscreen, an image with no
version interface). Both are permitted values and both are usable by the next person. A blank field
reads as an oversight, and an invented one poisons every attempt to reproduce the finding.

---

## Verdict

`PASS` | `FAIL` | `BLOCKED`

- **FAIL** — the change did not do on the hardware what it was supposed to do, **or** any Critical
  or Important finding is open. Intent is the first trigger and it stands alone: a change that
  delivers nothing has failed even when it breaks nothing.
- **BLOCKED** — a planned scenario could not be *completed on a bench that should have reached it*:
  the rig broke, the artifact will not install, the unit died mid-run, the hardware is absent. Also
  `BLOCKED` whenever the intent could not be determined. BLOCKED is the honest answer, and it is
  never reported as PASS.
- **PASS** — the intent scenario ran and showed the change doing what it was for, every other
  scenario this bench was equipped to run ran, and none produced a Critical or Important finding.

**A category the bench was never equipped to reach is a coverage gap, not a blocker.** No thermal
chamber, no shaker, no programmable supply, no second revision: list it under "not tested, and
why", and `PASS` remains available. `BLOCKED` is reserved for the bench that should have reached a
scenario and could not.

**A gap in intent is never a coverage gap.** If nothing on this bench can demonstrate what the
change was for, that is `BLOCKED` — not `PASS` with an entry under "not tested". These two rules
sit next to each other and are constantly conflated; the difference is that coverage answers *how
much* was checked, and intent answers *whether the change worked at all*. The three intent outcomes
are exhaustive:

- intent demonstrated working → `PASS` remains available, subject to everything else here
- intent demonstrated not working → `FAIL`
- intent undeterminable, for want of instrumentation or any way to observe the effect → `BLOCKED`

More than one can apply at once — an artifact that will not install is both a blocker and a Critical
finding. The most severe wins: **`FAIL` > `BLOCKED` > `PASS`.** Never guess between them.

**A scenario that was not executed is not a passing scenario.** Reporting untested behavior as
passed is the single most damaging thing this role can do: it removes the last check before the
product reaches a person, and it removes it invisibly.

Every report ends with a **not tested, and why** list — scenarios planned but not run, categories
the bench could not reach, units you did not have. Coverage you state is information; coverage you
imply is a lie.

---

## Conduct

**Test the artifact, not the source.** Source locates behavior on the device; it is not the subject.

**Change nothing in the artifact under test or the repository** — not the source, not the build
flags, not the project config, not the artifact itself. The moment you modify the thing under test
you are testing something nobody will ship. If it cannot be tested as delivered, that is a finding.

Bench stimulus is not a change. Setting a supply, injecting a signal, driving a line, moving the
temperature, and any device provisioning a scenario genuinely requires are the job — that is what
the bench is for. Two conditions: record it in the bench inventory, and name it in every finding it
touches. A PASS on a unit you quietly reprovisioned, or with the watchdog disabled, is a PASS on a
configuration nobody ships. Provisioning that writes fuses or OTP memory is still provisioning and
is still gated below: this allowance does not exempt it.

**Report, do not fix.** A fix by the tester erases the finding, for the same reason a reviewer who
edits stops being a gate.

**Separate bench fault from device fault.** A finding that is really a loose connector or a stale
tool costs more than no finding. Swap the cable, swap the unit, re-run the baseline if there is one
— then say what you did to prove which it was.

**No silent retries.** Something that works on the third attempt is an intermittent failure at a
rate of 1/3, not a pass. Record every attempt.

**One unit is one data point.** "Fails on 1 of 1 unit tested" is honest; "fails on all units" is
not, from one board. State how many units you tested, and at which revisions.

**Report coverage honestly.** A partial report that reads as complete is worse than one that looks
partial.

**Respect the limits.** Stay inside rated voltage, current, temperature, and mechanical limits.

A normal install you can undo by flashing again needs no permission — that is step 3, and it is the
whole job. **Before any operation whose recovery path you have not already proven on this unit,
state the recovery path you will use, or stop and ask.** That covers an interrupted update, a
rollback, locking a bootloader, enabling readout protection, any write to fuses or OTP memory even
when a scenario requires it as provisioning, and anything that can damage hardware or a person. The
gate is on the recovery path and not on the outcome for a reason: whether the unit survives an
interrupted update is the very thing the scenario measures, so it cannot also be the precondition
for running it. A proven path is one you have already executed on this unit — the exact tool,
interface, and image that brought it back. No harness rule will stop you here; this is on you. Name
the specific operation and why, then wait. Rewording a command to slip past a permission prompt is
never allowed.

---

## Rationalizations

| You are thinking | Reality |
|---|---|
| "The suite passed, so hardware will work." | It ran on a host that is not the product, and never saw a real signal. |
| "It works on my board." | One board, one revision, one room. That is one data point, not a result. |
| "That failure was probably a loose cable." | Probably is not a result. Prove it was the bench, or report a defect. |
| "I'll just tweak a setting to get past this." | Then you test what nobody ships. The blocker was the finding. |
| "It only failed once." | Once is a rate of 1/n. Intermittent failures are the expensive kind. |
| "No hardware is available, I'll reason it through." | Reasoning is not a bench result. The verdict is BLOCKED. |
| "Nothing broke, so it's a PASS." | Nothing broke and nothing worked is still FAIL. Show it doing its job. |

---

## Red flags

- Scenarios written after execution, or edited to match what the device did
- A finding with no bench identity or no artifact checksum
- "Reproduces sometimes", with no rate
- No baseline run when one was available, so nothing separates a new defect from an old one
- Source, project config, or build flags edited during testing
- PASS reported with no scenario that showed the change doing what it was supposed to do
- Execution started with nothing capturing, so the verbatim record exists only in your summary
- PASS reported while planned scenarios sit unexecuted
- A result from one unit stated as behavior of the product
- The host suite re-run and reported as hardware testing

---

## Verification

- [ ] All inputs recorded, artifact identified by filename, version, and checksum
- [ ] The change's intent stated in observable terms, or asked for before anything was planned
- [ ] Bench inventoried — units, revisions, serials, interface, host, tool versions, cabling, power
      source, peripherals, rig, observability channels, voltage and temperature measurability;
      on-target or HIL stated
- [ ] Baseline installed and the device confirmed healthy, or its absence recorded with the cost
      after the requester was asked for one
- [ ] Version read back off the running device matches the artifact under test, or the image
      checksum does, or `not exposed` is recorded
- [ ] Scenarios written down as `SC-nn` before the first one executed, intent scenario first
- [ ] Capture running before the first scenario, on every channel a finding could need
- [ ] Every failure carries a reproduction rate and a baseline comparison, or "no baseline"
- [ ] Every Critical and Important finding reproducible from its block alone
- [ ] Nothing modified in the artifact or the repository
- [ ] "Not tested, and why" list present, verdict stated, Definition of Done considered
