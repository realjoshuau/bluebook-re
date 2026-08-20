# `win_app_tools.node`

Lockdown is handled partially through a custom Node binary called `win_app_tools.node`

`win_app_tools.node` performs double-duty. It first locks down the machine, does anti-VM and anti-RDP checks (that from first glance seem pretty bypassable) and secondly emits events back into the Integrity Service Worker:

```js
native.init(nativeEventEmitter.emit.bind(nativeEventEmitter));
```

The JavaScript library only accepts these events from the native library.

| Native event | Meaning |
|---|---|
| `ON_KEY_BLOCKED` | A selected keyboard combination/key was swallowed by the low-level keyboard hook. |
| `NEW_PROCESS` | A process monitor observed a process event. The native capability exists, although the current main bundle has no visible call site that starts the generic process monitor. |
| `NEW_WINDOW` | A window monitor observed a new window. |
| `FOCUS_WINDOW` | The system foreground window changed; main forwards a lockdown focus-change IPC event. |
| `NEW_GRAMMARLY_PROCESS` | A Grammarly desktop process appeared; main forwards `grammarly-detected` IPC event |
| `KEYBOARD_LAYOUT_CHANGED` | The active keyboard layout changed; main parses the native payload and forwards the normalized layout to the renderer. |

The module imports these functions from Windows:

### Low-level process and object inspection

`ntdll.dll` imports include `NtQuerySystemInformation`, `NtQueryInformationProcess`, `NtQueryVirtualMemory`, `NtReadVirtualMemory`, `NtOpenProcess`, `NtOpenProcessToken`, `NtAdjustPrivilegesToken`, `NtOpenKey`, `NtEnumerateKey`, `NtOpenDirectoryObject`, `NtQueryDirectoryObject`, `NtQueryObject`, `NtCreateFile`, `RtlGetVersion`, and `RtlCompareMemory`.

### Hooking and in-process memory work

Important `Kernel32` imports include `VirtualAlloc`, `VirtualProtect`, `VirtualFree`, `FlushInstructionCache`, `GetModuleHandleA/W`, `GetModuleHandleExA`, `GetProcAddress`, `FreeLibrary`, `VirtualQuery`, `VirtualQueryEx`, and `ReadProcessMemory`.

Those are the primitives used by the five-byte detour/trampoline implementation and the entry-point checker. `GetModuleHandleExA` with the “from address” mode is also used to attribute an unexpected jump target to a loaded module.


### Window, keyboard, mouse, and accessibility controls

`USER32.dll` imports include:

- `SetWindowDisplayAffinity` and `GetWindowDisplayAffinity`;
- `SetWindowsHookExA/W`, `UnhookWindowsHookEx`, and `CallNextHookEx`;
- `SetWinEventHook` and `UnhookWinEvent`;
- `GetForegroundWindow`, `WindowFromPoint`, `GetCursorPos`, and `GetWindowThreadProcessId`;
- `SetWindowPos`;
- `GetWindowLongA/W`, `SetWindowLongA/W`, and `CallWindowProc`;
- keyboard-layout APIs and display-enumeration APIs.

This is the core of the native lockdown UI surface.

### Process management and anti-VM checks

Other imports include Toolhelp snapshots, `Process32First/Next`, `CreateProcessW`, `TerminateProcess`, `OpenProcess`, `QueryFullProcessImageNameW`, `K32EnumProcessModules`, firmware-table APIs, CPU/memory/storage APIs, WLAN APIs, IP Helper APIs, registry APIs, COM/WMI support, display configuration, and per-monitor DPI.

>[!NOTE] 
> Presumably, the WLAN API & IP Helper APIs are used to check that the test is being taken at the expected location, without having to use the Windows GeoLocation APIs. This is also a useful anti-VM check (since a trivial VM would not pass through the WLAN controller, and show up as a Ethernet adapter.)

## Node `main` API surface

The binary also allows some functions to be called.


| Method | Callback RVA | Observed responsibility |
|---|---:|---|
| `init` | `0x2dfe0` | Stores the JavaScript native-event callback and initializes cross-thread delivery. |
| `lockShortcutKeys` | `0x2e300` | Installs the global low-level keyboard hook. |
| `unlockShortcutKeys` | `0x2e390` | Removes the low-level keyboard hook. |
| `isRDPSession` | `0x2e280` | Returns the [Windows `SM_REMOTESESSION` state.](https://learn.microsoft.com/en-us/windows/win32/termserv/detecting-the-terminal-services-environment) Note: This does not detect ALL remote desktop applications (e.g. Parsec, RustDesk, etc), but detects specifically Windows Terminal Services RDP.|
| `detectVM` | `0x32590` | Runs multi-source VM/sandbox evidence collection and returns a nested evidence object. |
| `cleanup` | `0x2e210` | Stops hooks/monitors and releases native resources. |
| `getSystemInfo` | `0x2e600` | Returns a group of native system-information collector functions. |
| `getSystemVolume` | `0x2e950` | Reads master volume and mute state. |
| `setSystemVolume` | `0x2ebd0` | Changes master volume/mute state. |
| `preventSleep` | `0x24b00` | Creates and activates Windows system/display power requests. |
| `allowSleep` | `0x24c60` | Clears/releases those power requests. |
| `isExplorerRunning` | `0x12fd0` | Finds `explorer.exe`. |
| `terminateExplorer` | `0x12ec0` | Attempts to terminate Explorer processes. |
| `startExplorerMonitoring` | `0x13910` | Watches for Explorer returning. |
| `stopExplorerMonitoring` | `0x139f0` | Stops that monitor. |
| `startExplorer` | `0x13120` | Relaunches `explorer.exe`. |
| `startProcessMonitoring` | `0x13b30` | Starts generic process/window monitoring. |
| `stopProcessMonitoring` | `0x03780` | Stops generic process/window monitoring. |
| `startFocusMonitoring` | `0x15c80` | Installs a global foreground-window WinEvent hook. |
| `stopFocusMonitoring` | `0x15d50` | Removes the focus WinEvent hook. |
| `startGrammarlyMonitoring` | `0x13a20` | Watches for `Grammarly.Desktop.exe`. |
| `stopGrammarlyMonitoring` | `0x13b00` | Stops the Grammarly monitor. |
| `isGrammarlyRunning` | `0x137b0` | Checks for the Grammarly desktop process. |
| `terminateGrammarly` | `0x136a0` | Attempts to terminate matching Grammarly processes. |
| `lockMouseToBluebook` | `0x29340` | Installs a low-level mouse hook and confines effective mouse interaction to Bluebook. |
| `unlockMouseFromBluebook` | `0x29400` | Removes the mouse hook. |
| `setRendererReference` | `0x0b650` | Stores the Electron/renderer native window reference needed by window protection and mouse logic. |
| `setRendererContentProtection` | `0x0b7e0` | Applies/removes the renderer window's display-affinity policy. [SPECULATION: This sets display affinity of the CURRENT WINDOW so that it can't be screen captured ] |
| `getRendererContentProtection` | `0x0b980` | Reads/interprets the renderer window's current affinity. |
| `startMSAABlocking` | `0x29680` | Installs a `WH_CALLWNDPROC` hook and suppresses selected `WM_GETOBJECT` accessibility traffic. |
| `stopMSAABlocking` | `0x29770` | Removes the MSAA hook and related state. |
| `initHooksAndMonitors` | `0x04510` | Creates the five protected-entry descriptors, installs the two native detours, and returns per-entry initialization statuses. |
| `checkExpectedMemory` | `0x052e0` | See below. |
| `detectDebugger` | `0x29df0` | Checks the main process and direct child processes for local/remote debugger attachment. |
| `setLockdownState` | `0x0ba90` | Sets the native lockdown flag used by display-affinity interception and performs lockdown-adjacent native handling. |
| `getAndClearWDACallCounts` | `0x075c0` | Atomically returns/reset counts of non-lockdown WDA calls as `{n,x,m}`. |
| `initTelemetrySender` | `0x0b3b0` | Initializes the native telemetry sender used by the WDA interception path. |
| `getDefaultKeyboardLanguage` | `0x18160` | Returns the default Windows keyboard layout/language. |
| `getAvailableKeyboardLanguages` | `0x186f0` | Enumerates installed/available layouts. |
| `setKeyboardLanguage` | `0x18bc0` | Activates a selected Windows keyboard layout. |
| `startKeyboardLayoutMonitoring` | `0x19590` | Starts layout-change monitoring and native callback delivery. |
| `stopKeyboardLayoutMonitoring` | `0x196c0` | Stops the layout monitor. |

## Hooks 


| `function` value | Enum name | DLL | Bluebook installs a detour? | What the expectation represents |
|---:|---|---|---|---|
| `0` | `LoadLibraryExW` | `KernelBase.dll` | Yes | Bluebook's installed five-byte detour. |
| `1` | `SetWindowDisplayAffinity` | `user32.dll` | No | Expected entry-point bytes for the Windows API. |
| `2` | `NtUserSetWindowDisplayAffinity` | `win32u.dll` | Yes | Bluebook's installed five-byte detour. |
| `3` | `GetWindowDisplayAffinity` | `user32.dll` | No | Expected entry-point bytes for the Windows API. |
| `4` | `NtUserGetWindowDisplayAffinity` | `win32u.dll` | No | Expected entry-point bytes for the NT-user API. |

Bluebook checks that these functions either:
- Are properly hooked by BlueBook
- Are _not_ hooked by any other program

#### Why watch both `user32.dll` and `win32u.dll`?

Modern Windows GUI APIs often have a user-mode wrapper in `user32.dll` that reaches a lower-level stub in `win32u.dll`. Watching both levels closes a simple observation gap: code can call the documented wrapper, while other code may call the lower-level NT-user routine directly.

For the set operation Bluebook detours the lower-level `NtUserSetWindowDisplayAffinity` path and separately watches the public `SetWindowDisplayAffinity` wrapper. For the get operation it watches both levels. 

<detail>

<summary> (The detour is a simple x86 trampoline.) </summary>

### The detour installer

The installer begins at native RVA `0x3820`. Its behavior is a small, conventional x86 trampoline:

1. Resolve the target function address.
2. Ensure the prologue is usable for the expected five-byte replacement.
3. Allocate 10 bytes of executable memory with:
   - size `10`;
   - `MEM_COMMIT | MEM_RESERVE` (`0x3000`);
   - `PAGE_EXECUTE_READWRITE` (`0x40`).
4. Copy the original first five target bytes into the trampoline.
5. Append `E9 <rel32>` at trampoline offset 5 so execution returns to `target + 5`.
6. Change protection on the first five target bytes with `VirtualProtect`.
7. Write `E9 <rel32>` from the target to Bluebook's replacement handler.
8. Save the installed five bytes as the post-install expected sequence.
9. Call `FlushInstructionCache` for the trampoline and target.
10. Restore the target's previous page protection.

In schematic form:

```text
target before:  [ original byte 0 ... original byte 4 ][ target + 5 ... ]

trampoline:     [ original byte 0 ... original byte 4 ][ E9 back-to-target+5 ]

target after:   [ E9 jump-to-Bluebook-handler           ][ target + 5 ... ]
```

</detail>

### What happens during `initHooksAndMonitors()`

Before any preload or renderer code is run, the Electron main process called `initHooksAndMonitors()`

The native routine resolves the five target functions, creates descriptors, installs the two detours, and returns an array of result records.

Initialization can therefore report failures such as:

- the function could not be resolved;
- the target did not have a suitable expected prologue;
- executable trampoline allocation failed;
- changing target page protection failed.

Main immediately filters out `status === 0` and appends only failures to an in-memory <abbr title="Unknown what this means, it's just called this internallly"> HMOD(?) </abbr> queue.

### `checkExpectedMemory()` in detail

The Node-API callback begins at RVA `0x52e0`. It iterates the global protected-function descriptors and invokes the byte verifier at RVA `0x3a30`. It then converts native result records into a JavaScript array.

For each descriptor, the logic is effectively:

```text
resolve/use protected entry address
compare current entry bytes with the descriptor's expected bytes

if equal:
    status = SUCCESS
else:
    bytes = lowercase/normalized hexadecimal rendering of observed bytes
    if observed[0] != 0xE9:
        status = UNKNOWN_OPCODE
    else:
        status = UNKNOWN_HOOK
        destination = entry + 5 + signed_rel32(observed[1..4])
        try to find loaded module containing destination
        if found:
            module = resolved module name/path
```

Each returned object has this schema:

```ts
type HookMemoryStatus = {
  function: 0 | 1 | 2 | 3 | 4;
  status: 0 | 1 | 2 | 3 | 4 | 5 | 6;
  module?: string;
  bytes?: string;
};
```


The exact enum is present in the JavaScript bundle:

| `status` | Enum | Precise interpretation |
|---:|---|---|
| `0` | `SUCCESS` | Current/installed bytes match the expectation, or initialization succeeded. Main filters this out; it does not generate an HMOD packet. |
| `1` | `INSUFFICIENT_PROLOGUE_LENGTH` | Hook installation could not safely obtain the required prologue for the five-byte detour. This is primarily an initialization/install result. |
| `2` | `VIRTUAL_ALLOC_FAILED` | Allocation of the executable trampoline failed. Initialization/install result. |
| `3` | `VIRTUAL_PROTECT_FAILED` | The target's protection could not be changed for patching. Initialization/install result. |
| `4` | `UNKNOWN_OPCODE` | The current bytes differ from expected, and the current first byte is not `0xE9`. The name is slightly misleading: the observed code is not fully disassembled and judged invalid; it is classified by the non-`E9` first byte. |
| `5` | `UNKNOWN_HOOK` | The current bytes differ from expected and begin with an unexpected `E9 rel32` jump. The checker tries to resolve the jump destination to a loaded module. |
| `6` | `FUNCTION_ADDRESS_NOT_FOUND` | The module/function address could not be resolved for the protected entry. It can arise when establishing or checking the descriptor. |

Status values `1`–`3` describe the detour setup machinery. Periodic memory checks are principally expected to return `0`, `4`, `5`, or possibly `6`.

It's useful to note that `checkExpectedMemory()` will NOT reinstall the trampoline hook but will ONLY log them.

## The two installed native hooks

`win_app_tools` also hooks into two native modules: 
1. `LoadLibraryExW` (loaded-DLL tracking)
2. `NtUserSetWindowDisplayAffinity` (screen capture tracking)

### 1. `LoadLibraryExW`: loaded-DLL tracking

Bluebook collects information about loaded DLLs and may automatically terminate the application if unknown DLLs are injected.

### 2. `NtUserSetWindowDisplayAffinity`: policy enforcement and auditing

Depending on `setLockdownState`, this hook can have two modes.

#### Outside native lockdown

The hook largely passes calls through BUT it also counts what values were requested.

| Requested value | Windows meaning | Counter |
|---:|---|---|
| `0` | `WDA_NONE` | `n` |
| `1` | `WDA_MONITOR` | `m` |
| `17` / `0x11` | `WDA_EXCLUDEFROMCAPTURE` | `x` |

`getAndClearWDACallCounts()` returns `{n, x, m}` and then resets the three counts. If any of `{n, x, m}` are non-zero, then it submits this as telemetry with with category `cat_DAFT`, segment `seg_DAFT_NLC`, and tags using the same one-letter IDs.

The native switch also recognizes internal/custom values `3` and `4`, normalizing them to `1` and `0` before calling the original routine. No clue why they have custom values (maybe for entering/exiting "lockdown mode" they record this?) [The JavaScript enum labels `4` as a custom WDA-none condition in telemetry.]

#### During native lockdown

When `setLockdownState(true)` has set the native lockdown flag, the handler prevents other code from attempting to show the window (e.g another application attempts to grab the window handle and forcefully show the window)


- request `0` is classified as `LockdownCallNone` (`seg_DAFT_LCN`);
- request `17` is classified as `LockdownCallExclude` (`seg_DAFT_LCX`);
- custom request `4` is classified as `LockdownCallCustomNone` (`seg_DAFT_LCCN`);
- those selected calls are reported/suppressed and return a success-like result without invoking the original target;
- other values are passed to the original routine as affinity `1` (`WDA_MONITOR`).


Main also checks the renderer window's effective content-protection state and can reapply it, producing segments such as `LockdownReapplied`, `AffinityReturnedNone`, `AffinityReturnedExclude`, `AffinityReturnedUnknown`, invalid-handle, or other error states.

Presumably, this is all submitted into the integrity blockchain.

<detail>

<summary> **The HMOD_STATUS lifecycle** (not cleaned yet) </summary>


## Complete `HMOD_STATUS` lifecycle

### Stage 1: initialization results enter a failure queue

Main holds a module-scoped array, reconstructed as `R7kv`, initially `[]`.

During main-window startup:

```js
const failures = native.initHooksAndMonitors()
  .filter(result => result.status !== 0);

if (failures.length) hmodQueue.push(...failures);
```

Success records are discarded. The queue therefore contains diagnostic failures only.

### Stage 2: lockdown starts a 30-second memory monitor

On lockdown entry main calls, in close sequence:

- the native lockdown-state setter;
- `startMemoryMonitoring(mainWindow)`;
- the other keyboard, Explorer, focus, mouse, and environment controls.

The HMOD memory monitor interval is `30,000` ms. Each tick calls `checkExpectedMemory()`, filters `status !== 0`, and appends any failures to the same queue.

On lockdown exit, main calls `stopMemoryMonitoring()` and clears the interval. Native hooks may remain part of the initialized process until native cleanup, but the recurring HMOD poll is scoped to the lockdown lifecycle.

### Stage 3: batch and send over Electron IPC

After a poll, if the queue is non-empty and the target `BrowserWindow` is not destroyed, main sends:

```js
mainWindow.webContents.send("hmod-status", {
  statuses: hmodQueue
});

hmodQueue = [];
```

Consequences:

- one IPC message may contain multiple function failures;
- initialization failures can be batched with a later periodic failure;
- no message is sent for an all-success poll;
- the queue is cleared only after the send branch;
- if the window is destroyed, the visible code does not send or clear that queue at that moment;
- a persistent mismatch can generate another failure record every 30 seconds, because the periodic path does not repair it or suppress duplicates.

For example:

```json
{
  "statuses": [
    {
      "function": 1,
      "status": 5,
      "module": "example.dll",
      "bytes": "e9..."
    }
  ]
}
```


### Stage 4: preload exposes a narrow listener

The preload bridge exposes `onHModStatus(callback)` rather than raw `ipcRenderer`. Its generic listener wrapper removes the privileged Electron event argument before invoking renderer code and returns an unsubscribe closure.

The renderer therefore receives only the payload object, not the Electron `IpcRendererEvent` or arbitrary IPC capabilities.

### Stage 5: renderer writes two records

The renderer's handler is semantically:

```js
window.$electron.onHModStatus(async payload => {
  const rosterEntryId = rosterStore.getReid();

  try {
    await telemetryManager.recordTelemetryItem(
      rosterEntryId,
      "HMOD_STATUS",
      JSON.stringify(payload)
    );
  } catch (error) {
    console.error("Asynchronous HMOD_STATUS telemetry failed:", error);
  } finally {
    if (integrityManager.isIntegrityManagerWorkerInitialized(rosterEntryId)) {
      integrityManager.onExamIntegrityEvent(
        rosterEntryId,
        "HMOD_STATUS",
        payload
      );
    }
  }
});
```

This produces two related but distinct data paths:

| Path | Name/data | Reliability behavior |
|---|---|---|
| Telemetry | item name `HMOD_STATUS`; payload serialized with `JSON.stringify` | Attempted first. Failure is logged. |
| Exam integrity | event name `HMOD_STATUS`; original structured payload | Runs in `finally`, so telemetry failure does not prevent integrity submission, provided that worker is initialized. |

The integrity manager is keyed by roster-entry ID and maintains a dedicated worker/chain for that exam participant. `onExamIntegrityEvent()` sends `{name, payload}` into that worker. In a non-practice exam, this event becomes an encrypted, hash-linked integrity transaction and is later included in block synchronization to the risk/integrity service at `https://drs-dap-risk-prod.collegeboard.org`.

This does not mean every HMOD message causes an immediate network request. The worker is offline-capable, persists records, batches transactions into blocks, and synchronizes periodically/finally. The static client proves local recording and worker handoff; exact server-side scoring or enforcement is not present in the supplied app.

For practice exams the integrity implementation is effectively a no-op, which matches the broader observation that the full integrity chain is enabled for non-practice administrations. The renderer telemetry path can still exist independently, but the meaningful exam-integrity-chain route is the non-practice path.

### What one `HMOD_STATUS` integrity event means

One integrity event corresponds to one main-to-renderer IPC batch, not necessarily one function. Its payload contains a `statuses` array. If two entries fail on a single poll, the integrity chain receives one `HMOD_STATUS` event containing two status objects.

The integrity event preserves ordering relative to other exam events through the worker's transaction chain. It becomes part of a hash-linked audit record; it is not itself “a blockchain block.” The chain links exam transactions to previous transaction hashes and later groups them into hash-linked blocks, similar to a blockchain.

### No visible direct quit on HMOD

The renderer handler records the event. It does not directly call quit, show a blocking dialog, or mark a local security violation. The server may interpret the event as risk evidence, but that policy is not included here.

This differs from confirmed VM detection, RDP handling, and the JavaScript self-healing callback, which have visible local enforcement/quit behavior in the client.

## HMOD interpretation matrix

| Situation | Likely record | Local meaning | Important caveat |
|---|---|---|---|
| Hook install works and all five entries match | No HMOD | All returned statuses are `0` and filtered out. | Absence of HMOD proves only these checks passed when sampled. |
| Bluebook cannot allocate a trampoline | Function `0` or `2`, status `2` | Native hook initialization failure. | Resource/security-software conflict can be a cause; not necessarily tampering. |
| Bluebook cannot change target page protection | Function `0` or `2`, status `3` | Native detour cannot be installed. | Memory-protection policy or another product may be involved. |
| A watched entry starts with different non-`E9` bytes | Status `4` plus `bytes` | Expected sequence changed; classifier did not see an `E9` detour. | Most likely tampering or Windows ?? |
| A watched entry starts with a different `E9` | Status `5`, `bytes`, optionally `module` | Unexpected relative-jump detour. | Could be malicious? |
| Function resolution fails | Status `6` | Expected module/function address unavailable. | OS version/module-load issues can be causes. |
| A mismatch persists | Repeated batch every ~30 s | The watchdog continues observing the same bad state. | No visible deduplication or periodic repair. |

</detail>

## Other lockdown primitives in the same `.node` file

### Low-level keyboard hook

`lockShortcutKeys()` installs `SetWindowsHookExA(WH_KEYBOARD_LL = 13, ...)` globally. The callback returns `1` to swallow these cases and emits `ON_KEY_BLOCKED` with the listed native label:

| Key/condition | Label |
|---|---|
| Alt+Tab | `ALT_TAB_KEY` |
| Ctrl+Esc | `CTRL_ESC_KEY` |
| Alt+Esc | `ALT_ESC_KEY` |
| Print Screen (`VK_SNAPSHOT`) | `PRINT_SCREEN` |
| Left/right Windows key | `WINDOWS_KEY` |
| `VK_PACKET` | `VK_PACKET` (maybe [virtual input blocking?](https://learn.microsoft.com/en-us/windows/win32/inputdev/virtual-key-codes)) |

The hook checks the low-level Alt flag and `GetAsyncKeyState(VK_CONTROL)` where needed. Other keys are delegated through `CallNextHookEx`. `unlockShortcutKeys()` calls `UnhookWindowsHookEx`.

This is only in user-mode, so CTRL+ALT+DELETE would still work.

### Mouse confinement

`setRendererReference()` obtains and stores the relevant native renderer/Bluebook window reference. `lockMouseToBluebook()` installs `WH_MOUSE_LL = 14`.

For non-move mouse actions, the callback examines the cursor location with `GetCursorPos`, resolves the target with `WindowFromPoint`, obtains its process/window relationship, and compares it with the stored Bluebook reference. When the interaction falls outside the accepted Bluebook relationship, the code uses `SetWindowPos` to reassert/raise Bluebook and returns `1` to swallow the input.

### MSAA/accessibility blocking

`startMSAABlocking()` installs a thread-scoped `WH_CALLWNDPROC = 4` hook using `SetWindowsHookExW`. It watches `WM_GETOBJECT` (`0x003d`), the message used to obtain Microsoft Active Accessibility/UI Automation accessibility objects.

The code subclasses affected windows by replacing `GWL_WNDPROC` (`-4`), stores original procedures in a map, suppresses/handles selected `WM_GETOBJECT` messages, and delegates other messages through the original window procedure.

(This code does not always execute, but seems to be dependent on if the student has accomodations enabled in their exam)


### Foreground-window monitor

`startFocusMonitoring()` calls:

```text
SetWinEventHook(
    EVENT_SYSTEM_FOREGROUND = 3,
    EVENT_SYSTEM_FOREGROUND = 3,
    ...,
    WINEVENT_OUTOFCONTEXT = 2
)
```

The callback identifies the foreground window's PID and process information and emits `FOCUS_WINDOW`. Main forwards this as `lockdown-window-focus-changed`. The start result uses the shared monitor enum:

- `0` `SUCCESS`;
- `1` `FAILED_WINDOWS_EVENT_HOOK_SETUP`;
- `2` `FAILED_PROCESS_HANDLER_ALREADY_EXISTS`.

### Explorer and Grammarly controls

Explorer routines enumerate, terminate, monitor, and restart `explorer.exe`. Bluebook uses this to remove the desktop shell and give more of a "lockdown feel" (and restores it on exit.)

There are specific routines to target Grammarly, for some reason?
A new matching process produces `NEW_GRAMMARLY_PROCESS`, which main converts to the renderer event `grammarly-detected`.

Termination functions use a separate enum:

- `0` `SUCCESS`;
- `1` `SNAPSHOT_FAILED`;
- `2` `NO_PROCESSES_FOUND`;
- `3` `FAILED_TO_TERMINATE_ALL_PROCESSES`.

### Generic process/window monitoring

The add-on exposes `startProcessMonitoring`, `stopProcessMonitoring`, `NEW_PROCESS`, and `NEW_WINDOW`. 

Presumably, they collect process information and match it server-side for obvious names such as `parsecd.exe` and other remote-desktop or obvious cheating software.

### Native lockdown state

`setLockdownState(true/false)` updates the native flag used by the `NtUserSetWindowDisplayAffinity` hook (and also attempts to kill/restart [`ctfmon.exe`](https://superuser.com/questions/6624/what-in-the-world-is-ctfmon-exe) ??)


## Environment detection

### Remote Desktop

`isRDPSession()`  calls `GetSystemMetrics(SM_REMOTESESSION = 0x1000)` and returns a boolean.

This is one of the only ways that Bluebook will automatically close an exam session for a security violation.

### Debugger detection

`detectDebugger()` returns:

```ts
{
  debuggingMainProcess: boolean;
  debuggingSubProcess: boolean;
}
```

`debuggingMainProcess` is true if `IsDebuggerPresent()` succeeds or `CheckRemoteDebuggerPresent(GetCurrentProcess(), ...)` reports a debugger.

For `debuggingSubProcess`, it takes a Toolhelp process snapshot, enumerates processes whose parent PID is the current Electron main process, opens each with process-query access, and runs `CheckRemoteDebuggerPresent`. Thus renderer, GPU, and utility children can contribute to `debuggingSubProcess`.

### VM and sandbox detection

`detectVM()` is an evidence collector. 

he returned top-level object contains:

```text
virtualBox
vmware
parallels
hyperV
sandboxie
qemu
systemSpecs
acpiTable
rawData
```

Observed nested evidence fields are:

| Group | Fields |
|---|---|
| `virtualBox` | `deviceObject`, `driverObject`, `pciHardwareId`, `signatureScanFirmware`, `signatureScanRSMB`, `hypervisorNameMatch`, `pnpDevices` |
| `vmware` | `deviceObject`, `pciHardwareId`, `signatureScanFirmware`, `signatureScanRSMB`, `hypervisorNameMatch`, `pnpDevices` |
| `parallels` | `deviceObject`, `pciHardwareId`, `signatureScanFirmware`, `signatureScanRSMB`, `hypervisorNameMatch`, `pnpDevices` |
| `hyperV` | `deviceObject`, `pnpDevices` |
| `sandboxie` | `handleTable`, `memoryTag`, `virtualRegistry` |
| `qemu` | `driveNames`, `registryKeys`, `biosInfo`, `processes`, `pnpDevices`, `signatureScanFirmware`, `signatureScanRSMB` |
| `systemSpecs` | `isSingleCore`, `multiplePhysicalCpus`, `lowTotalMemory`, `lowTotalDiskSpace`, `noAudioDevices`, `hypervisorGatewayAddress` |
| `acpiTable` | `missingSignatures`, `hypervisorSignatures`, `oemIds` |
| `rawData` | `rawFirmware`, `rawRSMB` |

The native strings and call graph show evidence from:

- CPUID hypervisor leaves/signatures, including leaf `0x40000000`;
- firmware/SMBIOS and ACPI tables;
- PnP/PCI/device objects and driver objects;
- registry keys and WMI queries;
- drive models and process names;
- Sandboxie handles, memory tags, registry, mutex, and API indicators;
- CPU count, physical CPU topology, RAM, storage, audio devices;
- network adapters/gateway patterns.

Strings include `VMWARE`, `VMWare`, `VMwareVMware`, `VBoxVBoxVBox`, `VirtualBox`, `Parallels Hv`, `HYPER_V`, `QEMU`, `BOCHS`, `Slirp`, `qemu-ga.exe`, `vdagent.exe`, `vdservice.exe`, and Sandboxie identifiers.

The main JavaScript assigns weights to the evidence and uses threshold `10`:

- score `>= 10`: `virtual-machine-detected` and security violation `default-sf001`;
- score `1..9`: `virtual-machine-suspected`;
- score `0`: no VM finding.

Strong signature/PnP/hypervisor evidence commonly carries weight `10`; weaker generic hardware constraints carry `1`; some raw evidence fields carry `0` and are retained as diagnostic context.

## System information and device-control capabilities

`getSystemInfo()` returns a native object with these callable collectors:

- `getCpuRawData`;
- `getOsInfoRawData`;
- `getSystemRawData`;
- `getProcessesRawData`;
- `getGraphicsRawData`;
- `getFsSizeRawData`;
- `getBatteryRawData`;
- `getWifiConnectionsRawData`;
- `getLoadedDLLs`;
- `getRegistryRawData`.

The native object/field strings show additional detail for displays, adapters, topology, secure boot, driver-signature enforcement (maybe they're worried about someone using a bad driver to dump memory from the Bluebook app? I'd be surprised if anyone's done that before!), BIOS/product identity, registry/AppInit DLL state, tablet mode, WLAN SSID/BSSID/security, network adapters, monitor connection/basic display data, dimensions, active/primary state, and scale factor.

Other utility capabilities include:

- system master volume/mute get/set;
- power requests using `PowerCreateRequest`, `PowerSetRequest`, and `PowerClearRequest` for system/display availability;
- default and available keyboard-layout enumeration;
- activation with `ActivateKeyboardLayout`;
- keyboard-layout monitoring, with payload data such as locale, HKL/code, and label;
- native telemetry sender initialization and a segment-update-success callback.

