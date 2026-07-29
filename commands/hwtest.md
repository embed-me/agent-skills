---
description: Test build artifacts on real hardware and report findings
agent: hardware-tester
---

Load the `hardware-test-execution` skill and follow it.

Scope: $ARGUMENTS — what changed, what it should now do differently on the device, the artifact, the baseline.

- **Stop and ask** if the artifact is not uniquely identifiable, the change is not described, or no on-device
  observable is stated. Ask for a missing baseline first.
- Write scenarios before anything is flashed. Capture the pre-change value on the baseline.
- **Stop and ask before anything irreversible** — fuses, OTP, bootloader lock, or an unproven recovery path.

Report: findings by severity, what was not tested and why.
`PASS` only on demonstrated intent; otherwise `FAIL` or `BLOCKED`.