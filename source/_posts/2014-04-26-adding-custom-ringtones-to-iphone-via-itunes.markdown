---
layout: post
title: "Adding custom ringtones to iPhone via iTunes"
date: 2014-04-26 21:09:18 +0530
comments: true
categories:
---

I recently tried adding custom ringtones to my iPhone via iTunes and what a pain it was! I had to browse through multiple apple support pages and blogs to finally figure out how to accomplish this trivial task. God I love and miss those Nexus S days...

This post is to share the steps involved so that it saves the frustration for others.

So my gig:

* iTunes 11.1.5 running on Mavericks
* iPhone 5

<!-- more -->

## Steps to convert existing song in album to ringtone

* The first step is to enable Tones section as part of the Library drop-down. Go to preferences (<kbd>CMD</kbd>+<kbd>,</kbd>) and check on Show Tones. For some reason, this section is not visible by default.

{% img left /images/2014-04-26/enable_tones.png %}

* Now select the song that you would like to use as ringtone, right click and select `Get Info`. Navigate to `Options` tab. iTunes has a restriction that the ringtone synced with device cannot be longer than 30 seconds. So, using start time and end time, you can select the 30 second period of the composition that should be ripped as ringtone.

{% img left /images/2014-04-26/start_stop_time.png %}

* After saving the options, right click on the song again and click on Create AAC version.

{% img left /images/2014-04-26/create_aac.png %}

* iTunes will create another version of the song next to existing one with same name but with duration to the right as 0:30. Again, right click and select `Show in Finder`. The new file would have the extension `.m4a`. Rename the extension to `.m4r`. Finder would warn you about it and insist by selecting `Use .m4r`.

* Now, this is where you trick iTunes into believing that the new file created is a ringtone. Go back to iTunes, select the new file created and delete the file by hitting <kbd>DEL</kbd>. When warned, remember to select `Keep File` instead of Move to Trash.

* Go back to the finder window and double click on the .m4r file. This will open up in iTunes in the Tones section automatically and Viola!!!

Now, sync the library with the device and you should be able to select the ringtone from Settings -> Sounds -> Ringtone

For adding new music files (not in iTunes yet) as ringtone, just change the file extension to .m4r and click on it automatically adding it to iTunes. Then trim the length to 30 seconds using start time and end time and you are ready to sync it with the device.

Happy Missed Calls!!!
