# Webcam Usage

As of Bluebook 0.9.724, the Windows version of Bluebook has references to using the webcam to take photos of the user during the exam. 

Questions:
- [ ] what is the frequency of webcam usage? (every 5 minutes, every 10 minutes, etc.)
- [ ] Is webcam usage logged? 
- [ ] Where are the photos stored or sent to?

According to the [privacy policy](https://web.archive.org/web/20260801063341/https://bluebook.collegeboard.org/students/privacy-policy-use-bluebook), "we may also use a biometric-enabled system to collect biometric data, including facial images (using the front-facing camera on your testing device)....throughout the international test administration"

Sources:
[`main.index.js`](https://github.com/realjoshuau/bluebook-clients/blob/main/0.9.724/deobf/main.index.js), lines 409636 to 409675 reference `useWebcamCapture`

