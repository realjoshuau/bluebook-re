# Lockdown

This information is current as of Bluebook [`0.9.724`](clients/0.9.724/README.md)

## Windows

The CollegeBoard developers know that they won't be able to catch every instance of cheating across their many, many, administrations of the exam. 

However, they do have a few methods to make cheating more difficult and they don't want to make it obvious to the user that they know that the user is cheating. 

This ties into the "Integrity Worker" (WIP) and the Lockdown Mode (this article) of Bluebook.

Before we start, this analysis was completed using [Bluebook 0.9.724](clients/0.9.724/README.md) for Windows.

- application release: `exam-player-electron@0.9.724`;
- main-process distribution: `prod.electron-main.2026-08-13-12-00`;
- custom native add-on: `6c92b92643e96ce390280fba5d478a66.node`, internally named `win_app_tools.node`.

to be finished - there's a lot here!

## macOS

on macOS, Bluebook uses the [Automatic Assessment Configuration](https://developer.apple.com/documentation/automaticassessmentconfiguration) APIs. Currently unknown as if the app will use other macOS APIs (presumably no private APIs, since this is a App Store release.)

NB: This is also how "Bluebook Bypass" apps work on macOS. They use the internal `SkyLight` APIs [to overlay windows](https://github.com/trycua/cua/blob/main/blog/inside-macos-window-internals.md) that (presumably) read the screen. This is risky since Bluebook does, at times, take photos of the screen, the DOM structure, **and your webcam**

More information is needed. Please open a PR if you have any information!