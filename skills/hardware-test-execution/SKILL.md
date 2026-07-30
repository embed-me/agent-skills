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

- **The artifact** — filename, version or build id, and a checksum you computed from the files to test
- **What changed, and what it was for** — scope defines where to look
- **Absent hardware or Artefact** — report `BLOCKED`, do not reason.

---

## Rules

- **If a host-side automated test could have caught it, it is not a hardware scenario** — report `BLOCKED` with 
specific reasoning
- **Read source to locate behavior on the device** — never to judge it, never to modify it!

---

## Process

### 1. Inventory the bench

- Discover, never assume: units (model, revision, serial), attached equipment through MCPs and potentially other paths.

### 2. Derive the scenarios and write them down

- **Nothing is flashed before this list exists** — scenarios invented mid-run get rationalized into what happened.
- **Scenario Requirements** - has to contain expectation, specific pass criteria, observation/measurement, clarification text

### 4. Install the artifact under test

- **Install** - Record tool, command, interface, options
- **Verify** - read the version back off the device and compare it to the artifact.

### 5. Execute exactly as written

- **Establish execution order** - schedule them for efficience, ensure that issues can still be pinpointed
- **Execute defined scenarios** - timestamped

### 6. Confirm and characterize every failure

- **Repeat tests** If fast, repeat a test - document if it is flaky.
- **Prove the defect is new** - record that rather than imply a regression.
- **Device fault or bench fault.** - try to fix it, if not possible stop and raise issue 
- **If the results are unclear, settle it, record evidence.** - do not assume

## Findings

- **Critical** — device unusable, unsafe, unrecoverable or bricked; data lost; a safety or regulatory limit violated.
- **Important** — degraded or intermittent behavior, a spec violation, manual recovery needed, or a regression.
- **Observation** — cosmetic, environmental, or pre-existing. No action implied.

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
- A unit flashed/run before the scenario list existed, taking the pre-change value with it
- No baseline run when one was available
- Source, project config, or build flags edited during testing
- PASS with no scenario showing the change doing what it was for, or with planned scenarios unrun

---

## Verification

- [ ] Artifact identified and verified
- [ ] Bench inventoried with instrument conditions
- [ ] Scenarios written before any flash — intent first, expected results defined
- [ ] Version or normalized checksum read back off the running device, or `not exposed`
- [ ] Every failure carries a rate and a baseline comparison, or "no baseline"
- [ ] Every Critical and Important finding reproducible from its block alone, per unit where units differed
