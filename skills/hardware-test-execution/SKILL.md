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

Install the artifact that will ship on the hardware that will run it, and report what the device did. A
green suite proves the logic on a machine that is not the product; only the bench sees the supply sag.

---

## Inputs

- **The artifact** — filename, version or build id, and a checksum you computed from the file you install.
- **What changed, and what it was for** — scope says where to look; intent, an on-device observable, says if it works.
- **The baseline artifact** — the previous known-good build. If none was supplied, ask the requester.
- A missing artifact, a missing description of what changed, or an intent with no observable is **stop and ask**.
- **Absent hardware is not a question** — record the bench you attempted, report `BLOCKED`, do not reason instead.
- **The intent waiver** — only the requester waives the intent scenario, never you; ask, and no answer is
  `BLOCKED`. A waived run is labelled `PASS (regressions only)`, and a waiver with no baseline is `BLOCKED`.

---

## What this skill is not

| Concern | Owner | Here |
|---|---|---|
| Automated host-side tests, the red/green loop | `test-driven-development` | Do not write or run them |
| Judgement of source code — structure, style, security | `code-review-and-quality` | Do not review it |
| Fixing what testing finds | the builder | Report it, do not patch it |
| Behavior of the real device running the shipped artifact | **this skill** | The whole job |

- **The hard rule: if a host-side automated test could have caught it, it is not a hardware scenario.**
- **Read source to locate behavior on the device** — never to judge it, never to modify it.

---

## Process

### 1. Inventory the bench

- Discover, never assume: units (model, revision, serial), probe, host OS and tool versions, cabling, power,
  peripherals, every channel you will capture through, each instrument's **condition**, and whether this bench
  can measure **supply voltage and ambient temperature** at all. None of it is recoverable later.
- Prefer a hardware-in-the-loop rig (supply, signal source, load, analyser) to the bare board; say which you used.

### 2. Derive the scenarios and write them down

- **Nothing is flashed before this list exists** — scenarios invented mid-run get rationalized into what happened.
- **The intent scenario is mandatory and runs first**, showing the stated effect through a step 1 channel under
  a falsifiable criterion written before it runs. An intended observable no channel can capture is `BLOCKED`.
- Order the rest by risk: the direct path, paths sharing a bus, pin, timer, rail or memory, and the categories below.
- Number each `SC-01`, `SC-02` in its own sequence, with preconditions, steps, and an expected result **carrying
  its source** — datasheet page, requirement id, baseline observation, or `source: change description only`.
- Reset and recovery, update and rollback, and any fuse or OTP write clear the recovery-path gate in Conduct first.

### 3. Establish a baseline

- **Run any factory or out-of-box scenario before the first install** — that precondition is then gone for good.
- Install the previous known-good artifact, confirm the device is healthy, and **capture the intent scenario's
  pre-change measurement while it is installed**; the next install destroys it.
- Anything run here is a scenario: start capture (step 5) first, and record the result.
- With none, record `no baseline available` and treat every regression claim as unproven. Never build one yourself.

### 4. Install the artifact under test

- Record tool, command, interface, options; **read the version back off the device** and compare it to the artifact.
- With no version interface, normalize the checksum to the image's own address range and length: **a mismatch
  from container format or pad bytes is not a finding**. Where neither works, record `not exposed`.

### 5. Execute exactly as written

- **Start capture before the first scenario**, every step 1 channel, timestamped. `Evidence:` cannot be recovered.
- One scenario at a time, recorded verbatim. Observe rather than infer — a step did not work because the next one did.
- **No silent bench fixes.** Note any reseat, swap, or tool update, and re-run every scenario it could affect.

### 6. Confirm and characterize every failure

- **Repeat it** to a reproduction rate — 7/10, 1/20 — and **minimize it** to the shortest triggering sequence.
  Something that works on the third attempt is an intermittent failure at 1/3, not a pass.
- **Re-run it on the baseline** to prove the defect is new; with none, record that rather than imply a regression.
  A baseline re-flash is a bench change: re-install, re-verify the read-back, re-run what it invalidated, log it.
- **Device fault or bench fault — say which, and what told them apart.** Where one unit and no baseline cannot
  settle it, record `undetermined` in `Suspected:` with what would resolve it.

### 7. Report

- Fill the Output block in `agents/hardware-tester.md`, using the finding format below for Critical and Important.
- **The intent scenario carries its measured value, its criterion, and the rate across attempts**, passed or not.

---

## Where hardware finds what the suite cannot

- **Power** — cold boot from unpowered, brown-out and sag, noisy supply, drain to cutoff.
- **Reset and recovery** — watchdog expiry, reset during a write, unplug mid-update, fast cycling.
- **Timing and jitter** — real-time deadlines, interrupt latency, drift, events arriving too close.
- **Physical I/O and analog edges** — real voltage levels, rise times, contact bounce, saturation, noise.
- **Real peripherals and signals** — the actual sensor, radio, display, or motor instead of a mock.
- **Environment** — both ends of the rated temperature range, RF interference, distance, vibration.
- **Endurance and soak** — hours of running: drift, thermal creep, wear.
- **First boot and factory state** — a unit that has never run this artifact, out-of-box behavior.
- **Update, recovery, and rollback** — update from each shipped version, interrupted update, rollback.

---

## Findings

- **Critical** — device unusable, unsafe, unrecoverable or bricked; data lost; a safety or regulatory limit violated.
- **Important** — degraded or intermittent behavior, a spec violation, manual recovery needed, or a regression.
- **Observation** — cosmetic, environmental, or pre-existing. No action implied.

```text
HW-01  [Critical | Important | Observation]  <one-line summary>
Bench:        <unit model, revision, serial; probe/interface; host OS; cabling; power; peripherals; HIL rig>
Tooling:      <flasher and host tool versions, probe firmware version, the exact install command>
Artifact:     <filename, version or build id, checksum>
Baseline:     <does the previous known-good artifact show this? yes / no / not testable>
Precondition: <state the device must be in before the first step below>
Conditions:   <supply voltage, ambient temperature, other setpoints at failure; any degraded instrument>
Steps:        1. <exact action, exact value>   2. <…>
Elapsed / sequence: <time into the run; which scenarios ran before this one on this unit>
Expected:     <what should have happened, and its source as recorded in step 2>
Actual:       <what happened, verbatim>
Rate:         <n of m attempts>
Evidence:     <log excerpt, capture, trace, measurement, photo>
Suspected:    <area or subsystem to look at>
```

- Findings run `HW-01`, `HW-02` in a sequence of their own, separate from `SC-nn`, so no reference is ambiguous.
- **Every Critical and Important finding is reproducible from its block alone**, by someone who was not there.
- Where the bench cannot produce a value, write `not instrumented` or `not exposed`. Never blank, never invented.

---

## Verdict

- `PASS` | `FAIL` | `BLOCKED` — a headline, not a computation; you observe and record, the reader decides.
- **FAIL** — the change did not do on the device what it was for, **or** a Critical or Important finding is open.
  Intent is the first trigger: a change that delivers nothing has failed even when it breaks nothing.
- **BLOCKED** — the bench could not answer: rig broken, artifact will not install, unit died mid-run, hardware
  absent, intent never stated, or intent waived with no baseline.
- **PASS** — the intent scenario ran and showed the change doing what it was for, every scenario you ran passed,
  and nothing Critical or Important is open. Waived intent makes it `PASS (regressions only)`.
- **A scenario that was not executed is not a passing scenario.** Every report ends with a **not tested, and why**
  list: scenarios planned but not run, units you lacked, anything step 1 says this bench cannot do.
- **When the picture is mixed, report the mixture**, never a tie-break: the most serious token you observed, a
  split across units per unit in the scenario table, a measurement near its criterion with value, criterion, and
  rate, a degraded instrument named on the findings that leaned on it.

---

## Conduct

- **Change nothing**: not the source, build flags, project config, or artifact. Untestable as delivered is a finding.
- **Bench stimulus, and the provisioning a scenario requires — identity, keys, calibration, credentials,
  partition init — are not changes**, provided you record each in the bench inventory and name it on every finding.
- **Disabling a protective mechanism the shipped configuration enables is not provisioning** — a watchdog,
  brown-out detector or readout protection off makes the unit a different thing: no `PASS`, needing it is a finding.
- **One unit is one data point.** State how many units you tested, and at which revisions.
- **Respect the limits.** Stay inside rated voltage, current, temperature, and mechanical limits.
- **Before any operation whose recovery path you have not proven on this unit, state the path you will use, or
  stop and ask** — interrupted update, rollback, bootloader lock, readout protection, any fuse or OTP write even
  as provisioning, anything that can damage hardware or a person. The gate is the recovery path, not the outcome:
  whether the unit survives is what the scenario measures. **No harness rule gates flashing, fuses, or OTP.**

---

## Rationalizations

| You are thinking | Reality |
|---|---|
| "The suite passed, so hardware will work." | It ran on a host that is not the product, and never saw a real signal. |
| "It works on my board." | That is one data point, not a result about the hardware. |
| "That failure was probably a loose cable." | Probably is not a result. Prove it, or record it undetermined. |
| "No hardware is available, I'll reason it through." | Reasoning is not a bench result. The verdict is BLOCKED. |

---

## Red flags

- Scenarios written after execution, or edited to match what the device did
- A unit flashed before the scenario list existed, taking the pre-change value with it
- A finding with no bench identity, no artifact checksum, or no rate
- No baseline run when one was available
- Source, project config, or build flags edited during testing
- PASS with no scenario showing the change doing what it was for, or with planned scenarios unrun
- A split result across units flattened into one word
- The host suite re-run and reported as hardware testing

---

## Verification

- [ ] Artifact identified; intent stated observably, or waived by the requester on record
- [ ] Bench inventoried with instrument conditions, voltage and temperature measurability, on-target or HIL
- [ ] Scenarios written as `SC-nn` before any flash — intent first, falsifiable criterion, expected results sourced
- [ ] Recovery path stated and proven on this unit, or asked, before any irreversible operation
- [ ] Factory-state scenarios run before the first install; the pre-change value captured on the baseline
- [ ] Version or normalized checksum read back off the running device, or `not exposed`; capture running first
- [ ] Every failure carries a rate and a baseline comparison, or "no baseline"
- [ ] Every Critical and Important finding reproducible from its block alone, per unit where units differed
- [ ] Nothing modified; no protective mechanism disabled on a unit reported `PASS`
- [ ] "Not tested, and why" present, verdict stated, Definition of Done considered
