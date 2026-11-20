# Plan: ble-fixes-1

## What Sean asked for
"Start the next repo on the shortlist" (custom-components/ble_monitor), "respecting any CONTRIBUTING.md in the repo as a standing rule": fix real bugs and send each fix to the original project as its own pull request.

## Contribution rules
- **Written rules:** there is no CONTRIBUTING.md, pull request template or code of conduct, on any branch or at the organisation level. The project sets its rules through its checks and docs:
  - **Style check on every pull request:** pre-commit, with whitespace and file checks, a private-key check, pyupgrade, isort and a spelling check (codespell).
  - **Tests:** pytest on Python 3.14 with `requirements_test.txt`, plus Home Assistant's own checks (HACS, hassfest).
  - **New devices** need a docs page from the template in `docs/developer_docs/adding_device_document.md`. This doesn't apply to a bug fix.
- **What the maintainer does in practice:**
  - Parser fixes come with a regression test in `custom_components/ble_monitor/test/` that feeds a raw advertisement to the parser. His most common request is "fix the pre-commit errors" or "the tests fail".
  - Small, single-purpose fixes are merged within a day or two, often with just "thanks".
  - He closes changes he sees no need for, such as extra machinery or workflow changes.
  - Titles are short imperative sentences with no prefix; the title becomes the release-note line. Descriptions are a short paragraph or Problem / Fix sections, with "Fixes #N" when there is an issue.
  - He bumps the version himself. There is no changelog file. He merges with merge commits.
- **What we'll follow:** all of the above. One small fix, one pull request. Nothing touches the version, workflows, translations or dependencies.

## What we found
- **#1501 is a real bug on today's code, with no pull request yet.** Every Bluetooth advertisement from a Beckett boiler controller that the parser doesn't recognise (for example a new model) writes a warning with a full error trace to the log. The reporter saw dozens of them.
- **A second bug in the same function, with no issue:** the project's documented way to collect data from an unrecognised device (the "report unknown" setting) works backwards for Beckett. It hides the data of supported Beckett devices, and never shows the data of the unrecognised ones.
- **#1504 is already fixed** on the current code (merged #1535); the issue was just never closed. Two older issues (#1510, #1559) are fixed too.
- **#1622 is not a bug in this project:** the reporter's Python install mixes files from two versions of a library this project doesn't control, and the maintainer's open #1634 wouldn't change that.

## What we'll change
1. **#1501 Unknown Beckett advertisements:** in the Beckett parser (`custom_components/ble_monitor/ble_parser/beckett.py`, `parse_beckett`):
   - an advertisement the parser can't decode (unknown model or state, or too short) is skipped without a warning or error trace;
   - with "report unknown" set to Beckett, that advertisement is logged in the standard "unknown device" line that the other parsers use, so owners can share the data;
   - "report unknown" set to Beckett no longer hides the data of supported Beckett devices;
   - only the errors the decoder can actually raise for such data are caught; anything else still reaches the project's existing per-advertisement error handling.
   - **Checks:**
     - new tests in `custom_components/ble_monitor/test/test_beckett.py`, which fail on the current code: an unknown model gives no warning; with "report unknown" it gives the standard line; a supported device with "report unknown" still returns its data; a too-short advertisement gives no warning;
     - the existing Beckett test passes unchanged;
     - the full test suite and the style check pass (run from a folder named `ble_monitor`, as the project's checks do).
2. **Send upstream, only after Sean's final approval on this PR:**
   - The fix gets its own clean branch from the original project's latest `master`, with no plan file.
   - Sean opens the pull request to custom-components/ble_monitor from his own account. Its full title and description are shown here first.
   - Draft title: "Handle unknown Beckett advertisements"

## Who's on it
Researcher (done), Engineer, QA, Security Officer, Auditor.

## Risks
- **No real data in the issue:** #1501 has no raw advertisement, so the test uses the project's existing Beckett test packet with the product code changed to the one in the report. The pull request will say so.
- **Not new device support:** the fix stops the log flood and lets owners collect data; it doesn't add support for the unrecognised model.
- **Log level:** the "unknown device" line uses the same level as the other parsers; the maintainer may prefer another.

## Out of scope
- #1504, #1510 and #1559 (already fixed), #1622 (not this project's bug), #1531 (an intentional message asking for data).
- The maintainer's open #1634 (library swap) and the other open pull requests.
- Feature requests and new-sensor requests.
