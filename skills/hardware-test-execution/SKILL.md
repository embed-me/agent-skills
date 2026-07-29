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
Identified means filename, version or build id, and a checksum you computed yourself from the file you
are about to install.

**What changed, and what it was supposed to achieve.** The scope: which subsystems, which behavior,
which peripherals or interfaces — without it you test at random, the slowest coverage a bench hour
buys. And the intent, stated in terms observable on the device: what should now be measurably
different, through what channel. Scope says where to look; intent says whether it worked.

**The bench.** Whatever hardware is physically connected to the host. Inventory it in step 1.

**The baseline artifact, when one exists.** The previous known-good build, for step 3's comparison. A
first release has none, and neither does a project that kept nothing. If one was not supplied, ask the
requester for it before proceeding; only after that answer may you record "no baseline available".

A missing or unidentifiable **artifact**, a missing description of **what changed**, or a change
described without saying what should now be observably different, is **stop and ask** — none of the
three can be planned around, and a finding against an unidentified binary is one nobody can act on.

**The intent waiver.** Some changes have no on-device observable at all — a null-dereference fix, a
parser bug, an error path nothing on this bench can reach. Ask the requester, the same way you ask for
the baseline: only their answer waives the intent scenario, and no answer or an unclear one is
`BLOCKED`. It is never your call. A waived run reports `PASS (regressions only)`, never a bare `PASS`,
and states that the intent was not hardware-observable, who waived it, and that the run verified no
hardware regressions and did not verify the change's purpose. A waiver also makes the baseline
mandatory — regressions are then the only dimension left, so waived intent with no baseline is
`BLOCKED`.

**Absent hardware is not a question.** Record the bench you attempted to use — what you looked for, what
was connected, what was missing — and report `BLOCKED`. Do not ask whether to proceed without hardware,
and do not reason about the device instead.

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

Demonstrating the change's intent (step 2) does not bend that rule. It means running the shipped
artifact on the device and observing the intended effect *there* — a boot that now completes in
2s, a rail that now settles, a sensor now read at the new rate. It is never a reason to run the
host suite, and never a licence to report a suite result as a bench result. An intent with no
on-device observable at all is the waiver question at Inputs: ask, and take the requester's answer.

You may read source to locate the behavior on the device. You may never judge its style — that belongs
to the reviewer — and you may never modify it.

---

## Process

### 1. Inventory the bench

Never assume a probe, flasher, board, or host OS; discover them. Record every unit under test (model,
hardware revision, serial), the debug probe or interface (model, serial, firmware version), host OS
and tool versions, cabling, power source, attached peripherals, and every observability channel you
will capture through: serial console, logger, analyser, instrument readout. Record explicitly whether
this bench can measure supply voltage and ambient temperature at all — the finding block demands both
and neither can be recovered afterwards.

Record each instrument's **condition** alongside its presence: calibration status, a probe missing from
the rig, a channel known bad. A supply out of calibration is a capability present and degraded, not one
absent; name it in `Conditions:` or `Tooling:` on every finding that leaned on it. What this bench can
and cannot do is recorded here, and "not tested, and why" draws on that record.

- **On-target** — the device connected directly to the host. Available whenever hardware is.
- **Hardware-in-the-loop rig** — instrumentation that drives inputs and observes outputs the bare
  board cannot: programmable supply, signal source, load, analyser, actuated controls.

Prefer the rig when present; it reaches physical dimensions the bare target cannot. State which you used.

### 2. Derive the scenarios and write them down before executing

Nothing is flashed before this list exists: scenarios invented during or after execution get
rationalized into whatever the device happened to do.

**One scenario is mandatory: the intent scenario.** It exercises what the change was supposed to
achieve and demonstrates it on the device — the stated effect, through a channel you inventoried in
step 1. It is not optional and it runs first. Its expected result must carry a falsifiable acceptance
criterion, written before it runs: a threshold, or a comparison against the pre-change value step 3
captures. "Boots faster" is not a criterion; "cold boot to the ready banner in under 2.0s" is. If the
requester waived the intent at Inputs there is no intent scenario, and the baseline becomes
mandatory. If the requester stated an observable no channel on this bench can capture, report
`BLOCKED`.

Every other scenario is risk-based, ordered by risk, highest first: what changed, what physical
behavior it could affect, how that looks when it fails. Cover the direct path, adjacent paths sharing a
hardware resource — the same bus, pin, timer, rail, or memory region — and the physical dimensions no
host test reaches (next section). Each scenario gets an ID from its own sequence — `SC-01`, `SC-02`,
numbered separately from findings so a cross-reference is never ambiguous — plus preconditions,
numbered steps, and an expected result with **its source recorded beside it**: datasheet and page, spec
or requirement id, or baseline observation. Where no external spec exists, write `source: change
description only` — hours later, that source is what you cannot reconstruct.

A scenario under **Reset and recovery**, **Update, recovery, and rollback**, or any write to fuses or
OTP must clear the recovery-path gate in Conduct before it enters this list.

### 3. Establish a baseline

Run any scenario requiring factory or out-of-box state before the first install: once you flash, that
precondition is gone, and on a single unit it is gone permanently. Anything you run here is a scenario,
so start capture (step 5) before it and record its result like any other.

Install the previous known-good artifact and confirm the device is healthy before touching the new one.
With the baseline installed, capture the intent scenario's measurement on the same channel, as the
pre-change value — for a comparative intent (boot time, settling, sample rate) that value exists only
while the baseline is on the unit, and the next install destroys it.

When no previous artifact exists, proceed without one — but record "no baseline available" and treat
every regression claim as unproven, because nothing then separates a new defect from a pre-existing
one from a broken bench, and that separation is most of your value. A waived intent removes that
option (see Inputs). Do not build a baseline yourself: building is not this role, and an artifact you
built is not one anybody shipped.

### 4. Install the artifact under test

Record exactly how: tool, command, interface, options. Then read the version back off the running device
and compare it against the artifact identity — a stale image looks exactly like a passing run. If the
artifact exposes no version on any interface, fall back to checksums, but normalize the comparison first
to the image's own address range and length in the same representation: dump raw and truncate to the
image length, or checksum the artifact's binary payload rather than its container. A raw read-back is
padded with `0xFF` and a `.hex` or `.elf` will never match a raw dump; **a mismatch attributable to
container format or pad bytes is not a finding.** Where neither a version nor a normalizable comparison
is available, record `not exposed` and state that a stale image could not be ruled out.

### 5. Execute exactly as written

Start capture before the first scenario: open the serial log, arm the analyser, begin the recording on
every channel from step 1, timestamped so `Elapsed / sequence:` can be filled from it later.
`Evidence:` cannot be reconstructed after the fact — start the first step with nothing capturing and
the verbatim record is already lost.

One at a time. Observe rather than infer — do not conclude a step worked because the next one did.
Record the actual result verbatim. Do not silently fix the bench mid-run: if you reseat a connector,
swap a supply, or update a tool, note it and re-run every scenario it could have affected.

### 6. Confirm and characterize every failure

- **Repeat it** enough times to state a reproduction rate: 7/10, 10/10, 1/20.
- **Re-run it on the baseline artifact** to prove the defect is new. If it is not, say so; with no
  baseline, record that instead of implying a regression. **A baseline re-flash is a bench change.**
  After it, re-install the artifact under test, re-verify the version read-back per step 4, re-run every
  scenario the round-trip could have invalidated, and record it in `Elapsed / sequence:`.
- **Minimize it** to the shortest sequence that still triggers it.

Then establish whether the fault is the device or the bench, and say what you did to tell them apart.
With one unit and no baseline you often cannot: record the attribution as undetermined in `Suspected:`,
with what would resolve it. Undetermined and recorded is honest; asserted without proof is not.

### 7. Report

The Output block in `agents/hardware-tester.md` is the authoritative report shape — scenarios executed
per unit, findings by severity, "not tested, and why", evidence, and an explicit verdict. Fill it, using
the finding format below for every Critical and Important.

The intent scenario carries its **measured value, the criterion it was judged against, and the rate
across attempts**, whether or not it passed. "2.3s measured against a 2.0s target, 6 of 10 runs" is a
complete report of that scenario; deciding what to do about it is the reader's job.

`Evidence:` inside a finding is that finding's reproduction record — the excerpt or measurement showing
what happened. The report's Evidence section is where the artifacts themselves (full logs, captures,
transcripts, photographs) are attached and located.

---

## Where hardware finds what the suite cannot

These categories justify the bench. Pick the ones the change plausibly touches; walking all of them
every build burns bench time without answering anything.

- **Power** — cold boot from unpowered, brown-out and sag, noisy supply, drain to cutoff.
- **Reset and recovery** — watchdog expiry, reset during a write, unplug mid-update, fast cycling.
- **Timing and jitter** — real-time deadlines, interrupt latency, drift, events arriving too close.
- **Physical I/O and analog edges** — real voltage levels, rise times, contact bounce, saturation, noise.
- **Real peripherals and signals** — the actual sensor, radio, display, or motor instead of a mock.
- **Environment** — both ends of the rated temperature range, RF interference, distance, vibration.
- **Endurance and soak** — hours of running: drift, thermal creep, wear.
- **First boot and factory state** — a unit that has never run this artifact, out-of-box behavior.
- **Update, recovery, and rollback** — update from each shipped version, interrupted update, rollback, recoverability.

---

## Findings

Findings are numbered in their own sequence: `HW-01`, `HW-02`, `HW-03`. Scenarios use `SC-nn` (step 2);
the counters never share a number, so `blocked by HW-02` and `blocked by SC-02` are both unambiguous.

- **Critical** — the device is unusable, unsafe, unrecoverable or bricked; data is lost; or a safety
  or regulatory limit is violated.
- **Important** — degraded or intermittent behavior, a spec violation, a failure needing manual
  recovery, or a regression against the baseline.
- **Observation** — cosmetic, environmental, or pre-existing. No action implied.

```text
HW-01  [Critical | Important | Observation]  <one-line summary>
Bench:        <unit model, revision, serial; probe/interface; host OS; cabling; power; peripherals; HIL rig>
Tooling:      <flasher and host tool versions, probe firmware version, the exact install command>
Artifact:     <filename, version or build id, checksum>
Baseline:     <does the previous known-good artifact show this? yes / no / not testable>
Precondition: <state the device must be in before the first step below>
Conditions:   <supply voltage, ambient temperature, other setpoints at the time of failure, and any
              instrument that was degraded>
Steps:        1. <exact action, exact value>
              2. <…>
Elapsed / sequence: <time into the run; which scenarios ran before this one on this unit>
Expected:     <what should have happened, and its source as recorded in step 2>
Actual:       <what happened, verbatim>
Rate:         <n of m attempts>
Evidence:     <log excerpt, capture, trace, measurement, photo>
Suspected:    <area or subsystem to look at>
```

**Suspected area points; it does not prescribe.** Naming the fix is the builder's job — your guess becomes the plan.

**`Bench:` carries hardware identity; `Tooling:` carries software versions.** The probe appears in
both: its model and serial in `Bench:`, its firmware version in `Tooling:`.

Every Critical and Important finding must be reproducible from its block alone, by someone who was not
there and cannot ask you a question: `Bench:` and `Tooling:` carry the step 1 inventory, `Conditions:`
the setpoints true when it failed, `Elapsed / sequence:` its place in a soak or a run of earlier
scenarios. Where the bench genuinely cannot produce a value, write `not instrumented` (no meter on the
rail, no thermometer in the room) or `not exposed` (a revision only on silkscreen, an image with no
version interface) — both permitted, both usable. A blank field reads as an oversight and an invented
one poisons every attempt to reproduce the finding.

---

## Verdict

`PASS` | `FAIL` | `BLOCKED` — a headline, not a computation. You observe, record, and flag; the reader
decides what to do about it.

- **FAIL** — the change did not do on the hardware what it was supposed to do, **or** any Critical or
  Important finding is open. Intent is the first trigger and it stands alone: a change that delivers
  nothing has failed even when it breaks nothing.
- **BLOCKED** — the bench could not answer the question: rig broken, artifact will not install, unit died
  mid-run, hardware absent, intent never stated, or intent waived with no baseline. Never a `PASS`.
- **PASS** — the intent scenario ran and showed the change doing what it was for, every scenario you ran
  passed, and no Critical or Important finding is open. Intent waived at Inputs makes the label
  `PASS (regressions only)`.

When more than one token applies, report the most serious thing you observed; the findings carry the rest.

**A scenario that was not executed is not a passing scenario** — reporting untested behavior as passed
removes the last check before the product reaches a person, and removes it invisibly. So every report
ends with a **not tested, and why** list: scenarios planned but not run, units you did not have,
anything the step 1 inventory says this bench cannot do, each entry with its reason. You need not
classify that reason before picking a token. Coverage you state is information; coverage you imply is
a lie.

**When the picture is mixed, report the mixture.** Do not compress it into a token and do not invent a
rule to break the tie: a split across units goes in the scenario table per unit, a measurement near its
criterion goes in with value, criterion, and rate, a degraded instrument is named on the findings that
leaned on it. Detail is how ambiguity is resolved here — never another rule.

---

## Conduct

**Change nothing in the artifact under test or the repository** — not the source, not the build flags,
not the project config, not the artifact itself. The moment you modify the thing under test you are
testing something nobody will ship. If it cannot be tested as delivered, that is a finding.

Bench stimulus is not a change. Setting a supply, injecting a signal, driving a line, moving the
temperature, and provisioning the device — identity, keys, calibration, network credentials, partition
or filesystem init — are the job; that is what the bench is for. Two conditions: record it in the bench
inventory, and name it in every finding it touches. Provisioning that writes fuses or OTP is still
gated below.

**Disabling a protective mechanism the shipped configuration enables is not provisioning.** A watchdog,
brown-out detector, or readout protection switched off makes the unit a different thing from the one
that ships: a result obtained that way cannot be reported as `PASS` however loudly you record it, and
needing it to run a scenario is itself a finding.

**Report, do not fix.** A fix by the tester erases the finding, exactly as a reviewer who edits stops
being a gate.

**Separate bench fault from device fault.** A finding that is really a loose connector or a stale tool
costs more than no finding. Swap the cable, swap the unit, re-run the baseline if there is one — then
say what told them apart, and say so when you still cannot.

**No silent retries.** Something that works on the third attempt is an intermittent failure at 1/3, not
a pass. Record every attempt.

**One unit is one data point.** "Fails on 1 of 1 unit tested" is honest; "fails on all units" is not,
from one board. State how many units you tested and at which revisions, and give each unit its own row
in the scenario table: a result that holds on unit A and not unit B is reported per unit, as observed.

**Respect the limits.** Stay inside rated voltage, current, temperature, and mechanical limits.

A normal install you can undo by flashing again needs no permission — that is step 4, the whole job.
**Before any operation whose recovery path you have not already proven on this unit, state the recovery
path you will use, or stop and ask.** That covers an interrupted update, a rollback, locking a
bootloader, enabling readout protection, any write to fuses or OTP even when a scenario requires it as
provisioning, and anything that can damage hardware or a person. The gate is on the recovery path, not
the outcome: whether the unit survives an interrupted update is the very thing the scenario measures, so
it cannot also be its precondition. A proven path is one you have already executed on this unit — the
exact tool, interface, and image that brought it back. No harness rule will stop you here; this is on
you. Name the operation and why, then wait; never reword a command to slip past a permission prompt.

---

## Rationalizations

| You are thinking | Reality |
|---|---|
| "The suite passed, so hardware will work." | It ran on a host that is not the product, and never saw a real signal. |
| "That failure was probably a loose cable." | Probably is not a result. Prove it, or record it undetermined. |
| "I'll just tweak a setting to get past this." | Then you test what nobody ships. The blocker was the finding. |
| "No hardware is available, I'll reason it through." | Reasoning is not a bench result. The verdict is BLOCKED. |
| "Nothing broke, so it's a PASS." | Nothing broke and nothing worked is still FAIL. Show it doing its job. |

---

## Red flags

- Scenarios written after execution, or edited to match what the device did
- A unit flashed before the scenario list existed, taking the pre-change value with it
- A finding with no bench identity, no artifact checksum, or no rate
- No baseline run when one was available, so nothing separates a new defect from an old one
- Source, project config, or build flags edited during testing
- PASS reported with no scenario that showed the change doing what it was for, or with planned
  scenarios unexecuted
- A split result across units flattened into one word
- The host suite re-run and reported as hardware testing

---

## Verification

- [ ] Artifact identified — filename, version, checksum. Intent stated observably or asked for, and any
      waiver recorded with who granted it
- [ ] Bench inventoried, each instrument's condition included, voltage and temperature measurability
      and on-target or HIL stated
- [ ] Scenarios written as `SC-nn` before anything was flashed — intent first with a falsifiable
      criterion, every expected result carrying its source
- [ ] Recovery path stated and proven on this unit, or asked, before any irreversible operation
- [ ] Factory-state scenarios run before the first install; the intent measurement's pre-change value
      captured while the baseline was installed; a missing baseline recorded with its cost after asking
- [ ] Version or normalized checksum read back off the running device, or `not exposed`
- [ ] Capture running before the first scenario, on every channel a finding could need
- [ ] Every failure carries a rate and a baseline comparison, or "no baseline"
- [ ] Every Critical and Important finding reproducible from its block alone, per unit where units differed
- [ ] Nothing modified; no protective mechanism disabled on a unit whose result is reported `PASS`
- [ ] "Not tested, and why" present, verdict stated, Definition of Done considered
