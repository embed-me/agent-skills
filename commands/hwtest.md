---
description: Test build artifacts on real hardware and report findings
agent: hardware-tester
---

Load the `hardware-test-execution` skill and follow it.

Scope: $ARGUMENTS — what changed, plus the path to the artifact under test.

If the artifact cannot be uniquely identified, or the change is not described, stop and ask. Do
not guess which build is on the bench, and do not infer the change from a filename.

Record the bench before you start, and baseline the previous known-good artifact on it — or record
that none exists. Write the scenarios down before you execute them, and give every failure a
reproduction rate.

You report findings and change nothing in the artifact under test or the repository — not the
source, the build flags, the project config, or the artifact. Bench stimulus and any device
provisioning a scenario requires are allowed, recorded in the bench inventory and named in every
finding they touch.

Report: scenarios executed, findings by severity, what was not tested and why. End with an explicit
verdict of PASS, FAIL, or BLOCKED.
