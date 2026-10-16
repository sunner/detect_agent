# Desktop environment marker evidence

Recorded on 2026-10-16. WorkBuddy, Grok Bot and QoderWork-family rules below are
based on static inspection of installed macOS distribution artifacts. No agent
was launched and no real environment values, user configuration or session
contents were collected for these tests. Locations refer to the packaged files,
including minified line numbers, not unpublished source repositories. The
synthetic test cases do not establish coverage of every platform or execution mode.

## WorkBuddy

Inspected WorkBuddy 5.8.3, bundle ID `com.tencent.workbuddy.mac`.

- `Contents/Resources/app.asar` SHA-256:
  `a7eff702411704d140d7d8d706fe9e711d9b13e304928481c9eba0af9c06f731`.
- Inside that archive, `main/main-bootstrap.js:77` (`buildDaemonProcessEnv`)
  constructs `WORKBUDDY_APPLICATION_NAME`, `WORKBUDDY_APP_NAME` and
  `WORKBUDDY_IS_PACKAGED`; the daemon spawn at line 75 merges this object into
  the inherited environment. `main/sidecar-manager.js:16`
  (`buildSidecarProcessEnv`) also preserves inherited variables for its spawn.
- `Contents/Resources/app.asar.unpacked/cli/dist/codebuddy-lite-wb.mjs` SHA-256:
  `9eacdcdaaf3d53877ca4eac53836a446770b9a40f5dc6382b43230c73fdc5448`.
  Its `buildChildProcessEnv` at line 1684 supplies the ordinary shell and PTY
  spawn environments. The intervening cleanup at line 1819 removes trace/request
  fields rather than these product-name markers. The sandbox path at line 1134
  carries the target environment in an execution request; native sandbox
  propagation was not dynamically tested.

The rule accepts exact `WorkBuddy` in either complete product-name key. It does
not accept the unprefixed `APPLICATION_NAME` / `APP_NAME`, packaging flags, or
configuration directories alone. Renamed or branded variants may not match.
The public [product guide](https://www.codebuddy.cn/docs/workbuddy/From-Beginner-to-Expert-Guide/Product-Guide)
describes the product, but does not establish these markers as a stable public API.

## Grok Bot

Inspected Grok Bot 0.71.2, bundle ID `com.anysphere.sand`.

- `Contents/Resources/app.asar` SHA-256:
  `055b087a9bacc3be4e5f80c468edf2fd031dd5b928ab2e7da7e01dd10de0e860`.
- `dist/local-exec-daemon/main.cjs:756` (`iZe`) supplies both `SAND_AGENT=1`
  and `CURSOR_AGENT=1` to the local executor. Line 611 (`pI`) passes this
  environment to the Bash initialization at line 602 (`Jae`) and the Zsh
  initialization at line 608 (`Wae`), which merge it into the child environment.

The rule requires exact `SAND_AGENT=1` and precedes Cursor rules. `CURSOR_AGENT`
is shared compatibility context and is not an alternative Grok Bot marker.
This identifies the inspected local shell path; it does not assert complete
cloud or PowerShell coverage. The [public architecture documentation](https://cursor.com/docs/grok-bot/teams#architecture)
distinguishes the cloud service from the local execution daemon. Grok Bot has a
separate result from Grok Build.

## QoderWork family

The inspected distribution was **QwenWorkCN 1.2.5**, bundle ID
`cn.qwenwork.desktop.mac`, which retains QoderWork runtime naming.

- `Contents/Resources/app.asar` SHA-256:
  `d946457e5e3d7761f7203111775388d5131293a337bf13430e0c26b8993536fb`.
- `out/main/main.js:654` (`composeQoderAgentSdkRuntimeEnv`) supplies
  `QODERWORK_IS_CN=true`. Workspace Bash at line 11824 and background Bash at
  line 11838 supply `QODERWORK_SOURCE_CHAT_ID`, with the distribution marker
  when applicable.
- `ShellExecutionService` at lines 2313–2316 forwards the request environment
  to `QxcSeatbeltBackend.runBash` at line 1048 and its child-process spawn.
  The target-environment filtering at line 888 retains these two marker names.
  The VM command builder at line 1412 also exports them on applicable paths;
  the VM was not started for this inspection.

The rule accepts a non-empty `QODERWORK_SOURCE_CHAT_ID` or exact
`QODERWORK_IS_CN=true`, returning only `qoderwork`. It does not infer region,
commercial branding, or model identity, and does not claim independent
inspection of every QoderWork/QwenWork distribution. Fixtures use a synthetic
chat ID. The [QwenWork introduction](https://qwenwork.cn/docs/product-introduction)
identifies the inspected brand; the [QoderWork hooks documentation](https://docs.qoder.com/qoderwork/hooks)
does not establish these markers for hook invocations.

## Manus

A user-provided Linux desktop-worker runtime observation on **2026-10-16**
reported `MANUS_LONG_DESKTOP_WORKER=1`. The rule accepts only that exact value.
This observation was not reproduced by launching Manus during this contribution;
no inspected application version or artifact hash is claimed. Other `MANUS_`
path, endpoint or identifier variables are not independent detection conditions.
No real desktop identifier, path or endpoint is included in the evidence or tests.

## Grok Build terminal marker

Public source at commit `2bdd1d6a6369de0e8c68132ea4539e9abd9e14a8`
[documents the dual use of `GROK_AGENT`](https://github.com/xai-org/grok-build/blob/2bdd1d6a6369de0e8c68132ea4539e9abd9e14a8/crates/codegen/xai-grok-tools/src/util/env.rs#L31):
the terminal-child marker is exact `1`, while other values can select an agent.
Both the [ordinary terminal path](https://github.com/xai-org/grok-build/blob/2bdd1d6a6369de0e8c68132ea4539e9abd9e14a8/crates/codegen/xai-grok-tools/src/computer/local/terminal.rs#L3315)
and the [sandbox command path](https://github.com/xai-org/grok-build/blob/2bdd1d6a6369de0e8c68132ea4539e9abd9e14a8/crates/codegen/xai-grok-tools/src/computer/local/terminal.rs#L3391)
apply that marker. This extends the existing `grok` result without treating
arbitrary non-empty selector values as a match.

All of these markers can be set or inherited by other processes. They identify
environment context, not an authenticated immediate caller. Existing override
and first-match behavior still apply when several contexts coexist.
