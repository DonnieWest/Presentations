---
title: Building your first Neovim plugin in Javascript
author: Donnie West
patat:
    wrap: true
---

# The Dark Times - VIM Plugins

1. Plugins were introduced in version 6.0 back in 2001
  a. Vimscript
  b. Originally referred to one file
  c. Files had to be added to:
    i. `~/.vim/plugins`
    ii. `~/.vim/ftdetect`
    iii. `~/.vim/ftplugin`
    iv. `~/.vim/compiler`
    v. `~/.vim/autoload`
    vi. `~/.vim/doc`
  b. Vimball
    i. One packaged tar.gz package
    ii. Sourced at runtime via
    ```
    vim VIMBALL.vba
    :source %
    ```

---

# A New Hope

Tim Pope's [Pathogen](https://github.com/tpope/vim-pathogen)

1. Plugins could be installed from git
2. Used the old VIM Plugin system
    a. Same file structure as old plugins
    b. No real config changes
    c. Simply setup vim's runtimepath for each plugin
3. Could be backed up using git submodules
4. Birthed a new wave of VIM plugin management based on configs
    a. [Vundle](https://github.com/VundleVim/Vundle.vim)
    b. [Neobundle](https://github.com/Shougo/neobundle.vim)
    c. [Dein.nvim](https://github.com/Shougo/dein.vim)
    d. [Vim-Addon-Manager](https://github.com/MarcWeber/vim-addon-manager)
    e. [vim-plug](https://github.com/junegunn/vim-plug)
5. Nothing yet that has a "shrinkwrap" or "lock" capability :(


---
# Other languages

1. Originally VIM plugins could be written in:
  a. Vimscript
  b. Ruby
  c. Python
  d. Lua
2. The language had to be compiled into the VIM editor, otherwise you might miss the functionality
  a. Forced you to compile your own editor to get certain plugins to work - Ubuntu lacked a package with proper lua support
  b. Plugins still needed some residual vimscript to work, causing some people to develop wrappers to code as much as possible in their language of choice
  c. No Javascript support :(
  d. Other crazy languages could be added by building on top of supported languages (YouCompleteMe went from VIM -> Python -> Native code)
3. Plugins also blocked the editor from continuing and were not asynchronous
4. Often difficult to use the language's package system, leaving you stuck with whatever they had included
5. Impossible to add arbitrary languages to VIM\

___

# The plugins directory

The following are ripped directly from Vimscript the Hard Way by Steve Losh in [Chapter 42](http://learnvimscriptthehardway.stevelosh.com/chapters/42.html)

# ~/.vim/colors/

Files inside ~/.vim/colors/ are treated as color schemes. For example: if you run :color mycolors Vim will look for a file at ~/.vim/colors/mycolors.vim and run it. That file should contain all the Vimscript commands necessary to generate your color scheme.

---

# ~/.vim/plugin/

Files inside ~/.vim/plugin/ will each be run once every time Vim starts. These files are meant to contain code that you always want loaded whenever you start Vim.

---

# ~/.vim/ftdetect/

Any files in ~/.vim/ftdetect/ will also be run every time you start Vim.

ftdetect stands for "filetype detection". The files in this directory should set up autocommands that detect and set the filetype of files, and nothing else. This means they should never be more than one or two lines long.

---

# ~/.vim/ftplugin/

Files in ~/.vim/ftplugin/ are different.

The naming of these files matters! When Vim sets a buffer's filetype to a value it then looks for a file in ~/.vim/ftplugin/ that matches. For example: if you run set filetype=derp Vim will look for ~/.vim/ftplugin/derp.vim. If that file exists, it will run it.

Vim also supports directories inside ~/.vim/ftplugin/. To continue our example: set filetype=derp will also make Vim run any and all *.vim files inside ~/.vim/ftplugin/derp/. This lets you split up your plugin's ftplugin files into logical groups.

Because these files are run every time a buffer's filetype is set they must only set buffer-local options! If they set options globally they would overwrite them for all open buffers!

---

# ~/.vim/indent/

Files in ~/.vim/indent/ are a lot like ftplugin files. They get loaded based on their names.

indent files should set options related to indentation for their filetypes, and those options should be buffer-local.

Yes, you could simply put this code in the ftplugin files, but it's better to separate it out so other Vim users will understand what you're doing. It's just a convention, but please be a considerate plugin author and follow it.

___

# ~/.vim/compiler/

Files in ~/.vim/compiler/ work exactly like indent files. They should set compiler-related options in the current buffer based on their names.

Don't worry about what "compiler-related options" means right now. We'll cover that later.

___

# ~/.vim/after/

The ~/.vim/after/ directory is a bit of a hack. Files in this directory will be loaded every time Vim starts, but after the files in ~/.vim/plugin/.

This allows you to override Vim's internal files. In practice you'll rarely need this, so don't worry about it until you find yourself thinking "Vim itself sets option x, but I want something different".

___

# ~/.vim/autoload/

The ~/.vim/autoload/ directory is an incredibly important hack. It sounds a lot more complicated than it actually is.

In a nutshell: autoload is a way to delay the loading of your plugin's code until it's actually needed. We'll cover this in more detail later when we refactor our plugin's code to take advantage of it.

___

# ~/.vim/doc/

Finally, the ~/.vim/doc/ directory is where you can add documentation for your plugin. Vim has a huge focus on documentation (as evidenced by all the :help commands we've been running) so it's important to document your plugins.

---

# Neovim to the rescue

1. Best Neovim feature: Asynchronous plugins
  a. Arbitrary languages available via "Hosts"
    i. Don't have to compile VIM with them
    ii. Backwards compatible with former synchronous plugins
    iii. client -> server architecture via RPC
  b. Non-blocking API
  c. New plugin system
    i. "Job control" in normal Vimscript
2. This feature is later implemented in VIM 8.0, but only the Job control portion for Vimscript. No arbitrary languages or "Hosts"
3. Newer plugin system isn't available for VIM

---

# Neovim plugin structure

`~/.vim/rplugin/${HOST}/myPlugin`

After placing a file (or folder) in the host rplugin folder and having the host installed, simply run `:UpdateRemotePlugins` for Neovim to recognize

That's it.

`pip install neovim` is an example of a host installed this way, with [VimStudio](https://github.com/DonnieWest/VimStudio) being an example

---

# Neovim and Javascript

1. [Node-Host](https://github.com/neovim/node-host)
  a. Build on top of node-client
  b. Plugin like anything else
  c. API can be found in `node-host/node_modules/node-client/index.d.ts`

---

# Let's build one ourselves

```bash

mkdir ~/Code/CSSBeautifier.nvim
cd ~/Code/CSSBeautifier.nvim
npm init
npm install --save cssbeautify

```


in `./rplugin/node/`

```javascript

'use strict'

let beautify = require('cssbeautify')

plugin.commandSync('BeautifyCSS', ( nvim, cb ) => {
  nvim.getCurrentBuffer( ( err, buf ) => {
    buf.getLineSlice( 0, -1, true, true, ( err, lines ) => {
      let beautified = beautify( lines.join('\n') )
      buf.setLineSlice( 0, -1, true, true, beautified.split('\n'), cb )
    })
  })
})

```

And then point your plugin manager at it. With vim-plug in init.vim

```
Plug '~/Code/CSSBeautifier.nvim'

```

and run

```
:UpdateRemotePlugins
```

Open a CSS file and run `:BeautifyCSS`

---


JSBeautifier in AtOM
https://github.com/jdc0589/jsformat-atom/blob/master/lib/format.coffee

Basic guidlines for plugins
http://stevelosh.com/blog/2011/09/writing-vim-plugins/

Vimscript the Hard Way
http://learnvimscriptthehardway.stevelosh.com/
