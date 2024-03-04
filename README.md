# Don't use Appimages
Appimages are an insecure and user-unfriendly packaging system with very limited use cases.

This is a list of reasons not to use them.

### What are Appimages?

A packaging format similar to direct executables / portable Apps on Windows, or statically linked binaries.

This concept was quite anticipated, as it allows to run apps on all Linux Distributions.

That this is just as possible with statically linked binaries, that are possibly faster, is often ignored. [Firefox](https://www.mozilla.org/de/firefox/download), the [Jetbrains Toolbox](https://www.jetbrains.com/toolbox-app/download) and many more binaries work perfectly fine, but Appimages got a hype for whatever reason.

### Usability issues
While complex .tar archives (like firefox) may seem messy, they integrate many different things. An installer script takes care of placing a .desktop entry, you can have an updater script, a LICENSE, README and more. Those are all missing with Appimages.

Apps installed with the system package manager get their .desktop Entry in `/usr/share/applications`, installed Flatpaks get their entry linked to `~/.local/share/flatpak/exports/share/applications/`, user overrides and other installs can be put in `~/.local/share/applications/`.

Appimages have no desktop entry, so they have ([currently](https://gitlab.freedesktop.org/wayland/wayland-protocols/-/issues/52)) no icon on Wayland and they don't appear in your app list. Desktop entries are a standard, used by everytthing but Appimages.

Instead users follow strange habits like placing the files on their desktop, which is a highly discouraged "Windows workflow" ([symbolic image](https://www.firstclasstechnologies.co.uk/wp-content/uploads/busy-desktop.png)) and not even supported on many Desktop Environments, most notably [GNOME](https://linuxiac.b-cdn.net/wp-content/uploads/2023/09/gnome45-desktop.jpg).

This means that in the end users need to create their own .desktop entries, that will
- not get updated
- not get edited when an "update" changes the name of the appimage file (Appimages have no name!)
- and not deleted if you delete the .appimage file.

Example of such an entry

```bash
cat ~/.local/share/applications/someapp.desktop
[Desktop Entry]
Type=Application
Name=SomeApp
GenericName=Some description
Comment="$GenericName" #(yes you need both for different DE's)
Exec=/home/username/some-random-directory-probably-named-Apps/appname-version-architecture.desktop
Icon=/home/user/some-random-directory-probably-inside-the-app-dir/some-random-icon-also-not-an-svg-from-some-random-website-edited-with-gimp.jpg
```

Creating one manually it not fun.

### Updates
This is both a usability and a security issue. Traditional Linux apps, even if they are cross platform, don't have updater services, as package managers are way better at doing that. This means, packing as an Appimage either requires to implement an updating service, on a platform that doesn't need that, or to have no updates at all.

Instead users need to follow an RSS feed, get a mail, or **manually check for updates**, which is horrible UX. Then how do they update? 

### The lack of repositories
Appimages don't even have a central place where you can **find** them, not to mention download them. This is extremely insecure. Modern Application stores and every well made Linux repository uses cryptographic (mostly gpg) verification, which secures the authenticity of the software. You can be sure you downloaded the real package.

With Appimages, Linux users that always laughed about Windows malware through shady downloads, are catapulted back into that stone age too.

**Linux users should never download random software from the internet**. This is like rule No. 1.

There is no updating mechanism. On Android you may also update by downloading .apk files, but once installed, the .apk needs to be signed with the same key, otherwise updates are blocked. With Appimages... you just delete the old .appimage file, download the new one, change the name to remove the version and hope your .desktop entry didn't break.

This is how you get malware.

### They are not well maintained
There is [a well known "bug" on modern Ubuntu](https://www.omgubuntu.co.uk/2023/04/appimages-libfuse2-ubuntu-23-04), where Appimages lost their "works on every Linux Distro", because they are built for the outdated `libfuse2`, while Ubuntu now uses `libfuse3`. The fix is to install the outdated version of libfuse (!), and this is still not fixed. 

An application format, that is incompatible with the latest version of its core dependency, is broken.

### Lack of Sandboxing
Appimages use fuse, which [doesn't seem to work](https://github.com/AppImage/AppImageKit/issues/152) within a [bubblewrap](github.com/containers/bubblewrap) sandbox. ([Firejail is not recommended](https://madaidans-insecurities.github.io/linux.html#firejail)). Even [hacky attempts](https://github.com/igo95862/bubblejail/issues/8) to isolate them with [bubblejail](https://github.com/igo95862/bubblejail) needed to degrade the bubblewrap security model by running then in a root sandbox, and no fix was found in over 2 years.

**Appimages to this day run completely unconfined**, while (as mentioned above) their chaotic and unverified origin makes them **the perfect tool to distribute malware** on Linux.

(There already was malware [on the Snap store](https://www.bleepingcomputer.com/news/linux/malicious-package-found-on-the-ubuntu-snap-store/).)

Also, integration is important. Flatpaks can now be used and configured completely through the GUI. Appimages will not get that level of GUI tooling anytime soon. They have graphical tools, but those are incomplete.

### Random location
Even immutable distributions are far from the security of Android. A necessary step would be mounting the entire `/home` non-executable. This is no problem for system apps, or Flatpaks, but where do you put Appimages now?

There would need to be a standard directory to put such files in, which is normally the PATH. But this is also the easiest attack goal for malware, so PATH would be non-executable as well.

Without any system integration, Appimages can not be installed into a dedicated, secure directory (like flatpaks are).

### Duplicated libraries
Appimages bloat the system. They include all the libraries they need, and unlike system packages or Flatpaks, **they don't share a single libary**. If users would really install every Software as Appimages, they would waste insane amounts of storage space.

This also completely discourages efficient and up to date packaging, and the attached risk of outdates libraries is hidden away in that .appimage archive.

### Appimages are not needed
Flatpak solved many Linux desktop issues at once
- [runs on nearly all distributions](https://flatpak.org/setup)
- [sandboxes apps](https://docs.flatpak.org/en/latest/sandbox-permissions.html), using bubblewrap
- permission system with GUI integration ([KDE Settings](https://userbase.kde.org/Tutorials/Flatpak), [Flatseal](https://flathub.org/apps/com.github.tchx84.Flatseal))
- lots of [verified Applications](https://flathub.org/de/apps/collection/verified/1), that are officially supported (no downstream packaging issues)
- discoverable through a central repository with no restrictions, support for [many more repositories](https://github.com/trytomakeyouprivate/flatpak-remotes)
- shared libraries, [using deduplication](https://gitlab.com/TheEvilSkeleton/flatpak-dedup-checker)
- uses [Appstream metadata](https://www.freedesktop.org/software/appstream/docs/chap-Quickstart.html) for **an app name**, description, changelogs, screenshots and more
- Flathub has a rating system, displays the used license and a lot more. It can be filtered by `verified`, `floss` and `verified & floss`.

**Flatpak is by default installed** on Fedora, OpenSuse, Linux Mint, Debian, Pop!_OS, TuxedoOS, EndlessOS, ElementaryOS, PureOS, Solus, VanillaOS and more.

It is established and won't go away.

### The only use case for Appimages
If users want to carry applications around on a thumbdrive, or run on a fully immutable system like TAILS, Appimages may be needed. But this is the only target, and it is not a standard use case.

This would theoretically be possible with flatpaks, but not currently implemented.

---

There are many projects repackaging Appimages as Flatpaks, and those can then be maintained by the original developers.

To all developers considering using Appimage for "their linux version":

**DON'T**

Please just use Flatpak, save us all the trouble and support a modern Linux desktop.
