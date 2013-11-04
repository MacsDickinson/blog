---
layout: post
category: Chocolatey
title: Chocolatey - Why have cotton when you can have silk?
---

Setting up a new PC is always a bit of a bore, you end up spending days downloading and running installers. This is what I was doing when a friend tweeted me this:

> [@mattscode](https://twitter.com/mattscode) [@MacsDickinson](https://twitter.com/MacsDickinson) Chocolatey is amazing. Can get VS and everything for you [http://t.co/V5cBlbtswr](http://t.co/V5cBlbtswr)
> 
> — Jason Mitchell (@jmitch18) [August 7, 2013](https://twitter.com/jmitch18/statuses/365116400865525761)

And he was right. **[Chocolatey][1] is amazing!** It uses NuGet to download and install windows applications - all you have to do is run through the odd installation wizard.

<!--excerpt-->

Here's my powershell script with a brief description for each program.

	# Installing Chocolatey

	iex ((new-object net.webclient).DownloadString("http://bit.ly/psChocInstall"))

	# Web Browsers

	cinst GoogleChrome
	cinst Firefox

	# Utilities

	cinst gimp         # Open source paint and photo manipulation program
	cinst vlc          # The best media player
	cinst 7zip         # Package manager
	cinst paint.net    # MSPaint with more oomph
	cinst filezilla    # FTP Client
	cinst FoxitReader  # PDF Reader
	cinst dropbox      # Cloud storage
	cinst spotify      # Music Streaming
	cinst ChocolateyPackageUpdater # Package auto updater
	cinst steam        # PC gaming client
	cinst tweetdeck    # Twitter client

	# Productivity Tools

	cinst autohotkey_l # Key mapper for customised keyboard short cuts
	cinst greenshot    # Screen shot utility
	cinst ditto        # Clipboard history manager

	# Dev Tools

	cinst VisualStudio2012Professional # Visual Studio
	cinst resharper    # Visual Studio productivity tools
	cinst dotpeek      # .NET decompiler
	cinst fiddler      # Web traffic profiler and debugger
	cinst linqpad4     # C#/F# IDE
	cinst beyondcompare # IMHO the best file comparer out there
	cinst githubforwindows # Github GUI
	cinst poshgit      # Github powershell client
	cinst expresso     # Regular expression toolkit

   [1]: http://chocolatey.org/ (Chocolatey)