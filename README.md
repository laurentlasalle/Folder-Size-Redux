# Folder Size Redux

Folder Size Redux is a small utility that shows the size of every item in File Explorer, folders included. It is designed to help you manage your disk space without having to open a folder's properties every time you want to know its size.

The information is displayed in a secondary window, alongside File Explorer. Folder sizes are calculated in the background when you open a drive or a folder, so the information is complete and up to date as you browse its contents.

This project is actually a fork of [Folder Size for Windows](https://sourceforge.net/projects/foldersize/) 2.6 by Brian Oraas (Brio), last updated in 2013. It was originally designed for Windows 2000 and XP to add a column to File Explorer showing folder sizes. When Windows Vista made it impossible to customize File Explorer that way, Brio and his contributors reworked the code into a version with a companion window.

Folder Size 2.6 *runs* on Windows 10 and 11, but it's not fully compatible. For example, the size of ZIP files (or other archive types), which the OS now treats as folders, isn't shown (even though File Explorer shows it). Using tabs in File Explorer also causes problems, since the software was never designed to handle them (tabs simply didn't exist back then).

Folder Size Redux is therefore a spiritual successor to that project, adapted for Windows 10 and 11.

## Where it started

The original code was recovered from SourceForge's CVS, with its full history since 2005, and published in `laurentlasalle/Folder-Size-Redux` under the GPLv2. Each CVS change became a git commit with its original author, date and message. Original contributors include Brio, Kirill Müller and eisber.

I named this repository Folder Size Redux, since the idea was to bring back and revisit Folder Size so it is fully compatible with modern operating systems. Features have been added, but the original project's goal remains: lightweight and fast.

Let's cut to the chase: this project was vibe-coded with Claude (using the Opus 5.5 model). I think I steered it well enough that a human can understand its code, but if that's not the case, please don't throw rocks at me. I made this project first and foremost for myself, and I'm happy with where it stands.

At the time of writing, version 0.1 has only been tested by me, on Windows 11. I've dropped support for earlier versions of Windows, which already work with Folder Size 2.6 anyway.

It is also only built for x64. A Windows on ARM version is possible if there's demand for it.

## Build and installation

The project was converted to Visual Studio 2022, and its releases are built automatically by [GitHub Actions](https://github.com/laurentlasalle/Folder-Size-Redux/actions), which produces the MSI (the installer).

To install Folder Size Redux, simply visit the [Releases](https://github.com/laurentlasalle/Folder-Size-Redux/releases) page for the latest version, and download the `.msi` file in the **Assets** section. It requires the 64-bit edition of Windows 10 or 11. If Folder Size for Windows 2.6 is installed, the installer replaces it.

The installer isn't signed with a code signing certificate, so Windows may show "Windows protected your PC" when you open it. Click **More info**, then **Run anyway**.

By default, Folder Size Redux starts with Windows and shows a companion window whenever File Explorer is open. Either of these can be turned off from its menu (right-click the icon in the system tray). A settings panel can also be opened from that menu.

Controlling the service (**Start**, **Stop**, **Pause**, **Resume** on the **Service** tab) and changing the **Drive types to scan** need administrator rights. The settings panel runs with the rights of the user, so these actions show the shield and Windows asks for approval (UAC) when they are used. The approval prompt names Windows host process (Rundll32), which runs the action from `FolderSize.cpl`. The other settings don't need administrator rights.

For those who prefer not to use an installer, a portable version is available for download.

### Portable use

The portable version runs without installing anything: download the `-portable.zip` file from the **Assets** section of the [Releases](https://github.com/laurentlasalle/Folder-Size-Redux/releases) page, extract it to a folder of your choice, and run `FolderSize.exe`. As the Folder Size service isn't installed, it starts `FolderSizeSvc.exe` as an ordinary background process (no administrator rights needed) and ends it when it exits. In this mode:
- folder sizes are cached only while `FolderSize.exe` runs, so the sizes of large folders take a moment to appear after it starts;
- all drive types are scanned (choosing drive types requires the installed service);
- **Settings...** opens the control panel from its file; it doesn't appear in **Control Panel**.

## Windows 10 and 11 integration

The main changes to modernize the experience:

- Folder Size Redux offers a dark mode for users who have set Windows that way.
- It respects per-monitor display scaling, so the companion window doesn't look pixelated on a 4K or other high-density (high-DPI) screen.
- It supports File Explorer tabs, so the companion window always shows the information for the active tab (a single companion window per File Explorer window).
- The window hides when File Explorer is on a virtual location (such as **Home**, **Gallery** or **Network**), but shows the space used on each drive when you're on **This PC**. It's the same information File Explorer already calculates. Since Folder Size Redux can sort items by size, you can easily see all your drives, from heaviest to lightest.
- It can tell ZIP files (and other archives) apart from real folders, so the size of each is calculated correctly.
- Links (**Symlink**, **Junction** or **Mount point**) are identified by the arrow badge (like shortcuts), and their type is shown in place of the size. When you open one, sizes are calculated through its real path, and the Folder Size Redux title bar shows the actual location.

## Companion window

The companion window is the window named Folder Size Redux that automatically accompanies File Explorer. It has two columns (**Name** and **Size**) and lists items in classic alphabetical order (like Windows: folders first, then files). Two other sort orders are available:

- Clicking **Name** sorts the contents in mixed alphabetical order (no distinction between files and folders). Clicking **Name** again brings back the initial view.
- You can also click **Size** to see items from heaviest to lightest. You can't do the reverse. Why would you want to do the reverse?
- The default sort order can be changed in the settings.

The companion window sizes itself automatically to fit what it shows, up to a maximum width of 600 pixels (at 100% scaling). This width can be changed with the **Maximum width** setting. Resizing the window with the mouse only applies to the current window (it goes back to your settings when you move elsewhere).

When long file names are cut off, their full name appears in a tooltip when you hover over them with the mouse.

Folder Size Redux lets you pick from 13 languages in the settings, for those who don't want to use it in the language of their Windows setup.

## Performance

Compared to its predecessor, Folder Size Redux improves a few things:

- Item icons are only loaded for what is visible in the companion window, which speeds up the display of large folders (Downloads, for instance). Other icons are loaded as you scroll, only if needed.
- Processed items are added in batches, so large folders don't slow down the display, as the scanner is often slower than the window in that case. Instead of receiving items one by one, then redrawing the window and re-sorting its contents each time, items are handed over every 100 ms, which cuts down on redrawing and re-sorting.
- Information about items is now gathered within Folder Size Redux's own process. To know the name and type of an item to display (subfolder, file, shortcut, archive), the scanner has to ask the folder being browsed in File Explorer. Previously, it sent that request to File Explorer, which meant a round trip between the two programs for every item. Now, the Folder Size Redux process opens the folder itself and fetches the list of items in batches of 64, which speeds up the display.
- The list updates as soon as Windows reports a change in the folder being browsed in File Explorer. To avoid endlessly re-reading a very busy folder (such as %TEMP% or AppData), Folder Size Redux waits, after each re-read, at least 10 times as long as that re-read took before the next one (between 300 ms and 5 s). A small folder therefore updates almost instantly, while a folder with thousands of items that keeps changing is re-read at most every 5 seconds.
- Folder Size Redux automatically reconnects after File Explorer restarts.

Together, these changes cut the processing time of a 586-item folder from 1.2 to 0.1 seconds.

## Troubleshooting

If the size window seems slow or busy, a diagnostic log shows how long reading each folder takes and where the time goes. It has no setting: it is written only while its file exists. To start it, run in PowerShell:

```powershell
New-Item -Force "$env:LOCALAPPDATA\Folder Size Redux\FolderSize.log"
```

Then browse the folders in question with File Explorer. Each reading of a folder adds a few lines to the file: the number of items, the time until the list is complete, the CPU time used, and the details of the scanner and of the window. The log's own folder and the folder above it (`%LOCALAPPDATA%`) are left out, as writing the log changes them. To stop, delete the file:

```powershell
Remove-Item "$env:LOCALAPPDATA\Folder Size Redux\FolderSize.log"
```

## Translations

The texts of the program are translated with gettext files, like many other projects: `FolderSize/Lang/folder-size-redux.pot` holds the English texts, and each language has its own `.po` file next to it (`fr.po`, `de.po`, and so on). Edit them with [Poedit](https://poedit.net) or any `.po` editor. To add a language, copy the `.pot` file to `<language>.po` and add the language to the list at the top of `build_lang.py`.

Windows programs can't read `.po` files directly, so `FolderSize/Lang/build_lang.py` converts them into Windows string tables (`FolderSize/Common/LangStrings.*`), compiled into the programs. Run it after changing a `.po` or `.pot` file and commit the generated files along with it:

```powershell
python FolderSize/Lang/build_lang.py
```

The build on GitHub checks that the generated files are up to date. Texts that are missing or marked fuzzy in a `.po` file are shown in English.

By default the program uses the language of Windows when it is available, and English otherwise. The **Language** setting in the **General** tab of the settings chooses another language.

## License

Folder Size Redux is free software, released under the GNU General Public License, version 2. See `COPYING`. Folder Size for Windows is copyright Brio and his contributors; the changes made in Folder Size Redux are copyright Laurent LaSalle and released under the same license.
