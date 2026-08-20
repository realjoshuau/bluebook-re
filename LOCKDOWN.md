# Lockdown

This information is current as of Bluebook [`0.9.724`](clients/0.9.724/README.md)



## macOS

on macOS, Bluebook uses the [Automatic Assessment Configuration](https://developer.apple.com/documentation/automaticassessmentconfiguration) APIs. Currently unknown as if the app will use other macOS APIs (presumably no private APIs, since this is a App Store release.)

NB: This is also how "Bluebook Bypass" apps work on macOS. They use the internal `SkyLight` APIs [to overlay windows](https://github.com/trycua/cua/blob/main/blog/inside-macos-window-internals.md) that (presumably) read the screen. This is risky since Bluebook does, at times, take photos of the screen, the DOM structure, **and your webcam**

More information is needed. Please open a PR if you have any information!