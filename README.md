# Hyprscroller

[Hyprscroller](https://github.com/dawsers/hyprscroller) is a
[Hyprland](https://hyprland.org) layout plugin that creates a window layout
similar to [PaperWM](https://github.com/paperwm/PaperWM).

![Intro](./videos/hyprscroller.gif)

The plugin is quite feature complete and supports gaps, borders, decorations,
special workspace, full screen modes, overview, marks, pinned columns,
touchpad gestures, copying/pasting windows, trails/trailmarks, quick jump mode,
and installation through `hyprpm`.

Check the [Tutorial](./TUTORIAL.md) for a quick overview of *hyprscroller's* main
features.

I use *hyprscroller* on my main machine and will support it for as long as I
keep on using *Hyprland*. However, I will only add new features that I find
interesting, and support two *Hyprland* versions: the one my distribution uses,
and the latest tagged one. I have found problematic compiling trunk versions
of *Hyprland* on a system that already has a system-wide version of it
installed, so I will not make an extra effort there until things improve in
that front.


## Requirements

*hyprscroller* supports the version of *Hyprland* I use, which should be the
same as the Arch Linux `hyprland` package (v0.45.2). You can try your luck with the
latest `git` changes, but I will be slower to keep up with those, as there are
too many API changes going on upstream.

Aside from those two versions, even though the feature set will be frozen for
them, *hyprscroller* also supports most official Hyprland releases from v0.35
to the current one.


## Building and installing

The easiest and recommended mode to install *hyprscroller* is through `hyprpm`

### hyprpm

``` sh
hyprpm add https://github.com/dawsers/hyprscroller
# verify it installed correctly
hyprpm list
```

You can enable or disable it via `hyprpm enable hyprscroller` and
`hyprpm disable hyprscroller` or update it using `hyprpm update hyprscroller`.

Adding `exec-once = hyprpm reload -n` to your `hyprland.conf` will ensure all
your `hyprpm` managed plugins will be loaded at startup.

### Manually

If you want to build the plugin manually, it should be as simple as running

``` sh
# builds a shared object hyprscroller.so
make all
# installs the shared library in ~/.config/hypr/plugins
make install
```

then you can add the plugin to your `hyprland.conf`

``` conf
# path must be absolute
plugin = /home/xxxx/.config/hypr/plugins/hyprscroller.so
```

or load it temporarily using `hyprctl plugin`

``` sh
# path must be absolute
hyprctl plugin load /home/xxxx/.config/hypr/plugins/hyprscroller.so
```


### NixOS

I don't use NixOS, so the "flakes" in this repo are maintained by users, and
may not be up to date. However, it seems *hyprscroller* is now an official
unstable package in [nixpkgs](https://search.nixos.org/packages?channel=unstable&from=0&size=50&sort=relevance&type=packages&query=hyprlandPlugins.hyprscroller) so you can install it from there.


## Configuration

If you are not using `hyprpm`, to make Hyprland load the plugin, add this to
your configuration.

``` conf
# path must be absolute
plugin = /home/xxxx/.config/hypr/plugins/hyprscroller.so
```

Instead, if you use `hyprpm`, it should be as simple as adding this to
your `~/.config/hypr/hyprland.conf` :

``` conf
exec-once = hyprpm reload -n
```

To turn on the layout, use

``` conf
general {
    ...

    layout = scroller

    ...
}
```


## Dispatchers

The plugin adds the following dispatchers:

| Dispatcher                  | Description                                                                                                                 |
|-----------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| `scroller:movefocus`        | An optional replacement for `movefocus`, takes a direction as argument.                                                     |
| `scroller:movewindow`       | An optional replacement for `movewindow`, takes a direction as argument.                                                    |
| `scroller:setmode`          | Set mode: `r/row` (default), `c/col/column`. Sets the working mode. Affects most dispatchers and new window creation.       |
| `scroller:cyclesize`        | Resize the focused column width (*row* mode), or the active window height (*column* mode).                                  |
| `scroller:alignwindow`      | Align window on the screen, `l/left`, `c/center`, `r/right` (*row* mode), `c/center`, `u/up`, `d/down` (*col* mode)         |
| `scroller:admitwindow`      | Push the current window below the active one of the column to its left.                                                     |
| `scroller:expelwindow`      | Pop the current window out of its column and place it on a new column to the right.                                         |
| `scroller:fitsize`          | Resize columns (*row* mode) or windows (*col* mode) so they fit on the screen: `active`, `visible`, `all`, `toend`, `tobeg` |
| `scroller:toggleoverview`   | Toggle an overview of the workspace where all the windows are temporarily scaled to fit the monitor                         |
| `scroller:marksadd`         | Add a named mark. Argument is the name of the mark                                                                          |
| `scroller:marksdelete`      | Delete a named mark. Argument is the name of the mark                                                                       |
| `scroller:marksvisit`       | Visit a named mark. Argument is the name of the mark                                                                        |
| `scroller:marksreset`       | Delete all marks                                                                                                            |
| `scroller:pin`              | Toggle pin a column to its current position. The rest will adapt when changing focus etc.                                   |
| `scroller:selectiontoggle`  | Toggle on/off the selection status of a window                                                                              |
| `scroller:selectionreset`   | Resets selection (deselects all windows)                                                                                    |
| `scroller:selectionmove`    | Moves the selected windows/columns to the current workspace and location, takes a direction as argument (keeps sizes etc.)  |
| `scroller:trailnew`         | Creates a new trail                                                                                                         |
| `scroller:traildelete`      | Deletes the active trail                                                                                                    |
| `scroller:trailclear`       | Clears all the trailmarks of the current trail                                                                              |
| `scroller:trailnext`        | Moves to next trail                                                                                                         |
| `scroller:trailprevious`    | Moves to previous trail                                                                                                     |
| `scroller:trailtoselection` | Creates a selection from all the windows in the current trail                                                               |
| `scroller:trailmarktoggle`  | Toggles a trailmark for the current window in the active trail                                                              |
| `scroller:trailmarknext`    | Moves to next trailmark in the current trail                                                                                |
| `scroller:trailmarkprevious`| Moves to previous trailmark in the current trail                                                                            |
| `scroller:jump`             | Shows every window on the active monitors for a shortcut-based, quick focus mode                                            |


## Modes

*Hyprscroller* works in any of two modes that can be changed at any moment.

1. *row* mode: it is the default. It creates new windows in a new column.
   `cyclesize` affects the width of the active column. `alignwindow` aligns
   the active column according to the argument received. `fitsize` fits the
   selected columns to the width of the monitor.

2. *column* mode: It creates new windows in the current column, right below the
   active window. `cyclesize` affects the height of the active window.
   `alignwindow` aligns the active window within the column, according to the
   argument received. `fitsize` fits the selected windows in the column to the
   height of the monitor.


## Window/Column Focus and Movement

*Hyprscroller* is now compatible with *Hyprland's* default `movefocus`/`movewindow`
dispatchers and key bindings. But you can also add *Hyprscroller's* own
`scroller:movefocus`/`scroller:movewindow` for cases unsupported by Hyprland,
like `beginning` and `end`. Supporting the default dispatcher allows for
better switching of layouts without needing to modify the configuration, which
was the case in the past.

`movefocus` and `movewindow` accept the following directional arguments:
`l` or `left`, `r` or `right`, `u` or `up`, `d` or `dn` or `down`, `b` or
`begin` or `beginning`, `e` or `end`. So you can focus or move windows/columns
in a direction or to the beginning or end or the row.


## Resizing

`cyclesize` accepts an argument which is either `+1`/`1`/`next`, or
`-1`/`prev`/`previous`. It cycles forward or backward through a number of column
widths (in *row* mode), or window heights (in *column* mode). Those widths or
heights are a fraction of the width or height of the monitor, and are
configurable (see *options*). However, using the dispatcher `resizewindow`, you
can modify the width or height of any window freely.


## Aligning

Columns are generally aligned in automatic mode, always making the active one
visible, and trying to make at least the previously focused one visible too if
it fits the viewport, if not, the one adjacent on the other side. However, you
can always align any column to the *center*, *left* or *right* of the monitor
(in *row* mode), or *up* (top), *down* (bottom) or to the *center* in *column*
mode. For example center a column for easier reading, regardless of what happens to
the other columns. As soon as you change focus or move a column, the alignment
is lost.

`alignwindow` takes a parameter: `l` or `left`, `r` or `right`, `c` or
`center` or `centre`, `u` or `up` and `d` or `down`.

To use *right* or *left* you need to be in *row* mode, and to use *up* or
*down* in *column* mode. *center* behaves differently depending on the mode.
In *row* mode it aligns the active column to the center of the monitor. In
*column* mode, it aligns the active window within its column, to a centered
position.


## Admit/Expel

You can create columns of windows using `admitwindow`. It takes the active
window and moves it to the column left of its current one, right under the
active window in that column.

To expel any window from its current column and position it in a new column to
its right, use `expelwindow`.


## Fitting the Screen

When you have a ultra-wide monitor, one in a vertical position, or the default
column widths or window heights don't fit your workflow, you can use manual
resizing, but it is sometimes slow and tricky.

`scroller:fitsize` works in two different ways, depending on the active mode.

It allows you to re-fit the columns (*row* mode) or windows (*column* mode) you
want to the screen extents. It accepts an argument related to the
columns/windows it will try to fit. The new width/height of each column/window
will be proportional to its previous width or height, relative to the other
columns or windows affected.

1. `active`: It is similar to maximize, it will fit the active column/window.
2. `visible`: All the currently fully or partially visible columns/windows will
   be resized to fit the screen.
3. `all`: All the columns in the row or windows in the column will be resized
   to fit.
4. `toend`: All the columns or windows from the focused one to the end of the
   row/column will be affected.
5. `tobeg` or `tobeginning`: All the columns/windows from the focused one to
   the beginning of the row/column will now fit the screen.


## Overview

`scroller:toggleoverview` toggles a bird's eye view of the current workspace where
all the windows are scaled to fit the current monitor. You can still interact
with the windows normally (change focus, move windows, create or destroy them,
type in them etc.). Use it as a way to see where things are and move the active
focus, or reposition windows. You can even have all your windows in full screen
mode and use *overview* to navigate among them quickly.

*Overview* is affected by the option `overview_scale_content`, which is by
default `true`, meaning the windows' content will be scaled in the same
proportion as the windows. If your system doesn't support this (currently only
supported on x86_64 -Intel/AMD CPUs), the default value will be `false`. If
you simply want to have the content at the original size, you can also turn
this option manually to `false`.

## Jump

`scroller:jump` provides a shortcut-based quick focus mode for any window on
the active workspaces, similar to [vim-easymotion](https://github.com/easymotion/vim-easymotion)
and variations.

It shows all the windows on your monitors' active workspaces in overview, and
waits for a combination of key presses (overlaid on each window) to quickly
change focus to the selected window. Pressing any key that is not on the list or
a combination that doesn't exist, exits jump mode without changes.

Depending on the total number of windows and keys you set on your list, you
will have to press more or less times. Each window has its full combination
on the overlay.

You can call `jump` from any mode: overview, full screen or normal mode.

There are four options related to `scroller:jump`:
`plugin:scroller:jump_labels_font`, `plugin:scroller:jump_labels_color`,
`plugin:scroller:jump_labels_scale` and `plugin:scroller:jump_labels_keys`.
These options are explained in detail in **Options**.

## Marks

You can use *marks* to navigate to frequently used windows, regardless of
which workspace they are in (it even works for the special workspace windows).

`scroller:marksadd` adds a named mark. Use a *submap* to create bindings for
several named marks you may want to use. See the configuration example for
directions.

`scroller:marksdelete` deletes a named mark created with `scroller:marksadd`.

`scroller:marksvisit` moves the focus to a previously created mark.

`scroller:marksreset` clears all marks.

Marks reference windows, but are global, they may belong to different
workspaces, so visiting a mark may switch workspaces.

You can use any string name for a mark, for example in scripts. But they are
also very convenient to use with regular key bindings by simply using a letter
as the name. Again, see the example configuration.


## Pinning a Column

`scroller:pin` manages *pinned* columns (toggles *pinned* status). Having a
column *pinned* is useful to keep some column as an immovable reference; for
example some documentation or important code you want to always have available
while changing focus to other columns.

`scroller:pin` will *pin* a column if none is *pinned*, otherwise will *unpin*
the *pinned* one. So to move the *pin* from one column to another, call
`scroller:pin` twice, one to *unpin* the currently *pinned* one, and the
second to *pin* the active column.

There can only be one *pinned* column per workspace. The column will stay in
place while the flag is on. That means the rest of the columns (even the
active one) will not perturb that position. When changing focus, the *active* 
column will be seen on the screen as long as it fits on either side of the
*pinned* column.


## Window Copying/Pasting

`scroller:selectiontoggle` and `scroller:selectionreset` manage window/column
selections. You can select several windows at a time, even in different
workspaces and/or from overview mode. Those windows will change the border
color to the one specified in the option `plugin:scroller:col.selection_border`.

Once you have made a selection, you can move those windows to a different
workspace or location in the same workspace using `scroller:selectionmove`.
The selection order and column/window configuration will be maintained.

`scroller:selectionmove` accepts a direction as parameter. Valid directions
are:

1. `r` or `right`: Move the selection to the current workspace, locating it
   right of the active column.
2. `l` or `left`: Move the selection to the current workspace, locating it
   left of the active column.
3. `b`, `begin`, or `beginning`: Move the selection to the current workspace,
   locating it at the beginning (first column).
4. `e` or `end`: Move the selection to the current workspace, locating it at
   the end (last column).


## Trails and Trailmarks

Trails and Trailmarks are a concept borrowed from [trailblazer.nvim](https://github.com/LeonHeidelbach/trailblazer.nvim).

A **trailmark** is like an anonymous mark on a window, and a **trail** is a
collection of trailmarks. You can have as many trails as you want, and as many
trailmarks as you want in any trail. Each window can be in as many trails
as you want too.

Creating your first trailmark (`trailmarktoggle`) will create a trail. From
then on, every trailmark you create will be assigned to that trail. You can
navigate back (`trailmarkprevious`) and forth (`trailmarknext`) within the
collection of trailmarks contained in the trail.

To create a new trail, use `trailnew`. With `trailprevious` and `trailnext`
you can navigate trails, changing the active one. The active trail will be
the one used for the trailmark dispatchers (toggle, next, and previous).

Clear all the trailmarks of the active trail using `trailclear`, or delete
the trail from the list with `traildelete`.

`trailtoselection` creates a selection list from the trailmarks in the active
trail. You can use that selection for example to move all the windows to a new
workspace using `selectionmove`.

*hyprscroller* generates IPC signals for trail/trailmark events, and in this
README there is an example implementation for [ags](https://github.com/Aylur/ags)
to use those signals to display information on your desktop bar.

See the key bindings section for an example on how to set bindings for trail
and trailmark dispatchers.


## Touchpad Gestures

There are options to enable/disable touchpad gestures for `movefocus`
(scrolling), `overview` and `workspace` change.

The default for scrolling is swiping with three fingers to scroll left, right,
up or down. Four fingers up enables the *overview* mode, down disables it.
To change workspace, swipe right or left, also with four fingers.

*hyprscroller* touchpad gestures respect the global option
`input:touchpad:natural_scroll` and provide several others specific to the
plugin to tweak gestures behavior. You can find them in the **Options** section
of this document.

**Note**: Hyprland's `workspace_swipe` will be disabled while using
*hyprscroller*. Please, use the provided workspace change swipe gesture (four
fingers right/left by default, but you can change the number of fingers).

Hyprland doesn't understand windows in a workspace can exist outside of the viewport, so
`workspace_swipe` will not work, as it renders those windows thinking they are in
a different workspace, creating a small mess of windows. If you have forgotten
to disable it, *hyprscroller* will have to turn off three finger gestures until
you disable `workspace_swipe`.


## IPC

*hyprscroller* sends IPC messages in certain situations, in the same way
[*hyprland* does](https://wiki.hyprland.org/IPC/). These are currently the
events that trigger a message:

| Message                | Occurs when                | Data                |
|------------------------|----------------------------|---------------------|
| `scroller admitwindow` | admitting a window         |                     |
| `scroller expelwindow` | expelling a window         |                     |
| `scroller overview`    | toggling overview mode     | `0/1`               |
| `scroller mode`        | changing mode              | `row/column`        |
| `scroller trail`       | current trail information  | `number`, `size`    |
| `scroller trailmark`   | current window trailmark?  | `0/1`               |

You can use these events to show messages, or modify your bar. This simple
script captures the events and shows a notification each time:

``` bash
#!/usr/bin/env bash

function handle {
  if [[ ${1:0:8} == "scroller" ]]; then
    if [[ ${1:10:11} == "overview, 0" ]]; then
        hyprctl notify -1 3000 "rgb(ff1ea3)" "Normal mode!"
    elif [[ ${1:10:11} == "overview, 1" ]]; then
        hyprctl notify -1 3000 "rgb(ff1ea3)" "Overview mode!"
    elif [[ ${1:10:9} == "mode, row" ]]; then
        hyprctl notify -1 3000 "rgb(ff1ea3)" "Row mode!"
    elif [[ ${1:10:12} == "mode, column" ]]; then
        hyprctl notify -1 3000 "rgb(ff1ea3)" "Column mode!"
    elif [[ ${1:10:11} == "admitwindow" ]]; then
        hyprctl notify -1 3000 "rgb(ff1ea3)" "Admit Window!"
    elif [[ ${1:10:11} == "expelwindow" ]]; then
        hyprctl notify -1 3000 "rgb(ff1ea3)" "Expel Window!"
    fi
  fi
}

socat - "UNIX-CONNECT:$XDG_RUNTIME_DIR/hypr/$HYPRLAND_INSTANCE_SIGNATURE/.socket2.sock" | while read -r line; do handle "$line"; done
```

### Using IPC in AGS

For those using [ags](https://github.com/Aylur/ags) to create the desktop bar,
this is a module example on how you could use *hyprscroller* IPC events to show
the current mode (*row* or *column*), whether we are in *overview* mode or not,
your current trail (and its number of trailmarks), and whether the current
window contains a trailmark or not.

``` js
import Gio from 'gi://Gio'
import GLib from 'gi://GLib'

export function Scroller() {
    const decoder = new TextDecoder();
    const mode_label = Widget.Label({
        class_name: "scroller-mode",
        label: ''
    });
    const overview_label = Widget.Label({
        class_name: "scroller-overview",
        label: ''
    });
    const trailmark_label = Widget.Label({
        class_name: "scroller-trailmark",
        label: ''
    });
    const trail_label = Widget.Label({
        class_name: "scroller-trail",
        label: ''
    });
    const scroller = Widget.Box({
        class_name: "scroller",
        children: [
            mode_label,
            overview_label,
            trail_label,
            trailmark_label,
        ],
    })
    function event_decode(data) {
        const text = decoder.decode(data);
        if (text.startsWith("scroller>>mode,")) {
            const mode = text.substring(15).trim();
            if (mode == "row")
                mode_label.label = "-";
            else
                mode_label.label = "|";
        } else if (text.startsWith("scroller>>overview,")) {
            if (text.substring(19) == 1) {
                overview_label.label = "🐦";
            } else {
                overview_label.label = "";
            }
        } else if (text.startsWith("scroller>>trail,")) {
            const trail = text.substring(16).trim().split(",");
            const tnumber = trail[0].trim();
            if (tnumber == "-1") {
                trail_label.label = "";
            } else {
                const tsize = trail[1].trim();
                trail_label.label = tnumber + " (" + tsize + ")";
            }
        } else if (text.startsWith("scroller>>trailmark,")) {
            if (text.substring(20) == 1) {
                trailmark_label.label = "✅";
            } else {
                trailmark_label.label = "";
            }
        }
    }
    function connection() {
        const HIS = GLib.getenv('HYPRLAND_INSTANCE_SIGNATURE');
        const XDG_RUNTIME_DIR = GLib.getenv('XDG_RUNTIME_DIR') || '/';
        const sock = (pre) => `${pre}/hypr/${HIS}/.socket2.sock`;
        const path = GLib.file_test(sock(XDG_RUNTIME_DIR), GLib.FileTest.EXISTS)?
            sock(XDG_RUNTIME_DIR) : sock('/tmp');

        return new Gio.SocketClient()
            .connect(new Gio.UnixSocketAddress({ path }), null);
    }
    let listener = new Gio.DataInputStream({
        close_base_stream: true,
        base_stream: connection().get_input_stream(),
    });
    function watch_socket(sstream) {
        sstream.read_line_async(0, null, (stream, result) => {
            if (!stream)
                return console.error('Error reading Hyprland socket');

            const [line] = stream.read_line_finish(result);
            event_decode(line);
            watch_socket(stream);
        });
    }
    watch_socket(listener);
    return scroller;
}
```

and a simple `style.css`

``` css
@define-color wb-background #000000;
@define-color wb-primary #f0c671;
@define-color wb-green #00b35b;

.scroller {
    color: @wb-primary;
    background: @wb-background;
    min-width: 60px;
    padding-left: 15px;
}

.scroller-mode {
  padding-top: 10px;
  padding-bottom: 10px;
  padding-right: 10px;
}
.scroller-overview {
  padding-top: 10px;
  padding-bottom: 10px;
  padding-right: 10px;
}
.scroller-trailmark {
  padding-top: 10px;
  padding-bottom: 10px;
  padding-right: 10px;
}
.scroller-trail {
  color: @wb-green;
  padding-top: 10px;
  padding-bottom: 10px;
  padding-right: 10px;
}
```

### Using IPC with Waybar

[This comment](https://github.com/dawsers/hyprscroller/issues/57#issuecomment-2418305146)
by Chen-Yulin provides code to use *hyprscroller's* IPC messages with
[waybar](https://github.com/Alexays/Waybar).


## Options

*hyprscroller* currently accepts the following options:

### `column_default_width`

Determines the width of new columns in *row* mode.
Possible arguments are: `oneeighth`, `onesixth`, `onefourth`, `onethird`,
`threeeighths`, `onehalf` (default), `fiveeighths`, `twothirds`, `threequarters`,
`fivesixths`, `seveneighths`, `maximized`, `floating` (uses the
default width set by the application).

### `window_default_height`

Determines the default height of a new window. This is useful if you are using
a monitor in portrait mode and *column* mode.
Possible arguments are: `oneeighth`, `onesixth`, `onefourth`, `onethird`,
`threeeighths`, `onehalf`, `fiveeighths`, `twothirds`, `threequarters`,
`fivesixths`, `seveneighths`, `one` (default).

### `focus_wrap`

Determines whether focus will *wrap* when at the first or
last window of a row/column. Possible arguments are: `true`|`1` (default), or
`false`|`0`.

### `overview_scale_content`

Scales the content of the windows in overview mode, like GNOME/MacOS/Windows
overview mode. Possible arguments are: `true`|`1` (default), or
`false`|`0`.

### `col.selection_border`

It is the color of the border of selected windows. The default value is
`0xff9e1515`, which is `red`.

### `jump_labels_font`

It is a string with the font to use for `jump` labels. If omitted, the default
is to use whatever you have set as default for Hyprland (`misc:font_family`).

### `jump_labels_color`

Color of the text for the `jump` labels. The default is `0x80159e30`, which
is a tone of green.

### `jump_labels_scale`

`jump` labels will be centered in each window, and this parameter scales their
size. The default is `0.5`. `1.0` would make the label fit the full size of
the window, and it's the maximum allowed, `0.1` is the minimum allowed, and
would make the label's box size 10% of the width and height of the window.

### `jump_labels_keys`

It is a string with the keys that will be used for the labels. The default is
`1234`, which means labels will show a unique combination of `1`, `2`, `3` and
`4`. The more keys you set on this option, the shorter the combination of key
presses to reach any window, but some times, reaching for those keys can be
slower. Some people prefer to use `qwerasdf` which provides a shorter label
and good reachability. The perfect combination depends on how you use
your keyboard.

### `column_widths`

Determines the set of column widths *hyprscroler* will
cycle through when resizing the width of a column in *row* mode. It is a string
of any number of values chosen among: *oneeighth, onesixth, onefourth, onethird,
threeeighths, onehalf, fiveeighths, twothirds, threequarters, fivesixths,
seveneighths, one*. The default value is: *onehalf twothirds onethird*.

### `window_heights`

Determines the set of window heights *hyprscroler* will
cycle through when resizing the height of a window in *column* mode. It is a
string of any number of values chosen among: *oneeighth, onesixth, onefourth,
onethird, threeeighths, onehalf, fiveeighths, twothirds, threequarters,
fivesixths, seveneighths, one*. The default value is:
*one onethird onehalf twothirds*.

### `monitor_options`

**NOTE:** This option **deprecates** `monitor_modes`. Please, update your
configuration if you are using it.

`monitor_options` can be used to define different default options for each
monitor. Currently, the supported options are:

1. `mode`: `r/row` for *row* mode and `c/col/column` for *column* mode.
2. `column_default_width`: Possible values are the same as the global
   `column_default_width` option.
3. `window_default_height`: Possible values are the same as the global
   `window_default_height` option.

When you create a workspace in any monitor, instead of defaulting to the
global options, it will read them from this configuration value. For any
monitor or option not defined in this variable, the option will default to the
global one.

Monitor names can be inferred by running `hyprctl monitors` and using the
returned names.

```
Monitor HDMI-A-1 (ID 0):
	3840x2160@59.99700 at 0x0
    ...
```

means the name you need to use is HDMI-A-1.

`monitor_options` is a list with the following format:

```
monitor_options = (DP-2 = (mode = row; column_default_width = onehalf; window_default_height = one), HDMI-A-1 = (mode = col; column_default_width = one; window_default_height = onehalf))
```

The list of monitors is encapsulated by `()` and separated by `,`. Each
monitor entry consists of the name of the monitor followed by `=` and a list
of options enclosed by `()`, with each option separated by `;`, as in the
example above. Spaces are allowed anywhere for better readability.

This option is useful to configure ultra-wide monitors or those in non-standard
orientations (for example portrait instead of landscape). You can define any
combination.

### `gesture_scroll_enable`

`true` (default) or `false`. Enables or disables touchpad gestures for
scrolling (changing focus).

### `gesture_scroll_fingers`

Integer value, default is `3`. Number of fingers used to swipe when scrolling.

### `gesture_scroll_distance`

Integer value, default is `60`. Delta generated by swiping to move focus one
window. It is like a sensitivity value; the smaller, the more sensitive
scrolling will be to swipes.

### `gesture_overview_enable`

`true` (default) or `false`. Enables or disables touchpad gestures to call
*overview* mode and workspace switching.

### `gesture_overview_fingers`

Integer value, default is `4`. Number of fingers used to swipe for overview
or changing workspace.

### `gesture_overview_distance`

Integer value, default is `5`. Delta generated by swiping to call *overview*
mode or change workspace. It is like a sensitivity value; the smaller, the
easier it will be to trigger the command. Each swipe triggers it only once,
regardless of the length or the swipe.

### `gesture_workspace_switch_prefix`

String value (one character). The default is "". It is the prefix used when
calling the dispatcher to switch to a different workspace. Read [Workspaces](https://wiki.hyprland.org/Configuring/Dispatchers/#workspaces)
in the Hyprland wiki to understand in detail what each prefix means.

The *prefix* value will be concatenated to `+1` or `-1` to change workspace
when swiping to one side or the other. For example:

``` conf
# Default: swith to the next/previous workspace, regardless of monitor
plugin:scroller:gesture_workspace_switch_prefix =
# swith to a workspace on the current monitor
plugin:scroller:gesture_workspace_switch_prefix = m
# swith to a workspace on the current monitor, including empty workspaces
plugin:scroller:gesture_workspace_switch_prefix = r
# swith to the next/previous open workspace
plugin:scroller:gesture_workspace_switch_prefix = e
```


### Options Example

``` conf
plugin {
    scroller {
        column_default_width = onehalf
        focus_wrap = false
        # ultra-wide monitor
        column_widths = onefourth onethird onehalf onesixth
        # portrait mode monitors
        monitor_options = (DP-2 = (mode = row; column_default_width = onehalf; window_default_height = one), HDMI-A-1 = (mode = col; column_default_width = one; window_default_height = onehalf))
    }
}
```


## Window Rules

*hyprscroller* supports a number of static Window Rules v2 that can be triggered
at window creation. [Hyprland's Wiki](https://wiki.hyprland.org/Configuring/Window-Rules/)
explains what Window Rules are, and how to enable and configure them.

These are rules specific to *hyprscroller*

### group

You may want to keep every window of the same class, type etc. in the same
column. For example, in [#45](https://github.com/dawsers/hyprscroller/issues/45),
a user wants to open all the plot windows for a Python script in the same
column.

**Syntax of rule:** `plugin:scroller:group group_name`

```
windowrulev2 = plugin:scroller:group python_plots, class:(python3)
```

### alignwindow

Aligns the new opened window. Works in the same way as the `alignwindow`
dispatcher.

**Syntax of rule:** `plugin:scroller:alignwindow position`

Center any new Firefox window.

```
windowrulev2 = plugin:scroller:alignwindow center, class:(firefox)
```

### marksadd

Add a named mark to a window.

**Syntax of rule:** `plugin:scroller:marksadd name`

Add a mark named `m` to Thunderbird's main window as soon as it's opened. This
will let you navigate to Thunderbird from wherever you are by using a
`marksvisit` key binding. I use `Super + ' + m` to set the focus on Thunderbird.

```
windowrulev2 = plugin:scroller:marksadd m, class:(thunderbird),title:(Mozilla Thunderbird)$
```

### columnwidth

Overrides the default width of the column containing the window.

**Syntax of rule:** `plugin:scroller:columnwidth width`

Open any Firefox window with a width of `twothirds`

```
windowrulev2 = plugin:scroller:columnwidth twothirds, class:(firefox)
```

### windowheight

Overrides the default height of the new window.

**Syntax of rule:** `plugin:scroller:windowheight height`

Open any Firefox window with a height of `onehalf`

```
windowrulev2 = plugin:scroller:windowheight onehalf, class:(firefox)
```


## Key bindings

As an example, you could set some key bindings in your `hyprland.conf` like this:

``` conf
# Move focus with mainMod + arrow keys
bind = $mainMod, left, movefocus, l
bind = $mainMod, right, movefocus, r
bind = $mainMod, up, movefocus, u
bind = $mainMod, down, movefocus, d
bind = $mainMod, home, scroller:movefocus, begin
bind = $mainMod, end, scroller:movefocus, end

# Movement
bind = $mainMod CTRL, left, movewindow, l
bind = $mainMod CTRL, right, movewindow, r
bind = $mainMod CTRL, up, movewindow, u
bind = $mainMod CTRL, down, movewindow, d
bind = $mainMod CTRL, home, scroller:movewindow, begin
bind = $mainMod CTRL, end, scroller:movewindow, end

# Modes
bind = $mainMod, bracketleft, scroller:setmode, row
bind = $mainMod, bracketright, scroller:setmode, col

# Sizing keys
bind = $mainMod, equal, scroller:cyclesize, next
bind = $mainMod, minus, scroller:cyclesize, prev

# Admit/Expel
bind = $mainMod, I, scroller:admitwindow,
bind = $mainMod, O, scroller:expelwindow,

# Center submap
# will switch to a submap called center
bind = $mainMod, C, submap, center
# will start a submap called "center"
submap = center
# sets repeatable binds for resizing the active window
bind = , C, scroller:alignwindow, c
bind = , C, submap, reset
bind = , right, scroller:alignwindow, r
bind = , right, submap, reset
bind = , left, scroller:alignwindow, l
bind = , left, submap, reset
bind = , up, scroller:alignwindow, u
bind = , up, submap, reset
bind = , down, scroller:alignwindow, d
bind = , down, submap, reset
# use reset to go back to the global submap
bind = , escape, submap, reset
# will reset the submap, meaning end the current one and return to the global one
submap = reset

# Resize submap
# will switch to a submap called resize
bind = $mainMod SHIFT, R, submap, resize
# will start a submap called "resize"
submap = resize
# sets repeatable binds for resizing the active window
binde = , right, resizeactive, 100 0
binde = , left, resizeactive, -100 0
binde = , up, resizeactive, 0 -100
binde = , down, resizeactive, 0 100
# use reset to go back to the global submap
bind = , escape, submap, reset
# will reset the submap, meaning end the current one and return to the global one
submap = reset

# Fit size submap
# will switch to a submap called fitsize
bind = $mainMod, W, submap, fitsize
# will start a submap called "fitsize"
submap = fitsize
# sets binds for fitting columns/windows in the screen
bind = , W, scroller:fitsize, visible
bind = , W, submap, reset
bind = , right, scroller:fitsize, toend
bind = , right, submap, reset
bind = , left, scroller:fitsize, tobeg
bind = , left, submap, reset
bind = , up, scroller:fitsize, active
bind = , up, submap, reset
bind = , down, scroller:fitsize, all
bind = , down, submap, reset
# use reset to go back to the global submap
bind = , escape, submap, reset
# will reset the submap, meaning end the current one and return to the global one
submap = reset

# overview keys
# bind key to toggle overview (normal)
bind = $mainMod, tab, scroller:toggleoverview
bind = ,mouse:275, scroller:toggleoverview

# Marks
bind = $mainMod, M, submap, marksadd
submap = marksadd
bind = , a, scroller:marksadd, a
bind = , a, submap, reset
bind = , b, scroller:marksadd, b
bind = , b, submap, reset
bind = , c, scroller:marksadd, c
bind = , c, submap, reset
bind = , escape, submap, reset
submap = reset

bind = $mainMod SHIFT, M, submap, marksdelete
submap = marksdelete
bind = , a, scroller:marksdelete, a
bind = , a, submap, reset
bind = , b, scroller:marksdelete, b
bind = , b, submap, reset
bind = , c, scroller:marksdelete, c
bind = , c, submap, reset
bind = , escape, submap, reset
submap = reset

bind = $mainMod, apostrophe, submap, marksvisit
submap = marksvisit
bind = , a, scroller:marksvisit, a
bind = , a, submap, reset
bind = , b, scroller:marksvisit, b
bind = , b, submap, reset
bind = , c, scroller:marksvisit, c
bind = , c, submap, reset
bind = , escape, submap, reset
submap = reset

bind = $mainMod CTRL, M, scroller:marksreset

# Pin
bind = $mainMod, P, scroller:pin,

# Window copy/paste
bind = $mainMod, Insert, scroller:selectiontoggle,
bind = $mainMod CTRL, Insert, scroller:selectionreset,
bind = $mainMod SHIFT, Insert, scroller:selectionmove, right

# Trails and Trailmarks
bind = $mainMod SHIFT, semicolon, submap, trail
submap = trail
bind = , bracketright, scroller:trailnext,
bind = , bracketleft, scroller:trailprevious,
bind = , semicolon, scroller:trailnew,
bind = , semicolon, submap, reset
bind = , d, scroller:traildelete,
bind = , d, submap, reset
bind = , c, scroller:trailclear,
bind = , c, submap, reset
bind = , Insert, scroller:trailtoselection,
bind = , Insert, submap, reset
bind = , escape, submap, reset
submap = reset

bind = $mainMod, semicolon, submap, trailmark
submap = trailmark
bind = , bracketright, scroller:trailmarknext,
bind = , bracketleft, scroller:trailmarkprevious,
bind = , semicolon, scroller:trailmarktoggle,
bind = , semicolon, submap, reset
bind = , escape, submap, reset
submap = reset

bind = $mainMod, slash, scroller:jump,
```

