---
name: harness-browser
description: "Trigger: a task needs a web page that runs behind the owner's browser session, such as an OAuth consent page, an identity login, or a form with a redirect callback. Drives the built-in browser of the agent harness (Orca) to open, inspect, fill and submit pages, capture redirect callbacks, and hand the result to the code under test. Herdr has no built-in browser: ask the owner for the manual step."
license: MIT
---

# Harness browser

Use this skill when a task needs a page that only works with the owner's browser session. Typical cases: an OAuth consent page behind an identity provider, a device enrollment, a callback capture. The goal is to automate every step that needs no manual owner action.

## Detect the harness

Check the environment once at the start of the task.

| Signal | Harness | Browser |
| --- | --- | --- |
| `ORCA_TERMINAL_HANDLE` is set | Orca | Built-in browser through the `orca` CLI |
| `HERDR_ENV=1` | Herdr | None. Ask the owner for each manual step |
| Neither | Unknown | Ask the owner |

Inside Orca, load the version-matched guide before any browser command:

```text
orca skills get orca-cli --reference browser
```

The `orca` executable is on PATH. The fallback path on macOS is `/Applications/Orca.app/Contents/Resources/bin/orca`.

## Orca recipe

1. Open the page: `orca tab create --url <url> --json`. Keep the `browserPageId` and pass it as `--page <id>` to every later command.
2. Wait for the page: `orca wait --page <id> --load networkidle`.
3. Inspect the page: `orca snapshot --page <id> --json`. The accessibility tree gives element refs such as `e3`.
4. Fill and click: `orca fill --page <id> --element <ref> --value "<text>"` and `orca click --page <id> --element <ref>`.
5. Submit a form that a click does not post: run JavaScript on the page.

```text
orca eval --page <id> --expression "(function(){var f=document.querySelector('form');f.requestSubmit(f.querySelector('button[value=approve]'));return 'ok';})()"
```

6. Capture the redirect: `orca tab list --json` shows the final URL of the tab. A `localhost` callback works from the Orca browser. A private scheme such as `obsidian://` does not open an app from the Orca browser. Copy the full callback URL from the tab list and give it to the code under test, or to a local listener, as the exchange input.
7. Close the tab when the task no longer needs it.

A page that lands on an identity provider login instead of the expected page means the owner session expired. Stop and ask the owner to log in inside that tab. Then continue from step 2.

## Herdr and other harnesses

Herdr has no built-in browser. Ask the owner once per page with the exact URL and the exact value you need back, for example the full callback URL. Wait for the answer. Do not open a system browser from the agent.

## Rules

- Never print cookie or token values. `orca cookie get` returns them. Do not call it unless the task needs a cookie name, and then read only the names.
- Never write a token, a cookie or a callback with a code into a tracked file or a task note. Record only client ids, times and results.
- Use the device or client names the task gives. Do not reuse a name from an earlier task.
- One tab per flow. Read the tab list before you open another one.
- The owner does the identity login and anything inside a third-party account. Everything after the login is automated.

## Report

State in the task notes which steps ran through the browser, the client ids created, the times, and the results. State the owner-manual steps that remain.
