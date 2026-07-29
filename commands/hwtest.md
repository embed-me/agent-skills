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
its intent. If no baseline was given, ask for it before recording that none exists.

Record the bench before you start — including the channels you will capture through and whether it
can measure supply voltage and ambient temperature. Write the scenarios down as `SC-nn` before you
execute them, run the one that demonstrates the change's intent first, start capture before it, and
number every finding `HW-nn` with a reproduction rate.

You report findings and change nothing in the artifact under test or the repository. Bench stimulus
and the device provisioning a scenario requires are the exception; the skill states the terms.

Stop and ask before anything irreversible: fuses, OTP, bootloader lock, readout protection, or any
scenario whose recovery path is unproven. No harness rule gates these.

Report: intent demonstrated or not, scenarios executed, findings by severity, what was not tested
and why. End with an explicit verdict of PASS, FAIL, or BLOCKED — PASS only if the intent scenario
showed the change doing what it was for.
