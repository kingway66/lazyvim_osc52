# lazyvim_osc52 over ssh
## nvim > 0.11
## 1. 如果只用macos、Linux

```lua title=~/.config/nvim/lua/config/options.lua
vim.opt.clipboard:append("unnamedplus")
```
```lua title=~/.config/nvim/init.lua
if vim.env.SSH_TTY then
  vim.g.clipboard = "osc52" -- the official usage
end
```
## 2. 如果也用windows，能接受用终端ctrl+shift+v来paste，不用nvim的p功能

### 使用系统剪贴板并使用osc52

```lua title=~/.config/nvim/lua/config/options.lua
vim.opt.clipboard:append("unnamedplus")
```

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
## 3. 使用windows，但希望在linux、mac下能用p

```lua title=~/.config/nvim/lua/config/options.lua
vim.opt.clipboard:append("unnamedplus")
```

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
