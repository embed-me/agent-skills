---
description: Test build artifacts on real hardware and report findings
agent: hardware-tester
---

Load the `hardware-test-execution` skill and follow it.

Scope: $ARGUMENTS — what changed, what it should now do differently on the device, the artifact, the baseline.

- **Stop and ask** if the artifact is not uniquely identifiable, the change is not described, or no on-device
  observable is stated. Ask for a missing baseline first. Only the requester waives intent; no answer is `BLOCKED`.
- Record the bench, its capture channels, whether voltage and temperature are measurable, and each instrument's
  condition. Write scenarios as `SC-nn` before anything is flashed, intent first under a falsifiable criterion,
  capture the pre-change value on the baseline, start capture before executing, number findings `HW-nn` with a rate.
- **Change nothing** in the artifact or repository; bench stimulus and required provisioning are the exception
  on the skill's terms. A watchdog, brown-out detector or readout protection off is not provisioning, never a `PASS`.
- **Stop and ask before anything irreversible** — fuses, OTP, bootloader lock, or an unproven recovery path.
  No harness rule gates these.

Report: intent demonstrated or not, scenarios per unit, findings by severity, what was not tested and why,
the mixture where mixed. `PASS` only on demonstrated intent, `PASS (regressions only)` on a requester waiver
with a baseline; otherwise `FAIL` or `BLOCKED`.
