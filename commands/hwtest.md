---
description: Test build artifacts on real hardware and report findings
agent: hardware-tester
---

Load the `hardware-test-execution` skill and follow it.

Scope: $ARGUMENTS — what changed and what it was supposed to achieve on the device, the path to the
artifact under test, and the path to the previous known-good artifact to baseline against.

If the artifact cannot be uniquely identified, if the change is not described, or if the change is
described without saying what should now be observably different on the device, stop and ask. Do
not guess which build is on the bench, do not infer the change from a filename, and do not infer
its intent. If no baseline was given, ask for it before recording that none exists. If the change has
no on-device observable at all, ask: only the requester can waive the intent scenario, and no answer
means `BLOCKED`.

Record the bench before you start — the channels you will capture through, whether it can measure
supply voltage and ambient temperature, and the condition of each instrument. Write the scenarios down
as `SC-nn` before anything is flashed, give the intent scenario a falsifiable criterion and run it
first, capture the pre-change value while the baseline is installed, start capture before executing,
and number every finding `HW-nn` with a reproduction rate.

You report findings and change nothing in the artifact under test or the repository. Bench stimulus
and the device provisioning a scenario requires are the exception; the skill states the terms. Turning
off a watchdog, brown-out detector, or readout protection is not provisioning, and no result obtained
that way is a `PASS`.

Stop and ask before anything irreversible: fuses, OTP, bootloader lock, readout protection, or any
scenario whose recovery path is unproven. No harness rule gates these.

Report: intent demonstrated or not, scenarios executed per unit, findings by severity, what was not
tested and why. Where the picture is mixed, report the mixture rather than compressing it into one
word. End with an explicit verdict of PASS, FAIL, or BLOCKED — PASS only if the intent scenario showed
the change doing what it was for, or `PASS (regressions only)` where the requester waived the intent
and a baseline was available.
