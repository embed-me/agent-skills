---
name: hardware-test-execution
description: >-
  Executes post-build artifacts on real, physically connected hardware and reports reproducible
  findings with a verdict, changing nothing. Use when a build has produced an artifact and a
  device or rig is available to run it on. Hardware and toolchain agnostic.
license: MIT
compatibility: opencode, claude-code, cursor, codex, any agent that can read a file
metadata:
  phase: validate
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

**What changed.** The scope of the change: which subsystems, which behavior, which peripherals or
interfaces. Without it you test at random, and random is the slowest coverage a bench hour buys.

**The bench.** Whatever hardware is physically connected to the host. Inventory it in step 1.

If any of the three is missing, or the artifact cannot be uniquely identified, **stop and ask.**
Testing an unidentified binary produces findings nobody can act on: later, no one can tell whether
the defect was in that build, an older one, or a file that no longer exists.

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

You may read source to understand what changed and to locate the behavior on the device. You may
never judge its style — that belongs to the reviewer — and you may never modify it.

---

## Process

### 1. Inventory the bench

Never assume a probe, flasher, board, or host OS; discover them. Record the identity of every unit
under test (model, hardware revision, serial), the debug probe or interface, host OS and tool
versions, cabling, power source, and attached peripherals.

- **On-target** — the device connected directly to the host. Available whenever hardware is.
- **Hardware-in-the-loop rig** — instrumentation that drives inputs and observes outputs the bare
  board cannot: programmable supply, signal source, load, analyser, actuated controls.

Prefer the rig when present; it reaches physical dimensions the bare target cannot. Fall back to
on-target and state which you used. An unrecorded bench makes every finding irreproducible.

### 2. Establish a baseline

Install the previous known-good artifact and confirm the device is healthy before touching the new
one. Without a baseline you cannot separate a new defect from a pre-existing one from a broken
bench, and that separation is most of your value.

### 3. Install the artifact under test

Record exactly how: tool, command, interface, options. Then read the version back off the running
device and compare it against the artifact identity. Testing a stale image is the classic wasted
session, and it looks exactly like a passing run.

### 4. Derive scenarios from the change

Risk-based: what changed, what physical behavior it could affect, how that looks when it fails.
Cover the direct path, adjacent paths sharing a hardware resource — the same bus, pin, timer, rail,
or memory region — and the physical dimensions no host test reaches (next section).

### 5. Write the scenarios down before executing

Each gets an ID, preconditions, numbered steps, expected result. Order by risk, highest first, and
write all of them before the first one runs. Scenarios invented during or after execution get
rationalized into whatever the device happened to do.

### 6. Execute exactly as written

One at a time. Observe rather than infer — do not conclude a step worked because the next one did.
Record the actual result verbatim. Do not silently fix the bench mid-run: if you reseat a connector,
swap a supply, or update a tool, note it and re-run every scenario it could have affected.

### 7. Confirm and characterize every failure

- **Repeat it** enough times to state a reproduction rate: 7/10, 10/10, 1/20.
- **Re-run it on the baseline artifact** to prove the defect is new. If it is not, say so.
- **Minimize it** to the shortest sequence that still triggers it.

Then establish whether the fault is the device or the bench, and prove which.

### 8. Report

Findings in the format below, an explicit verdict, and the list of what you did not test.

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
- **Endurance and soak** — hours of running: leaks, handle exhaustion, drift, thermal creep, wear.
- **First boot and factory state** — a unit that has never run this artifact, out-of-box behavior.
- **Update, recovery, and rollback** — update from each shipped version, interrupted update,
  rollback after a failure, and whether a unit can be recovered at all.

---

## Findings

Findings are numbered in sequence: `HW-01`, `HW-02`, `HW-03`.

- **Critical** — the device is unusable, unsafe, unrecoverable or bricked; data is lost; or a safety
  or regulatory limit is violated.
- **Important** — degraded or intermittent behavior, a spec violation, a failure needing manual
  recovery, or a regression against the baseline.
- **Observation** — cosmetic, environmental, or pre-existing. No action implied.

```text
HW-01  [Critical | Important | Observation]  <one-line summary>
Bench:        <unit model, revision, serial; probe/interface; host OS; HIL rig if used>
Artifact:     <filename, version or build id, checksum>
Baseline:     <does the previous known-good artifact show this? yes / no / not testable>
Precondition: <state the device must be in before step 1>
Steps:        1. <exact action, exact value>
              2. <…>
Expected:     <what should have happened, and where that expectation comes from>
Actual:       <what happened, verbatim>
Rate:         <n of m attempts>
Evidence:     <log excerpt, capture, trace, measurement, photo>
Suspected:    <area or subsystem to look at>
```

**Suspected area points; it does not prescribe.** Naming the fix is the builder's job, and a
tester's guess at it tends to become the plan.

Every Critical and Important finding must be reproducible from its block alone, by someone who was
not there and cannot ask you a question.

---

## Verdict

`PASS` | `FAIL` | `BLOCKED`

- **PASS** — every planned scenario actually ran, and none produced a Critical or Important finding.
- **FAIL** — any Critical or Important finding is open.
- **BLOCKED** — the bench cannot answer the question: hardware absent, rig broken, artifact will
  not install. BLOCKED is the honest answer, and it is never reported as PASS.

**A scenario that was not executed is not a passing scenario.** Reporting untested behavior as
passed is the single most damaging thing this role can do: it removes the last check before the
product reaches a person, and it removes it invisibly.

Every report ends with a **not tested, and why** list — scenarios planned but not run, categories
the bench could not reach, units you did not have. Coverage you state is information; coverage you
imply is a lie.

---

## Conduct

**Test the artifact, not the source.** Source locates behavior on the device; it is not the subject.

**Change nothing.** Not the code, not the config, not the artifact, not the build flags. The moment
you modify the thing under test you are testing something nobody will ship. If it cannot be tested
as delivered, that is a finding.

**Report, do not fix.** A fix by the tester erases the finding, for the same reason a reviewer who
edits stops being a gate.

**Separate bench fault from device fault.** A finding that is really a loose connector or a stale
tool costs more than no finding. Swap the cable, swap the unit, re-run the baseline — then say what
you did to prove which it was.

**No silent retries.** Something that works on the third attempt is an intermittent failure at a
rate of 1/3, not a pass. Record every attempt.

**One unit is one data point.** "Fails on 1 of 1 unit tested" is honest; "fails on all units" is
not, from one board. State how many units you tested, and at which revisions.

**Report coverage honestly.** A partial report that reads as complete is worse than one that looks
partial.

**Respect the limits.** Stay inside rated voltage, current, temperature, and mechanical limits.
**Stop and ask before anything irreversible** — writing fuses or OTP memory, locking a bootloader,
enabling readout protection, any update that can brick a unit, anything that can damage hardware or
a person. This repo gates such commands deliberately, and rewording one to slip past the prompt is
not allowed. Say what you wanted to do and why, then wait.

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

---

## Red flags

- Scenarios written after execution, or edited to match what the device did
- A finding with no bench identity or no artifact checksum
- "Reproduces sometimes", with no rate
- No baseline run, so nothing separates a new defect from an old one
- Source, config, or build flags edited during testing
- PASS reported while planned scenarios sit unexecuted
- A result from one unit stated as behavior of the product
- The host suite re-run and reported as hardware testing

---

## Verification

- [ ] All three inputs recorded, artifact identified by filename, version, and checksum
- [ ] Bench inventoried — units, revisions, serials, interface, host, rig; on-target or HIL stated
- [ ] Baseline artifact installed and the device confirmed healthy first
- [ ] Version read back off the running device matches the artifact under test
- [ ] Scenarios written down before the first one executed
- [ ] Every failure carries a reproduction rate and a baseline comparison
- [ ] Every Critical and Important finding reproducible from its block alone
- [ ] Nothing modified — not code, config, artifact, or build flags
- [ ] "Not tested, and why" list present, verdict stated, Definition of Done considered
