# INC-003 — High CPU Utilization During Microsoft Edge Usage

## Environment

- Organization: Northstar Solutions
- Workstation: `NS-W11-01`
- Operating System: Windows 11 Pro
- Platform: VMware Workstation
- Assigned Department: Finance
- VM Memory: 8 GB
- Virtual Processors: 2

## User Report

A simulated Finance user reported that the workstation became sluggish while using Microsoft Edge.

## Initial Assessment

Task Manager was used to inspect overall system resource utilization.

Observed resource utilization included:

- CPU utilization reaching approximately 96% during an initial observation
- Memory utilization around 68–71%
- Disk activity near 0%
- Network activity near 0%

CPU utilization remained around 50% during subsequent observations, indicating that CPU usage required further investigation.

## Investigation

The Processes tab in Task Manager was sorted by CPU utilization.

Microsoft Edge repeatedly appeared as the largest CPU consumer, using approximately 38–44% CPU during multiple observations.

The Microsoft Edge process group was expanded to inspect individual browser processes.

An individual Edge tab/content process was observed consuming approximately 40% CPU.

The Details tab was also reviewed to examine individual `msedge.exe` processes and their process identifiers (PIDs).

## Initial Remediation

Two browser tabs associated with high resource utilization were closed normally rather than forcibly terminating their processes.

## Result

CPU utilization remained elevated at approximately 47%.

Another Edge tab became the primary CPU consumer at approximately 38%.

This indicated that closing the initially identified tabs did not fully resolve the issue.

## Further Remediation

After confirming that no required work needed to be preserved, Microsoft Edge was closed normally.

A graceful application shutdown was used instead of immediately terminating the application through Task Manager.

## Verification

After closing Microsoft Edge and allowing the workstation to stabilize:

- CPU utilization decreased to approximately 11%
- Memory utilization decreased to approximately 56%
- Disk utilization remained near 0%
- Network utilization remained near 0%
- Remaining Microsoft Edge background processes showed approximately 0% CPU utilization

The substantial reduction in CPU utilization confirmed that the active Microsoft Edge browsing workload accounted for most of the observed CPU pressure.

## Root Cause

The workstation was experiencing high CPU utilization associated with active Microsoft Edge browser workloads.

The virtual workstation had two virtual processors, making sustained browser CPU consumption more noticeable within the guest operating system.

There was insufficient evidence to conclude that Microsoft Edge itself was defective.

## Resolution

The active Microsoft Edge browsing session was closed normally, which removed the high-CPU workload and returned processor utilization to a substantially lower level.

## Troubleshooting Principles Applied

- Established a system resource baseline
- Compared CPU, memory, disk, and network utilization
- Sorted processes by resource consumption
- Narrowed application-level utilization to individual processes
- Used PIDs to distinguish process instances
- Tested a hypothesis before declaring a root cause
- Used the least disruptive remediation first
- Protected potential user work before terminating applications
- Re-tested the original condition after remediation
- Documented before-and-after evidence

## Lessons Learned

- High CPU utilization is a symptom, not automatically a root cause.
- A single Task Manager snapshot may not accurately represent sustained system behavior.
- Multiple processes can belong to the same application.
- Modern browsers use multiple processes for tabs, rendering, GPU operations, networking, and other functions.
- Closing one high-resource process may not resolve an application-wide workload issue.
- Virtual machine resource allocation must be considered when evaluating performance.
- Troubleshooting conclusions should be based on verification rather than assumptions.