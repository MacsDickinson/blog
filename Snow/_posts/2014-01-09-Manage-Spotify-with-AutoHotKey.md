---
layout: post
category: Productivity
title: Manage Spotify with AutoHotKey
---

I've got a fancy keyboard at home that's so futuristic it has play and pause buttons on it. Crazy I know! My computer at work isn't quite as spectacular as this so I have to resort to other means if I don't want to be constantly opening [Spotify][0]. A couple of years ago a [colleague][1] showed me [AutoHotKey][2] - a scriptable desktop automation application -  for this very reason. I've since lost that script (and that keyboard) so I'm writing this post to help me next time I lose it.

Setting up your script
----------------------

Downloading and installing AutoHotKey is as straightforward as it should be. Once you've got it running you're going to want to create your script. If like me you want the script to run on startup so go ahead and create it in your startup folder. Call it something like startup.ahk. My file looks a little something like this:

      ^!Space::Media_Play_Pause	; Ctrl+Alt+Space => Play/Pause music
      ^!Left::Media_Prev				; Ctrl+Alt+← => Previous track
      ^!Right::Media_Next				; Ctrl+Alt+→ => Next Track
      ^!Up::Volume_Up					; Ctrl+Alt+↑ => Volume up
      ^!Down::Volume_Down			; Ctrl+Alt+↓ => Volume down

You can do a lot more with AutoHotKey, check out their [quick start guide][3], [hotkey list][4], [key list][5] and [command list][6] to get an idea of what you can do.

   [0]: https://www.spotify.com
   [1]: https://twitter.com/mattscode
   [2]: http://www.autohotkey.com/
   [3]: http://www.autohotkey.com/docs/Tutorial.htm
   [4]: http://www.autohotkey.com/docs/Hotkeys.htm
   [5]: http://www.autohotkey.com/docs/KeyList.htm
   [6]: http://www.autohotkey.com/docs/commands.htm