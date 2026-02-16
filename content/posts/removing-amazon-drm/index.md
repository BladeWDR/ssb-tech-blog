+++
title = 'Using Calibre and the DeDRM plugin to strip DRM from Amazon eBooks'
date = 2026-02-15T20:58:33-05:00
draft = false
+++

# Using Calibre and the DeDRM plugin to remove DRM from Amazon eBooks

## Overview

I'm not going to spend too much time going over this as there are plenty of screenshots in other guides, two of which I've linked at the bottom of this page.

However I felt their instructions were a bit all over the place and hard to follow, so here's my own notes.

{{% callout note %}}
### **Disclaimer:**  
This should only be used with Amazon eBooks that you legally own.
{{% /callout %}}

## Steps

1. Install Kindle for PC 2.8.0. As of 2026-02-15 this is the version available from Amazon's own website.
    - ***As soon as you get it installed, open the program's options menu and disable updates.***
    - You'll need to sign into your Amazon account and download whatever books you'd like to strip DRM from.
2. Go to this repository: [Satsuoni\/DeDRM repo](https://github.com/Satsuoni/DeDRM_tools/)
    - Under the releases tab, you _need_ to download one of the pre-releases, the current stable version doesn't work to strip the more recent KFX encryption.
    - Download the zip file and extract it. There will be several more zip files within it, do _not_ extract those as that is the format Calibre expects for plugins.
3. Copy the `KFXKeyExtractor28.exe` file into `%localappdata%\Amazon\Kindle\application`.
    - You'll need to run a command similar to this one to grab the keys from your downloaded books:
      ```cmd
        C:\Users\ssbtech\AppData\Local\Amazon\Kindle\application\KFXKeyExtractor28.exe "C:\Users\ssbtech\Documents\My Kindle Content" kindlekey.txt kindle_account.k4i
      ```

      This application is what gathers the keys needed by the Calibre DeDRM plugin to strip the DRM.

      <span style=color:red>***You need to run this every time you download new books! Every book you download has a unique key!***</span>

      I created this convenience batch script that I run every time I download a new book to strip the DRM from.

      ```cmd
      @echo
      cd C:\users\ssbtech
      C:\Users\ssbtech\AppData\Local\Amazon\Kindle\application\KFXKeyExtractor28.exe "C:\Users\ssbtech\Documents\My Kindle Content" kindlekey.txt kindle_account.k4i
      ```
4. Next we need to configure the DeDRM plugin in Calibre. Install [Calibre for PC](https://calibre-ebook.com/) and navigate through the welcome wizard.
    - Once it drops you on the main screen, go into Preferences > Plugins.
    - Click on "Add Plugin From File". You'll want to add the `DeDRM_plugin.zip` file that was in the zip you extracted earlier.
    - Click on OK to go back to the application, don't restart Calibre yet. Click on "Get Plugins". Search for a plugin called `KFX Input` and install that, too.
    - Now you can let Calibre restart to make sure that your new plugins are loaded.
5. Open the plugins menu once more and find the DeDRM plugin that you installed earlier.
    - Click on "Customize Plugin".
    - Click on "Kindle for PC/MAC eBooks" and then "Import existing keyfiles". You'll want to import the `.k4i` file that we created earlier.
    - Next, click on "Set Keyfile" and set it to the `kindlekey.txt` file that was created along with the `.k4i`.
6. At this point you're ready to import Kindle eBooks and strip them of DRM.
7. Click on "Add Books". Kindle for PC stores the eBooks under your Documents folder (`%USERPROFILE%\Documents\My Kindle Content`).
    - They're stored in folders that look like strings of random characters, but inside of each of them is a `.azw` file, that's your eBook. Import that into Calibre.
    - If you did it right, it will show as type `KFX`, which you can then convert to EPUB.

## Troubleshooting

### Files showing KFX-ZIP extension instead of KFX, can't strip DRM.

If the files you're importing show the type `KFX-ZIP` instead of `KFX` that means that they're still encrypted and can't be run through the DRM stripping process.

<span style=color:red>_This is usually because you didn't re-run KFXKeyExtractor28.exe after downloading a new book._</span>

Remove the book from Calibre, then re-run the batch script we created earlier so it grabs the new keys for the newly downloaded books.

If that doesn't work, I'd probably start by going back to the GitHub repo and seeing if there's a new release or a new method. Amazon is constantly changing their DRM to make it more difficult to remove.

### Converted files are huge

This is because of the images embedded into the EPUB file.

I typically recommend setting your "device" in Calibre to something like an actual Kindle e-Reader.

Calibre will automatically scale images based on that option. If you chose the "Generic" type it won't scale the images down at all.

Just make sure when you change the device type to an Amazon eReader that it didn't override your default output type.

Go under Preferences > Behavior and set the "Preferred output format" to EPUB.

## Links to the guides I used

1. [Techy-notes guide](https://techy-notes.com/remove-drm-from-kindle-ebooks/)
2. [Thecodeshewrites guide](https://thecodeshewrites.com/2025/02/25/how-to-remove-drm-from-amazon-kindle-adobe-ade-ebooks-for-free/)
