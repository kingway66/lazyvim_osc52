# lazyvim neovim tmux alacritty osc52 over ssh

网上很多相关信息过时了，总结了前人智慧，记录在此。如有错误还请海涵。
A lot of information about nvim and osc52 is outdated, and the wisdom of predecessors is summarized and recorded here. 
If there is an error, please also Haihan.

## 需要的环境The environment that is needed
* nvim > 0.11 
* lazyvim startup > 2025.06
* tmux 3.5a
* alacritty 0.15.1

## 1. 如果只用macos、Linux，这就很好办了If you only use macOS and Linux, this is very easy

~/.config/nvim/lua/config/options.lua
```lua title=~/.config/nvim/lua/config/options.lua
vim.opt.clipboard:append("unnamedplus")
```

~/.config/nvim/init.lua
```lua title=~/.config/nvim/init.lua
if vim.env.SSH_TTY then
  vim.g.clipboard = "osc52" -- the official usage
end
```

## 2. 如果主要用windows，希望在全平台用终端ctrl+shift+v来paste，不用nvim的p功能If you mainly use Windows, you want to use the terminal Ctrl+Shift+V to paste on all platforms, instead of the P function of nvim

~/.config/nvim/lua/config/options.lua
```lua title=~/.config/nvim/lua/config/options.lua
vim.opt.clipboard:append("unnamedplus")
```
~/.config/nvim/init.lua
```lua title=~/.config/nvim/init.lua
local function my_paste(reg)
  return function(lines)
    local content = vim.fn.getreg('"')
    return vim.split(content, "\n")
  end
end

if vim.env.SSH_TTY then
  vim.g.clipboard = {
    name = "OSC 52",
    copy = {
      ["+"] = require("vim.ui.clipboard.osc52").copy("+"),
      ["*"] = require("vim.ui.clipboard.osc52").copy("*"),
    },
    paste = {
      ["+"] = my_paste("+"),
      ["*"] = my_paste("*"),
    },
  }
end

```

## 3. [recommended]备用windows，希望在linux、mac下能用p。Occasionally work on Windows, hope to be able to use P under Linux, Mac

~/.config/nvim/lua/config/options.lua
```lua title=~/.config/nvim/lua/config/options.lua
vim.opt.clipboard:append("unnamedplus")
```

~/.config/nvim/init.lua
```lua title=~/.config/nvim/init.lua
local function my_paste(reg)
  return function(lines)
    local content = vim.fn.getreg('"')
    return vim.split(content, "\n")
  end
end

if vim.env.SSH_TTY then
  if vim.env.SSH_CLIENT_OS == "Windows_NT" then
    vim.g.clipboard = {
      name = "OSC 52",
      copy = {
        ["+"] = require("vim.ui.clipboard.osc52").copy("+"),
        ["*"] = require("vim.ui.clipboard.osc52").copy("*"),
      },
      paste = {
        ["+"] = my_paste("+"),
        ["*"] = my_paste("*"),
      },
    }
  else
    vim.g.clipboard = "osc52" -- the official usage
  end
end
```

/etc/ssh/sshd_config # add the line and restart sshd or reboot
```
AcceptEnv SSH_CLIENT_OS
```

on client windows

c:/Users/yourusername/.ssh/config
```
Host *
    SendEnv SSH_CLIENT_OS
```

c:\app\sshgo.bat
```bat
@echo off

set SSH_CLIENT_OS=%OS%

echo starting_with: %SSH_CLIENT_OS%

:: ssh %*
ssh -p 22 username@hostname.ip
```
go the alacritty and start ssh with this bat file


## 4. the alacritty osc52 support and vi mode

~/.config/alacritty/alacritty.toml
```toml
[terminal]
osc52 = "CopyPaste" # or CopyOnly on default
```

进入vi模式默认的快捷键是ctrl+shift+space，在linux和windows下可以直接用
The default shortcut key to enter VI mode is Ctrl+Shift+Space, which can be used directly under Linux and Windows

在mac下我用着不行，改成下面的组合可以用
I can't use it under Mac, so I can use it by changing it to the following combination

```toml
[keyboard]
bindings = [
  { key = "I", mods = "Command|Option", action = "ToggleViMode" },
]
```

使用步骤：
   1. 按下command+option+i（或ctrl+shift+space）启动vi模式
   2. 和vi一样，用v或V启动选择
   3. 选好后，按y进行复制
   4. 到任何地方去粘贴（p或ctrl+v,command+v等等）
   5. 要退出vi模式，再次按下启动组合键即可

Directions of use:

   1. Press Command+Option+I (or Ctrl+Shift+Space) to start VI mode
   2. As with VI, start the selection with v or V
   3. Once selected, press y to copy
   4. Paste anywhere (p or ctrl+v, command+v, etc.)
   5. To exit VI mode, press the start combo key again

## 5. the tmux vi mode

tmux 3.5a，好像并不需要配置，直接就支持osc52
我设置了两项是支持鼠标和vi快捷键
it doesn't seem to need to be configured, it directly supports OSC52 
I've set two items to support mouse and vi shortcuts

~/.tmux.conf
```
set-option -g mouse on 
setw -g mode-keys vi
```

使用步骤
   1. 按下ctrl+b，然后按[，右上角会出现进入vi模式的行号提示
   2. 移动光标到想复制的地方，按下space（相当于vi里按下v），出现黄色选择
   3. 或者按下大V，可以复制行
   4. 选择完成后按下enter
   5. 到任何地方去粘贴（在tmux里可以用ctrl+b,]或者ctrl+v,command+v）

Directions:

   1. Press Ctrl+B, and then press [, and a line number prompt will appear in the upper right corner to enter VI mode
   2. Move the cursor to the place you want to copy, press Space (equivalent to v in VI), and a yellow selection will appear
   3. Or press the big V to copy the line
   4. Press Enter when the selection is complete
   5. Paste anywhere (CTRL+B,] or ctrl+v/command+v in TMUX)

## 6. tested platform
   1. linux(client alacritty) ssh to mac(sshd server,homebrew nvim/tmux)
   2. mac(client alacritty) to linux(sshd server)
   3. windows(client alacritty) to mac or linux
   4. [failed] ios(client prompt3) to mac or linux（用prompt3设置允许存取剪贴板，直接字符串echo测试或tmux测试成功，nvim测试失败，原因不明）

## 7. special thanks 

* [穿透 wsl 和 ssh, 新版本 neovim 跨设备任意复制，copy anywhere!](https://www.cnblogs.com/sxrhhh/p/18234652/neovim-copy-anywhere)
* [Alacritty 终端配置指南](https://www.kevnu.com/zh/posts/18)



