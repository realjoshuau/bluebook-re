# Bluebook Reverse Engineering

Obviously, this is not to endorse or even encourage cheating. Bluebook is great - better than any other Lockdown-style browser I've used before and it makes testing less of a pain. This project is just to reverse how it works for curiosity's sake.
Absolutely work-in-progress. Focusing on the macOS version as of now.

Current Version: "Bluebook/905" (User Agent), "1.12.4" (macOS Version), "VSN-1.12.9 BT-2024-4-27 1:11" (Bluebook Internal), "ac8842221989527b7e75" (Internal Git Hash?)

Bluebook internally is a browser, but instead of wrapping Chromium (like Respondus does), it uses the internal WebView of macOS, and polyfills ES6 as necessary.

Things that need to be done:

- [ ] Telemetry
- [ ] Test Data Packages
- [x] Auth
- [ ] Lockdown Mode
  - Only thing we know so far: it can bypass Lock Screen (!?) and pressing Command + (.) during a Lockdown Session (or within Assessment Mode as Bluebook calls it internally) will window the screen, showing a dark background with a gray radial gradient extending from the center of the screen into the edges.
- [ ] "Integrity Worker"
- [ ] On First Boot
- [ ] On startup
- [ ] The "Watermark" (colored-bars) placed onto every test.
- [ ] Test Uploads
  - I was not able to catch this via an HTTP sniffer. Maybe some other form of upload?
