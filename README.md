# Bluebook Reverse Engineering

Obviously, this is not to endorse or even encourage cheating.
This project is just to reverse how it works for curiosity's sake.
Absolutely work-in-progress. Focusing on the Windows version for now.

>[!NOTE]
> LLMs were used to assist in the reverse engineering process. Reverse Engineering was helped by Codex (GPT 5.6 "Sol" & 5.5) & Qwen 3.8 27B (with `pi`)

This is current as of [Bluebook 0.9.724](clients/0.9.724/README.md)

Bluebook is a web app, but depending on the platform uses different renderers & native code tricks to get native things done (such as locking down the system, webcam accesses, and anti-VM checks on Windows)


| Platform | Rendering Engine                  | Polyfills? | Trustability         |
|----------|-----------------------------------|------------|----------------------|
| macOS    | Internal WebView                  | Y          | Relatively OK        |
| Windows  | Electron                          | Y          | Much less than macOS |
| iPad     | Internal Webview                  | Y          | ?                    |
| ChromeOS | It's literally a browser based OS | Y          | ?                    |

(It always polyfills ES6 if necessary.)

In general, Bluebook does not automatically close exams for security and/or "integrity violations" unless in very small specific circumstances such as:
- Obvious Javascript modification
- Obvious RDP (through Windows Terminal Services, NOT through Parsec or other third-party Remote Desktop software)
- Obvious code injection or obvious "bad processes"
- Obvious VM detection (NOTE: VM detection is two-fold and _suspected_ VMs will be flagged)

Things that need to be done:

- [ ] Telemetry
- [ ] Test Data Packages
- [x] ~~Auth~~ (probably outdated?)
- [ ] Lockdown Mode
  - macOS uses the ["Automatic Assessment Configuration" API](https://developer.apple.com/documentation/automaticassessmentconfiguration), and in general, BlueBook treats macOS as a more secure platform to test on due to tampering being significantly more obvious on this platform.
  - Windows uses a custom binary Node module [`win_app_tools.node`](win_app_tools.md) to lock down the machine, including terminating Windows Explorer & hooking into most shortcuts.
- [X] "Integrity Worker"
- [ ] Telemetry Worker
- [ ] Repsonses Worker
- [ ] Webcam Usage
- [ ] On First Boot
- [ ] On startup
- [X] [Test Watermarking](WATERMARK.md)
- [ ] Test Uploads
  - I was not able to catch this via an HTTP sniffer. Maybe some other form of upload?

--- 

All questions, comments should be directed to my email:

realjoshuau at proton dot me
