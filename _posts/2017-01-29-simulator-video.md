---
layout: post
date: 2017-01-29 12:00:00 -0800
title: Recording iOS Simulator Video
---

At the latest meeting of the Redmond Xcoders, there was a talk by [Lori Hill](https://mobile.twitter.com/lorilhill7) about creating App Preview videos. During the Q&A period after her talk, questions were raised about recording such a video from the iOS Simulator rather than an actual device. The first few comments were about doing an actual screen capture of the simulator, and recording the simulator _directly_ was an option it seemed most people didn't know about. I've attempted to find out what I can about the subject and put it somewhere that might be helpful to Google users in the future.

**Disclaimer**: It is unclear whether these videos can be used for App Preview. Some differences between device recordings and simulator recordings are discussed below. Apple's [App Preview documentation](https://developer.apple.com/support/app-previews/) has no real stance on the matter; while they discuss how to record with a physical device, they don't explicitly state that a device _must_ be used. If you attempt this and find out one way or another, please e-mail me and I will update this post.

Video from the Simulator can be recorded using the `simctl` Xcode tool, like so:

```
xcrun simctl io <device> recordVideo <path>
```

Recording begins immediately, and continues until you kill the `simctl` process by pressing ^C.

The easiest way to get the `device` parameter is to launch the simulator manually through Xcode, then use the value  `booted`:

```
xcrun simctl io booted recordVideo <path>
```

Video is recorded at the full resolution of the simulator in question, regardless of whether you have scaled the window to allow it to fit on your screen. If you have an iPhone SE simulator running, scaled to 50%, the recorded video's resolution will still be 640×1136, not 320×568. One place where this might cause some surprising effects is with Plus-sized devices: simulator recordings are at "true" 3× resolution: 1242×2208, _not_ the 1080×1920 you'll get from a physical device.[^tyten] 

Recording the simulator this way includes _only_ what is seen on the iOS device. It does not include the mouse cursor, the disk shapes seen when using a two-finger gesture, etc. If you want to have the keyboard display, etc., make sure you enable the correct settings in the simulator before recording.

There are a few other differences between simulator videos and device videos. When recording a device with QuickTime Player, the carrier text is removed entirely. The simulator displays the text "Carrier". Likewise, a device is permanently fixed to 9:41 AM while recording, whereas the simulator uses your Mac's actual time.

Obviously, you cannot perform anything in your recording you wouldn't normally be able to perform in the simulator. This means no camera, no Bluetooth, etc.

Hopefully this gives a good overview of using the simulator to make recordings of your apps. It's a quick way of demonstrating behavior to your users, sharing updates with stakeholders, or getting bug reports from your QA team. Apple's official documentation can be found [here](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/iOS_Simulator_Guide/InteractingwiththeiOSSimulator/InteractingwiththeiOSSimulator.html#//apple_ref/doc/uid/TP40012848-CH3-SW4), with more information available by running `xcrun simctl io`.

[^tyten]: Hat tip to [Tyten Teegarden](https://twitter.com/Tyten) for verifying that a device gives you the lower-resolution video.