+++
title = "DuckDB Queries from the Comfort of Your Editor"
date = 2026-07-20

[taxonomies]
tags = ["vim", "duckdb", "sql"]
+++

## The Problem

I use [DuckDB](https://duckdb.org/) basically every day. It's a swiss army knife for analytics work and helps me answer questions across the gamut of data sources my company uses (S3, Postgres, etc.). One issue I run into as a terminal-native developer is unweildy queries in the DuckDB CLI being odd to edit since there is not native Vim support.

## The Fix

There's actually an [external editor mode](https://duckdb.org/docs/lts/clients/cli/editing#external-editor-mode) that can help with this. This can be ran easily with `.edit` or `\e`, opening up a tempfile with the previous query loaded in your `$EDITOR` (Neovim, BTW) named something to the effect of `duckdb.editor.NUMBER.sql`. When you `:wq` DuckDB will then automatically execute your query.

Later, if I want to `Ctrl + R` that query, I want it to be formatted. I set up [sqlfluff](https://www.sqlfluff.com/) with format-on-save with [conform.nvim](https://github.com/stevearc/conform.nvim). When using SQLFluff it needs the user to define a SQL dialect via `--dialect` (see [Dialects Reference](https://docs.sqlfluff.com/en/stable/reference/dialects.html)). Just one issue, I don't use DuckDB exclusively. We use PostgreSQL for all our database migrations and DuckDB is more of an ad-hoc workflow making it a poor default. So, here's what I set up in my NeoVim config:

```lua
return {
	"stevearc/conform.nvim",
	config = function()
		require("conform").setup({
			formatters_by_ft = {
				lua = { "stylua" },
				python = { "isort", "ruff" },
				rust = { "rustfmt" },
				json = { "prettier" },
				markdown = { "prettier" },
				sql = function(bufnr)
					local name = vim.api.nvim_buf_get_name(bufnr)
					-- DuckDB spawns editor buffers named duckdb.edit.NUMBER.sql via \e
					if name:match("duckdb%.edit%.%d+%.sql$") then
						return { "sqlfluff_duckdb" }
					end
					return { "sqlfluff_postgres" }
				end,
			},
			formatters = {
				sqlfluff_postgres = {
					command = "sqlfluff",
					args = { "format", "--dialect", "postgres", "-" },
				},
				sqlfluff_duckdb = {
					command = "sqlfluff",
					args = { "format", "--dialect", "duckdb", "-" },
				},
			},
			format_on_save = {
				timeout_ms = 500,
				lsp_format = "fallback",
			},
		})

		vim.api.nvim_create_autocmd("BufWritePre", {
			pattern = "*",
			callback = function(args)
				require("conform").format({ bufnr = args.buf })
			end,
		})
	end,
}
```

This checks if our buffer is named `duckdb.edit.NUMBER.sql`, and if yes, applies `--dialect duckdb`, otherwise `--dialect postgres`.

## Conclusion

Now when I need to edit a query in DuckDB I know to just use `.edit` or `\e` _and_ I will get a nicely formatted query. This will also format our Postgres with SQLFluff prior to a CI push, which is always a time save.
