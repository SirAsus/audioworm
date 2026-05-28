# About AudioWorm & Features
---
*AudioWorm* is a music player designed for convenient offline listening of music directly from the user's computer. The music player features a simple and intuitive interface in a skeuomorphic style. The program is also highly configurable and customizable. It supports plugins and user themes, but only 2 themes are available by default: white (WhiteTechno) and dark (BlackDream). Which theme is applied depends on the GTK theme. The program is written entirely in C using GTK+, and uses CSS technology for its unique interface. It supports almost all possible music formats, even the oldest ones. The program includes a convenient equalizer, numerous playlist coordination modes, and more. For convenience, there is a built-in file manager through which music can also be opened. When loading music with tags, all tags will be displayed. You can enable the song title to be shown in the program's window title, and this can be adjusted as desired. Drag-and-drop technology is also present, allowing you to drag a music file into the program to add it to the playlist.

# How to Install
---
To install from GitHub on Debian, go to Releases, select the latest release, and download the installation .deb file from there.

You can also compile the program. To do this, run the following in your Linux terminal:
```
git clone https://github.com/SirAsus/audioworm
```
Then navigate to the project directory and run:
```
doas/sudo make install
```
You also need to install the following dependencies:
  - GCC (with C11 support)
  - GTK+ 3.0 development files
  - GStreamer 1.0 development files
  - GStreamer Plugins Base
  - GLib 2.0

# How to Create Your Own Theme & Plugin?
Audioworm is the most customizable music player. Audioworm plugins are shared libraries (.so files) written in C. They are loaded when the program starts from the ~/.audioworm/plugins/ directory or manually via the View -> Add Plugin... menu. Plugins can radically change the program, but they can also contain malicious software, since a plugin is a full C program. *Be careful.*
For more details, see [Information](https://github.com/SirAsus/audioworm/blob/main/text/plugin.md)

You can also create your own themes in AudioWorm. This is much easier than plugins because you only need to know CSS to create a theme.

Archive structure of a theme:
```
mytheme.zip
├── theme.css
└── icons/ (optional)
├── start.png
├── pause.png
├── stop.png
├── next.png
├── previous.png
├── forward.png
├── repeat.png
├── random.png
├── eq.png
├── muteon.png
├── muteoff.png
└── music.png
```

# Contact & Support
The AudioWorm program was created entirely by SirAsus alone. Any support from you is very important to the author.
Contact email:
<popbob.mail@gmail.com>
