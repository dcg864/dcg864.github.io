---
title: 'Finding Unique Ways to Grow Skills'
date: '2022-09-28T19:48:00-04:00'
layout: post
author: overcast
permalink: /2022/09/finding-unique-ways-to-grow-skills/
image: /assets/uploads/2022/09/steamstoragemgr.png
tags: [critical thinking, growth, hacker mindset, scripting, Member Article]
---

Most skills take time to hone and master. I’m not only referring to technical but also soft skills related to being inquisitive. Admittedly, I often focus most of my attention on strengthening knowledge and skill of a technical topic. Most technology today includes the means for gaining easy answers even while learning a new technology; a GUI screen that summarizes the configuration of multiple complex subcomponents, a GUI for building out a new virtual machine, or, to stop picking on GUIs, leveraging a exploit, Vagrant or Terraform script from GitHub. These aids are helpful and make us more efficient in our tasks but there is a depth of understanding that is often obscured by someone else’s work. I’m not saying to avoid using these options, by the way, simply pointing out that GUI tools or structured training plans often implicitly include defaults or assumptions that you may not particularly want.

Easy numbs our creativity in asking good questions.

Easy does help learn a foundational body of knowledge of a topic. There are plenty of ways to ask questions at this level of training. But there is value in probing further.

In this article I will explore being inquisitive about weird random things in life. Let's step outside the official bounds of what’s presented to us and ask good questions, even in areas outside the realms of cyber security.

## **Scenario 1: Why is water dripping behind the toilet?**

**Should I just call a plumber?**

Ha! I’ve played Mario games since I was a wee child, I’ve got this. Plus, I cannot stand the word "just".

**From where is the leak occurring?**

Appears to be the left side of the tank, the bolt or maybe the gasket. I had to look up an image on how this is assembled. The terms are tank bolt and spud gasket. Cool.

**What can help stop any damage right now?**

Towel or bowl to catch the water.

**How can I remediate the leak?**

Tightening the tank bolt didn’t fully stop the leak but did slow it, looks really corroded and there’s a goldfish in the tank(?).

**How do you replace these tank bolts?**

Thanks YouTube.

**Can I script this solution to be more efficient next time?**

Infrastructure as code is lagging in this area, so no.

## **Scenario 2 (*and the real reason for this article*): What is the total size of my Steam library in GB?**

As a legacy gamer I have not always done a great job of keeping track of where I install everything. Recently I’ve been thinking about moving all my Steam games to an SSD as I know most are spread out over one or more on a 7200 RPM HHDs.

The easy option is to use the Steam client GUI. Its library will show each game’s respective size in GB with the “Sort by Size on Disk” option along with the drive letter. This is an 80% solution. I still need to manually add all those values per drive.

Another option exists, maybe a 90% option, under the open the Settings menu, then under Downloads &gt; Content Libraries (“Steam library folders” button) there is a Storage Manager. Ooo, pretty.

Neither of these options show the real rolling total. Let’s use this as an opportunity to ask interesting questions.

**What do I know about Steam?**

Steam is a locally installed software package that integrates with Steam cloud services.

**How does the client know about the size of each installed game?**

It seems unlikely this info is synced from the cloud. Maybe the client calculates this on the fly or stores a cache of sizes in a file on a local disk (*spoiler*).

**What is the working directory for the Steam client?**

The Steam startup launcher shortcut links to steam.exe executable is located in "C:\\Program Files (x86)\\Steam\\Steam.exe".

**Anything interesting in this directory?**

After nosing around that directory I came across the `steamapps `subdirectory and the `libraryfolders.vdf` file. This file has a JSON-like format structure with big picture views of each library location as well as an appId and size in bytes for each installed product. There are a few possible solution elements here: `totalsize `and `update_clean_types_tally`. Neither matches what I am manually seeing on disk.

This file is refreshed after each Steam client restart. There are no game product names listed in this file but there is an `appId `which seems to relate to a game product and numbers which may be the size. Manually checking the numbers shows that they are the size in bytes for each installed game.

There’s a path for each library storage location.

**Try harder. What other info can you find?**

Each library storage location has a respective `steamapps `subdirectory. Included in `steamapps `there are files named “`appmanifest_<appId>.acf`” with a similar JSON-like format to `libraryfolders.vdf`. Each appmanifest contain details about each installed app/game/soundtrack. Most importantly the manifest contains a value for `SizeOnDisk`. There’s also a mapping of `appId` in the file name to a “`name`” element containing the actual product name. This is a lot of good information.

**Can I script this to be more efficient next time and achieve the ultimate Steam library size goal?**

Yes. I should be able to \[re-\]create the Steam Storage Manager in a language of my choice with the details I’m interested in. Let’s talk through the pseudo code for how this can be done. For each library listed in libraryfolders.vdf get the library path, append steamapps to it and get a list and content of all appmanifest files to calculate the game size in GB.

Here’s an initial, straight forward proof of concept script in PowerShell to size up your Steam libraries. Beyond that, we practiced skills of critical thinking, enumeration, and scripting. In the spirit of the hacker ethos I'm dropping the script here for you to make optimize efficiency, rewrite in another language, or find some other discovery from this knowledge.

## PoC: PowerShell

```
#-----------------------------------------------------------------------------
#  Steam Game Library Size on Disk
#
#  Parse the Steam VDF file and calculate the size for each game.
#  Steam has a main file listing all library locations.  By default this is: 
#      "C:\Program Files (x86)\Steam\steamapps\libraryfolders.vdf"
#      The VDF only lists app IDs not product names.
#  Each library location has a steamapps subdirectory with an *.acf file 
#  per game/product currently installed.
#-----------------------------------------------------------------------------

$library_file = "C:\Program Files (x86)\Steam\steamapps\libraryfolders.vdf"  # Default library file.
$vdf_file_contents = Get-Content $library_file
$total_size_gb = 0       # running library size
$list_of_sizes = @{}     # running list of game sizes
$list_of_libraries = @() # library list
$app_manifests = @()     # game manifest list

"//                Steam Game Sizer                 //"
"// Calculated since the last Steam client restart. //"

ForEach ( $line in $vdf_file_contents ) {
    # Collect list of libraries.
    if ( $line -match '\s+\"path\"\s+\"(.*)\"' ) {
        # Get a list of libraries.
        $lib_path = ($Matches.1).replace("\\", "\")
        $list_of_libraries += $lib_path
        # Manifest files contain the mapping between AppId and Game Name.
        $app_manifests += Get-ChildItem "$lib_path\steamapps" -Filter *.acf | ForEach-Object { $_.FullName }
    }
    # Filter only AppID and Game size in GB.
    if ( $line -match '^\s+\"(\d+)\"\s+\"(\d+)\"$' ) {
        # Convert size to GB
        $size_gb = [math]::round($Matches.2 / 1024 / 1024 / 1024, 2)
        # Accumulate a rolling size in GB.
        $total_size_gb = [math]::round($total_size_gb + $size_gb, 2)
        
        # Look up the game name.
        $game_id = $Matches.1
        try {
            $game_name = (Get-Content -Path ($app_manifests | Where-Object { $_ -match $game_id }) | Where-Object { $_ -match "name" })
            # Sanitize cruft.
            $game_name = $game_name -replace '"', '' # Remove double-quotes
            $game_name = $game_name -replace '^\s+name\s+', '' # remove leading stuff
            $game_name = $game_name -replace "®", '' # Remove copyright superscript
            $game_name = $game_name -replace '™', '' # Remove trademark superscript
            $game_name = "$game_name ($game_id)"
        }
        catch {
            # Getting manifest content failed, most likely due to game uninstall.
            "Cannot find manifest for game ID {0}. Steam may need a restart to remove from VDF file." -f $game_id
            $game_name = "Unknown, recently uninstalled ($game_id)"
        }
        # Store the game and size.
        $list_of_sizes[$game_name] = $size_gb
    }
}
# Display the results, sorting on the size.
$list_of_sizes.GetEnumerator() | Sort-Object -Property Value -Descending | Format-Table -AutoSize
"============================================"
"Total Library Size (GB): {0}" -f $total_size_gb
```