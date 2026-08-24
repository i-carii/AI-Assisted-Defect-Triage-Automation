# AI-Assisted Defect Triage Automation

A process-automation project built during my Security Consulting Engineer internship at Cisco.
It replaced a manual, high-volume defect-review workflow with an automated extraction pipeline
feeding an AI analysis step.

I do not come from a software engineering background. I acted as the architect: I analyzed the
system's behavior, directed AI code generation to implement the tooling, and then hardened it
against the failure modes it hit in real use.

*Note: this write-up describes the engineering approach only. Internal system names, tooling,
customer data, and implementation specifics are deliberately omitted.*

## The Problem

Engineers reviewing a customer's planned network upgrade have to filter large volumes of known
software defects to determine which ones will realistically affect that customer's deployment.
Done manually, that meant reading and transferring up to **2,000 defect records** one at a time
into an analysis step — the single largest time sink in the process.

## What I Built

A browser-based automation tool that walks the defect-tracking interface, expands collapsed
detail panels, extracts the relevant text, and packages it into a structured payload for bulk AI
analysis against the customer's deployment profile.

## Engineering Problems Solved

**Data hidden in nested UI elements.** The first generated scripts returned empty fields —
they couldn't reach content inside collapsed menus. Browser inspection showed the script had to
interact with the page before reading it. I specified logic to locate and expand every detail
control while explicitly avoiding navigation buttons that would advance the record.

**Unpredictable render timing.** Fixed delays were unreliable — the script frequently ran ahead
of the page and captured blanks. I replaced them with a dynamic wait: don't capture until the
new record ID is actually visible on screen.

**Memory pressure at scale.** Processing thousands of records in one session froze the browser.
I added hard iteration ceilings and memory-efficient handling to keep runs bounded.

**Infinite loops on stalled pages.** If the interface lagged and failed to advance, the script
would capture the same record forever. I added ID tracking — on detecting a duplicate, the tool
recognizes it's stuck, retries, and halts safely if it still can't advance.

**Catastrophic data loss on error.** Early versions crashed entirely on a single malformed
record, destroying hundreds of successful captures. I wrapped the extraction loop in error
handling that performs a salvage: on fatal error, stop and persist everything captured so far.

**Browser clipboard restrictions.** Modern browsers block programmatic clipboard writes for
security. I added a graceful fallback — the tool renders a temporary text area, selects the
content, and prompts the user to copy manually.

## Outcome

The most time-consuming manual bottleneck in the workflow became a bulk operation, cutting the
time to produce customer-facing upgrade reports and removing a significant source of
transcription error.

## What I Took Away

AI defaults to happy-path code. The value I added was acting as the QA layer — finding the edge
cases, reproducing the failures, and directing specific fail-safes until the tool held up under
real conditions.
