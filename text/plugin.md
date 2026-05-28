# Audioworm Plugin Development Guide

Audioworm plugins are shared libraries (`.so` files) loaded at runtime via `GModule`. Plugins have full access to the `App` struct and can modify any aspect of the player.

A plugin must export one function: `void plugin_init(App *app);`

## Required App Struct Definition

```c
#include <gtk/gtk.h>
#include <stdlib.h>
#include <string.h>
#include <gmodule.h>

typedef struct AudiowormApp App;

struct AudiowormApp {
    GtkWidget *window;
    GtkWidget *outer;
    GtkWidget *mainbg;
    GtkWidget *playbtn;
    GtkWidget *pausebtn;
    GtkWidget *stopbtn;
    GtkWidget *nextbtn;
    GtkWidget *prevbtn;
    GtkWidget *volslider;
    GtkWidget *seekslider;
    GtkWidget *timelabel;
    GtkWidget *durlabel;
    GtkWidget *titlelabel;
    GtkWidget *artistlabel;
    GtkWidget *albumlabel;
    GtkWidget *coverimg;
    GtkWidget *playlistview;
    GtkListStore *playliststore;
    GtkWidget *fileview;
    GtkListStore *filestore;
    GtkWidget *tabs;
    GtkWidget *status;
    GtkWidget *repeatbtn;
    GtkWidget *shufflebtn;
    GtkWidget *sequentialbtn;
    GtkWidget *eqwin;
    GtkWidget *keywin;
    GtkWidget **eqscales;
    GtkWidget *mutebtn;
    GtkWidget *speedscale;
    GtkWidget *pitchscale;
    GtkWidget *echoscale;
    GtkWidget *reverbscale;
    GtkWidget *stereoscale;
    void *pipeline;
    void *equalizer;
    void *vol;
    void *pitch;
    void *audiopitch;
    void *audioecho;
    void *audioreverb;
    void *audiostereo;
    GList *tracks;
    GList *current;
    gint count;
    gint index;
    gboolean playing;
    gboolean repeat;
    gboolean shuffle;
    gboolean sequential;
    gboolean seeking;
    gboolean muted;
    gint64 duration;
    gint premutevol;
    GList *shuffleremaining;
    gchar *confdir;
    gchar *conffile;
    gchar *iconpath;
    gchar *themedir;
    gchar *iconsdir;
    gchar *musiciconpath;
    gchar *plugindir;
    void *css;
    gchar *customcsspath;
    gboolean darktheme;
    gint width;
    gint height;
    gint posx;
    gint posy;
    gint volume;
    gboolean showtitle;
    gdouble opacity;
    gchar *keys[10];
    gchar *keynames[10];
    GList *plugins;
};

typedef struct {
    gchar *path;
    gchar *title;
    gchar *artist;
    gchar *album;
    gchar *duration;
    gint seconds;
    void *cover;
    gchar *filename;
    gchar *displayname;
} Track;

# Complete Plugin Example: Track Info Panel

\```c
#include <gtk/gtk.h>
#include <stdlib.h>
#include <string.h>
#include <gmodule.h>

typedef struct AudiowormApp App;

struct AudiowormApp {
    GtkWidget *window;
    GtkWidget *outer;
    GtkWidget *mainbg;
    GtkWidget *playbtn;
    GtkWidget *pausebtn;
    GtkWidget *stopbtn;
    GtkWidget *nextbtn;
    GtkWidget *prevbtn;
    GtkWidget *volslider;
    GtkWidget *seekslider;
    GtkWidget *timelabel;
    GtkWidget *durlabel;
    GtkWidget *titlelabel;
    GtkWidget *artistlabel;
    GtkWidget *albumlabel;
    GtkWidget *coverimg;
    GtkWidget *playlistview;
    GtkListStore *playliststore;
    GtkWidget *fileview;
    GtkListStore *filestore;
    GtkWidget *tabs;
    GtkWidget *status;
    GtkWidget *repeatbtn;
    GtkWidget *shufflebtn;
    GtkWidget *sequentialbtn;
    GtkWidget *eqwin;
    GtkWidget *keywin;
    GtkWidget **eqscales;
    GtkWidget *mutebtn;
    GtkWidget *speedscale;
    GtkWidget *pitchscale;
    GtkWidget *echoscale;
    GtkWidget *reverbscale;
    GtkWidget *stereoscale;
    void *pipeline;
    void *equalizer;
    void *vol;
    void *pitch;
    void *audiopitch;
    void *audioecho;
    void *audioreverb;
    void *audiostereo;
    GList *tracks;
    GList *current;
    gint count;
    gint index;
    gboolean playing;
    gboolean repeat;
    gboolean shuffle;
    gboolean sequential;
    gboolean seeking;
    gboolean muted;
    gint64 duration;
    gint premutevol;
    GList *shuffleremaining;
    gchar *confdir;
    gchar *conffile;
    gchar *iconpath;
    gchar *themedir;
    gchar *iconsdir;
    gchar *musiciconpath;
    gchar *plugindir;
    void *css;
    gchar *customcsspath;
    gboolean darktheme;
    gint width;
    gint height;
    gint posx;
    gint posy;
    gint volume;
    gboolean showtitle;
    gdouble opacity;
    gchar *keys[10];
    gchar *keynames[10];
    GList *plugins;
};

typedef struct {
    gchar *path;
    gchar *title;
    gchar *artist;
    gchar *album;
    gchar *duration;
    gint seconds;
    void *cover;
    gchar *filename;
    gchar *displayname;
} Track;

typedef struct {
    App *app;
    GtkWidget *label;
    guint timer;
} PluginData;

static gboolean update(gpointer data) {
    PluginData *pd = (PluginData *)data;
    gchar *text = g_strdup_printf("Tracks: %d | Playing: %s",
        pd->app->count,
        pd->app->playing ? "Yes" : "No");
    gtk_label_set_text(GTK_LABEL(pd->label), text);
    g_free(text);
    return TRUE;
}

void plugin_init(App *app) {
    PluginData *pd = g_new0(PluginData, 1);
    pd->app = app;

    pd->label = gtk_label_new("Tracks: 0 | Playing: No");
    gtk_widget_set_halign(pd->label, GTK_ALIGN_START);

    GtkWidget *plbox = gtk_widget_get_parent(app->playlistview);
    plbox = gtk_widget_get_parent(plbox);
    gtk_box_pack_start(GTK_BOX(plbox), pd->label, FALSE, FALSE, 0);
    gtk_box_reorder_child(GTK_BOX(plbox), pd->label, 0);

    pd->timer = g_timeout_add(1000, update, pd);
    gtk_widget_show_all(pd->label);
}
\```

# Compile
\```
gcc -std=gnu11 -shared -fPIC -o trackinfo.so trackinfo.c `pkg-config --cflags gtk+-3.0`
\```
# Install
\```
cp trackinfo.so ~/.audioworm/plugins/
\```
Restart Audioworm or load via *View → Add Plugin...*

# App Struct Reference

| Field | Type | Description |
|-------|------|-------------|
| `app->window` | `GtkWidget*` | Main application window |
| `app->outer` | `GtkWidget*` | Outermost vertical box container |
| `app->mainbg` | `GtkWidget*` | Main background box below menubar |
| `app->playbtn` | `GtkWidget*` | Play button |
| `app->pausebtn` | `GtkWidget*` | Pause button |
| `app->stopbtn` | `GtkWidget*` | Stop button |
| `app->nextbtn` | `GtkWidget*` | Next track button |
| `app->prevbtn` | `GtkWidget*` | Previous track button |
| `app->volslider` | `GtkWidget*` | Volume slider (0-100) |
| `app->seekslider` | `GtkWidget*` | Seek/progress slider (0-1000) |
| `app->timelabel` | `GtkWidget*` | Current time label (e.g. "1:23") |
| `app->durlabel` | `GtkWidget*` | Duration label (e.g. "4:56") |
| `app->titlelabel` | `GtkWidget*` | Track title display label |
| `app->artistlabel` | `GtkWidget*` | Track artist display label |
| `app->albumlabel` | `GtkWidget*` | Track album display label |
| `app->coverimg` | `GtkWidget*` | Album art image widget |
| `app->playlistview` | `GtkWidget*` | Playlist tree view |
| `app->playliststore` | `GtkListStore*` | Playlist data store (columns: 0=Title, 1=Artist, 2=Index) |
| `app->fileview` | `GtkWidget*` | File browser tree view |
| `app->filestore` | `GtkListStore*` | File browser data store (columns: 0=Name, 1=Path, 2=Icon) |
| `app->tabs` | `GtkWidget*` | Notebook tabs (Playlist, File Browser) |
| `app->status` | `GtkWidget*` | Status bar label at bottom |
| `app->repeatbtn` | `GtkWidget*` | Repeat mode toggle button |
| `app->shufflebtn` | `GtkWidget*` | Shuffle mode toggle button |
| `app->sequentialbtn` | `GtkWidget*` | Sequential mode toggle button |
| `app->eqwin` | `GtkWidget*` | Equalizer window (NULL if closed) |
| `app->keywin` | `GtkWidget*` | Keyboard shortcuts window (NULL if closed) |
| `app->eqscales` | `GtkWidget**` | Array of 10 equalizer band sliders |
| `app->mutebtn` | `GtkWidget*` | Mute button |
| `app->speedscale` | `GtkWidget*` | Speed control slider (0.5-2.0) |
| `app->pitchscale` | `GtkWidget*` | Pitch control slider (0.5-2.0) |
| `app->echoscale` | `GtkWidget*` | Echo effect slider (0-1.0) |
| `app->reverbscale` | `GtkWidget*` | Reverb effect slider (0-1.0) |
| `app->stereoscale` | `GtkWidget*` | Stereo width slider (0-1.0) |
| `app->pipeline` | `void*` | GStreamer pipeline (GstElement*) |
| `app->equalizer` | `void*` | GStreamer equalizer element (GstElement*) |
| `app->vol` | `void*` | GStreamer volume element (GstElement*) |
| `app->pitch` | `void*` | GStreamer pitch element (GstElement*) |
| `app->audiopitch` | `void*` | GStreamer scaletempo element (GstElement*) |
| `app->audioecho` | `void*` | GStreamer audioecho element (GstElement*) |
| `app->audioreverb` | `void*` | GStreamer audioreverb element (GstElement*) |
| `app->audiostereo` | `void*` | GStreamer stereo element (GstElement*) |
| `app->tracks` | `GList*` | Linked list of `Track*` - the playlist |
| `app->current` | `GList*` | GList node of currently playing track |
| `app->count` | `gint` | Number of tracks in playlist |
| `app->index` | `gint` | Index of current track (-1 if none) |
| `app->playing` | `gboolean` | TRUE if audio is playing |
| `app->repeat` | `gboolean` | TRUE if repeat mode enabled |
| `app->shuffle` | `gboolean` | TRUE if shuffle mode enabled |
| `app->sequential` | `gboolean` | TRUE if sequential mode enabled |
| `app->seeking` | `gboolean` | TRUE while user is dragging seek slider |
| `app->muted` | `gboolean` | TRUE if audio is muted |
| `app->duration` | `gint64` | Duration of current track in nanoseconds |
| `app->premutevol` | `gint` | Volume level before muting (0-100) |
| `app->shuffleremaining` | `GList*` | List of unplayed indices for shuffle mode |
| `app->confdir` | `gchar*` | Path to `~/.audioworm/` directory |
| `app->conffile` | `gchar*` | Path to `~/.audioworm/config.ini` |
| `app->iconpath` | `gchar*` | Path to `~/.icons/audioworm.png` |
| `app->themedir` | `gchar*` | Path to `~/.audioworm/theme/` |
| `app->iconsdir` | `gchar*` | Path to `~/.audioworm/icons/` |
| `app->musiciconpath` | `gchar*` | Path to `~/.audioworm/icons/music.png` |
| `app->plugindir` | `gchar*` | Path to `~/.audioworm/plugins/` |
| `app->css` | `void*` | GtkCssProvider for current theme |
| `app->customcsspath` | `gchar*` | Path to custom CSS file (NULL if default) |
| `app->darktheme` | `gboolean` | TRUE if dark theme is active |
| `app->width` | `gint` | Window width in pixels |
| `app->height` | `gint` | Window height in pixels |
| `app->posx` | `gint` | Window X position |
| `app->posy` | `gint` | Window Y position |
| `app->volume` | `gint` | Current volume level (0-100) |
| `app->showtitle` | `gboolean` | TRUE if track info shown in window title |
| `app->opacity` | `gdouble` | Window opacity (0.3-1.0) |
| `app->keys[10]` | `gchar*[]` | Keyboard shortcut key names |
| `app->keynames[10]` | `gchar*[]` | Keyboard shortcut display names |
| `app->plugins` | `GList*` | Linked list of loaded GModule plugin handles |

## Track Struct

| Field | Type | Description |
|-------|------|-------------|
| `track->path` | `gchar*` | Full file path to audio file |
| `track->title` | `gchar*` | Title from ID3 tag (NULL if none) |
| `track->artist` | `gchar*` | Artist from ID3 tag (NULL if none) |
| `track->album` | `gchar*` | Album from ID3 tag (NULL if none) |
| `track->duration` | `gchar*` | Duration string (e.g. "3:45") |
| `track->seconds` | `gint` | Duration in seconds |
| `track->cover` | `void*` | Album art GdkPixbuf (NULL if none) |
| `track->filename` | `gchar*` | Original filename (e.g. "song.mp3") |
| `track->displayname` | `gchar*` | Display name (title if available, else filename without extension) |
