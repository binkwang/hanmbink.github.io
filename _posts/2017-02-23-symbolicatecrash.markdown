---
layout: post
title:  "How to use symbolicatecrash tool to analyze a crash report"
date:   2017-2-23 08:00:00
categories: [tech]
---

How to use symbolicatecrash tool to analyze a crash report

`Crash report`

Export crash report from your device:
connect the device to mac-Xcode-window-device-select the device-view device log
view crash report, crash report will just show memory addresses of objects and methods:
![](https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/symbolicatecrash/image-01.png)


`.dSYM file`

A dSYM file is a "debug symbols file". It is generated when the "Strip Debug Symbols" setting is enabled in the build settings of your project.
When this setting is enabled, symbol names of your objects are removed from the resulting compiled binary (one of the many countermeasures to try and prevent would be hackers/crackers from reverse engineering your code, amongst other optimisations for binary size, etc.).
dSYM files will likely change each time your app is compiled (probably every single time due to date stamping), and have nothing to do with the project settings.
They are useful for re-symbolicating your crash reports. With a stripped binary, you won't be able to read any crash reports without first re-symbolicating them. Without the dSYM the crash report will just show memory addresses of objects and methods. Xcode uses the dSYM to put the symbols back into the crash report and allow you to read it properly.
Ideally, your dSYM file shouldn't be tracked in your git repo. Like other binaries that change on building, it's not useful to keep them in source control. However, having that said, it is important that you keep the dSYM files for each distributed build (betas, press releases, app store distributions, etc.) somewhere safe so that you are able to symbolicate any crash reports you might get. Xcode does this automatically for you when you use the Archive option. The created archive contains your app and its dSYM and is stored within Xcode's derived data directory.

get your.dSYM file, maybe it stored on HockeyApp or other place

![](https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/symbolicatecrash/image-02.png)

`symbolicatecrash`

Where is symbolicatecrash
use command lind to find symbolicatecrash
$ find /Applications/Xcode.app -name symbolicatecrash -type f

Result:
/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash

Copy symbolicatecrash into a new fold:
cp /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKitBase.framework/Versions/A/Resources/symbolicatecrash /Users/username/Desktop/crash

`Generate the crash stack information`

Move .dSYM file and crash report file in to the same directory with symbolicatecrash
Make the same name of crash file and .dsym file (SHDR.crash / SHDR.dsym)

![](https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/symbolicatecrash/image-06.png)

Run command line to generate the crash stack information:
$ export DEVELOPER_DIR="/Applications/XCode.app/Contents/Developer" 
$ ./symbolicatecrash -v SHDR.crash  SHDR.dsym > ./SHDR-symbolicate.crash

Check the result SHDR-symbolicate.crash file:

![](https://raw.githubusercontent.com/hanmbink/hanmbink.github.io/master/pic/symbolicatecrash/image-08.png)

Reference: 
http://www.cocoachina.com/bbs/read.php?tid=180736
