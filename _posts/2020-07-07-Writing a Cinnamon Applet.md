---
title: "Writing a Cinnamon Applet"
categories:
  - Development
tags:
  - development
  - linux
  - tech
comments: true

---

Applets in the Cinnamon Desktop environment allow you to show information on your panel bar and execute actions from there.

They are extremely useful but at the same time there are few of them, so if you'd like to write one yourself you've come to the right place.

## Setting up a basic indicator

First of all head over to:

[https://projects.linuxmint.com/reference/git/cinnamon-tutorials/write-applet.html](https://projects.linuxmint.com/reference/git/cinnamon-tutorials/write-applet.html)

to get a basic understanding of the Applet structure.

Once you've obtained an ```applet.js``` and a ```metadata.json```  it's time to get our hands dirty.

```js
const Applet = imports.ui.applet;
const Util = imports.misc.util;

function MyApplet(orientation, panel_height, instance_id) {
    this._init(orientation, panel_height, instance_id);
}

MyApplet.prototype = {
    __proto__: Applet.IconApplet.prototype,

    _init: function(orientation, panel_height, instance_id) {
        Applet.IconApplet.prototype._init.call(this, orientation, panel_height, instance_id);

        this.set_applet_icon_name("force-exit");
        this.set_applet_tooltip(_("Click here to kill a window"));
    },

    on_applet_clicked: function() {
        Util.spawn('xkill');
    }
};

function main(metadata, orientation, panel_height, instance_id) {
    return new MyApplet(orientation, panel_height, instance_id);
}

```

## Editing the basic example

There are different types of Applets:

| Type | Name |
|------|---------|
| Text only | Applet.TextApplet|
| Icon only | Applet.IconApplet|
| Icon + text | Applet.TextIconApplet|

Depending on your preferences update the ```__proto__``` and the ```__init__``` functions accordingly.

I'll be using a ```TextIconApplet``` for completeness.

Adjust the displayed icon and label in the ```__init__``` function:

```js
// icon
this.set_applet_icon_name("network-vpn");
// tooltip on hover
this.set_applet_tooltip(_("Manage your VPN connection"));
// label
this.set_applet_label("Hello");
```

Remove the click function, as we don't need it.

```js
on_applet_clicked: function() {
    Util.spawn('xkill');
}
```

## Update the label on a loop

Now every good indicator updates its displayed value, to do so add these methods to the ```applet.js```, before closing the ```MyApplet.prototype``` bracket.

```js
on_applet_removed_from_panel: function () {
// stop the loop when the applet is removed
	if (this._updateLoopID) {
		Mainloop.source_remove(this._updateLoopID);
	}

},

_run_cmd: function(command) {
// run a command and return the output
    try {
        let [result, stdout, stderr] = GLib.spawn_command_line_sync(command);
        if (stdout != null) {
            return stdout.toString();
        }
    }
    catch (e) {
        global.logError(e);
    }

    return "";
},


_get_status: function(){
   let status = this._run_cmd("your command to run");
   // update the label with the output of your command
   this.set_applet_label(status);
},

_update_loop: function () {
   this._get_status();
   // run the loop every 5000 ms
   this._updateLoopID = Mainloop.timeout_add(5000, Lang.bind(this, this._update_loop));
},

```

Then import the requred libraries:
```js
const GLib = imports.gi.GLib;
const Mainloop = imports.mainloop;
const Lang = imports.lang;
```


## Dropdown options

A good applet allows you to do more than display a value, so now we'll add dropdown options with commands.

Import:
```js
const PopupMenu = imports.ui.popupMenu;
const St = imports.gi.St;
```

In your ```__init__``` function add:

```js

// Create the popup menu
this.menuManager = new PopupMenu.PopupMenuManager(this);
this.menu = new Applet.AppletPopupMenu(this, orientation);
this.menuManager.addMenu(this.menu);

this._contentSection = new PopupMenu.PopupMenuSection();
this.menu.addMenuItem(this._contentSection);

// First item: Turn on
let item = new PopupMenu.PopupIconMenuItem("Label 1", "icon 1", St.IconType.FULLCOLOR);

item.connect('activate', Lang.bind(this, function() {
               Util.spawnCommandLine("command item 1");
             }));
this.menu.addMenuItem(item);


item = new PopupMenu.PopupIconMenuItem("Label 2", "icon 2", St.IconType.FULLCOLOR);

item.connect('activate', Lang.bind(this, function() {
               Util.spawnCommandLine("command item 2");
             }));
this.menu.addMenuItem(item);

// Second item: Turn off
item = new PopupMenu.PopupIconMenuItem("Label 3", "icon 3", St.IconType.FULLCOLOR);

item.connect('activate', Lang.bind(this, function() {
               Util.spawnCommandLine("command item 3");
             }));
this.menu.addMenuItem(item);

```

And then change the labels, icons and commands accordingly.


## Settings page

A nice feature to have is a settings page to tweak a couple of parameters, in this example we'll update the update frequency of the applet.

Create a new file: ```settings-schema.json```

```json
{

  "update-interval": {
    "type": "spinbutton",
    "default": 5000,
    "min": 2000,
    "max": 30000,
    "step": 100,
    "units": "ms",
    "description": "Update interval"
  }

}
```

Now on the ```applet.js``` import and set the UUID:
```js
const Settings = imports.ui.settings;
const UUID = "yourUUID";
```

In the ```__init__``` method add:

```js
this.settings = new Settings.AppletSettings(this, UUID, this.instance_id);
this.settings.bindProperty(Settings.BindingDirection.IN, "update-interval", "update_interval", this._new_freq, null);
```

Now that we're storing the settings value in ```this.update_interval```, simply modify the loop parameter:

```js
_update_loop: function () {
       this._get_status();
       this._updateLoopID = Mainloop.timeout_add(this.update_interval, Lang.bind(this, this._update_loop));
   },

```

## Final thoughts

When writing my applet I found the lack of extensive documentation a little disorienting.
I hope that this brief tutorial clears some doubts.

Remember that looking at other developer's work is extremely helpful in many cases.

You can find my NordVPN applet here [https://github.com/nickdurante/nordvpn-indicator-cinnamon](https://github.com/nickdurante/nordvpn-indicator-cinnamon)
