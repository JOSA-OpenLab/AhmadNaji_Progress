WinBoat Issue Triage - High GPU Usage #796

Date: June 16, 2026
Issue: WinBoat v0.9.0 causes ~50% GPU usage spike on idle - Framework 16, Kubuntu 25.10/26.04

Triage Actions Performed:
1. Reviewed issue report for completeness and uniqueness. Reporter provided versions, hardware, logs, and clear repro steps.
2. Attempted reproduction on my system: Arch Linux 7.0.9-arch2-1, WinBoat v0.9.0 via AUR using yay. 
3. Result: Issue did not reproduce. GPU usage remained at baseline with WinBoat idle.

Triage Conclusion: Cannot confirm bug due to environment differences. Not a false positive, duplicate, or stale issue. Classified as 'Needs More Info' and potentially 'Hardware-Specific'.

Clarifying Questions Asked in Issue Comment:
- Requested radeontop/nvtop output to identify GPU-heavy process
- Asked to confirm Wayland vs X11 session  
- Suggested testing AppImage to isolate packaging issues

Status: Issue left open. Did not close, duplicate, or mark as stale. Recommended maintainers seek repro from someone with Framework 16 + Ubuntu.

Reasoning: Closing as 'Cannot Reproduce' would be premature since reporter's environment is very different from mine and issue is clearly occurring for them. Next step depends on reporter providing additional diagnostics.
