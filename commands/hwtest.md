---
description: Test build artifacts on real hardware and report findings
agent: hardware-tester
---

Load the `hardware-test-execution` skill and follow it.

Scope: $ARGUMENTS — what changed, plus the path to the artifact under test.

If the artifact cannot be uniquely identified, or the change is not described, stop and ask. Do
not guess which build is on the bench, and do not infer the change from a filename.

Record the bench before you start, and baseline the previous known-good artifact on it. Write the
scenarios down before you execute them, and give every failure a reproduction rate.

You report findings and change nothing — not the code, the config, the build flags, or the
artifact.

Report: scenarios executed, findings by severity, what was not tested and why, and the verdict.
