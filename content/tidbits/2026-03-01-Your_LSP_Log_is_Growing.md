+++
title = "Your LSP Log is Growing"
subtitle = "Grow your log file to 100MB+ with this one simple \"pragmatic shortcut\"" 
date = 2025-03-01

[extra]
subtitle = "Grow your log file to 100MB+ with this one simple \"pragmatic shortcut\"" 

[taxonomies]
tags = ["bugs", "vim", "oss"]
+++

## The Problem

My Neovim LSP log file (`~/.local/share/nvim/lsp.log`) had ballooned to 139MB. I wasn't sure why it was growing so fast, but after inspecting with `:LspLog`, I saw 300,000+ `[ERROR]` entries. ~97% of these were from [terraform-ls](https://github.com/hashicorp/terraform-ls).

Now, I do work with a "Terralith"[^1] at work, but I am not sure that we have _that_ many Terraform LSP errors. We do enforce `tflint` and the works.

## Root Cause

This is a known upstream bug in terraform-ls: [hashicorp/terraform-ls#1271](https://github.com/hashicorp/terraform-ls/issues/1271)

`terraform-ls` writes ALL log messages (INFO, DEBUG, etc.) to stderr instead of using the LSP-standard `window/logMessage` notification. Neovim's LSP client correctly treats stderr as errors, so every `terraform-ls` log line becomes an `[ERROR]` entry in your `lsp.log`.

The issue has been open since May 2023 - it was a "pragmatic shortcut" taken early in development that never got fixed.

## The Fix

Until HashiCorp fixes this upstream, periodic cleanup is the workaround. I added a command to nuke the log from within Neovim:

```lua
-- lsp log management
-- Clears the LSP log file (useful due to terraform-ls stderr bug: https://github.com/hashicorp/terraform-ls/issues/1271)
vim.api.nvim_create_user_command('LspLogNuke', function()
    local log_path = vim.lsp.get_log_path()
    local file = io.open(log_path, 'w')
    if file then
        file:close()
        vim.notify('Nuked LSP log: ' .. log_path, vim.log.levels.INFO)
    else
        vim.notify('Failed to nuke LSP log: ' .. log_path, vim.log.levels.ERROR)
    end
end, { desc = 'Clear the LSP log file' })
```

Now `:LspLogNuke` clears the log when it gets out of hand.

## Conclusion

I'd like to see this fixed upstream of course, this _does_ feel like a misuse of the LSP log. For now, I'm happy with this workaround. See my commit [here](https://github.com/quantike/dotfiles/commit/f80cfa761209900bbeddde4195fd965cdd093da0) where I had also changed my Oh-My-ZSH theme to half-life. Serendipitous.

[^1]: A Terralith is a portmanteau of Terraform and Monolith. It's also a [Minecraft mod](https://modrinth.com/datapack/terralith).
