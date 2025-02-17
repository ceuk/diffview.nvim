# Diffview.nvim

Single tabpage interface for easily cycling through diffs for all modified files
for any git rev.

![preview](https://user-images.githubusercontent.com/2786478/131269942-e34100dd-cbb9-48fe-af31-6e518ce06e9e.png)


## Introduction

Vim's diff mode is pretty good, but there is no convenient way to quickly bring
up all modified files in a diffsplit. This plugin aims to provide a simple,
unified, single tabpage interface that lets you easily review all changed files
for any git rev.

## Requirements

- Git ≥ 2.31.0
- Neovim ≥ 0.7.0
- [plenary.nvim](https://github.com/nvim-lua/plenary.nvim)
- [nvim-web-devicons](https://github.com/kyazdani42/nvim-web-devicons) (optional) For file icons

## Installation

Install the plugin with your package manager of choice.

```vim
" Plug
Plug 'nvim-lua/plenary.nvim'
Plug 'sindrets/diffview.nvim'
```

```lua
-- Packer
use { 'sindrets/diffview.nvim', requires = 'nvim-lua/plenary.nvim' }
```

## Merge Tool

![merge tool showcase](https://user-images.githubusercontent.com/2786478/188286293-13bbf0ab-3595-425d-ba4a-12f514c17eb6.png)

Opening a diff view during a merge or a rebase will list the conflicted files in
their own section. When opening a conflicted file, it will open in a 3-way diff
allowing you to resolve the merge conflicts with the context of the target
branch's version, as well as the version from the branch which is being merged.

The 3-way diff is only the default layout for merge conflicts. There are
multiple variations on this layout, a 4-way diff layout, and a single window
layout available.

In addition to the normal `:h copy-diffs` mappings, there are default mappings
provided for jumping between conflict markers, obtaining a hunk directly from
any of the diff buffers, and accepting any one, all, or none of the versions of
a file given by a conflict region.

For more information on the merge tool, mappings, layouts and how to
configure them, see:

- `:h diffview-merge-tool`
- `:h diffview-config-view.x.layout`

## File History

![file history showcase](https://user-images.githubusercontent.com/2786478/188331057-f9ec9a0d-8cda-4ff8-ac98-febcc7aa4010.png)

The file history view allows you to list all the commits that affected a given
set of paths, and view the changes made in a diff split. This is a porcelain
interface for git-log, and supports a good number of its options. Things like:

- Filtering commits by grepping commit messages and commit authors.
- Tracing the line evolution of a given set of line ranges for multiple files. 
- Only listing changes for a specific commit range, branch, or tag.
- Following file changes through renames.

Get started by opening file history for:

- The current branch: `:DiffviewFileHistory`
- The current file: `:DiffviewFileHistory %`

For more info, see `:h :DiffviewFileHistory`.

## Usage

### `:DiffviewOpen [git rev] [options] [ -- {paths...}]`

Calling `:DiffviewOpen` with no args opens a new Diffview that compares against
the current index. You can also provide any valid git rev to view only changes
for that rev.

Examples:

- `:DiffviewOpen`
- `:DiffviewOpen HEAD~2`
- `:DiffviewOpen HEAD~4..HEAD~2`
- `:DiffviewOpen d4a7b0d`
- `:DiffviewOpen d4a7b0d^!`
- `:DiffviewOpen d4a7b0d..519b30e`
- `:DiffviewOpen origin/main...HEAD`

You can also provide additional paths to narrow down what files are shown:

- `:DiffviewOpen HEAD~2 -- lua/diffview plugin`

For information about additional `[options]`, visit the
[documentation](https://github.com/sindrets/diffview.nvim/blob/main/doc/diffview.txt).

Additional commands for convenience:

- `:DiffviewClose`: Close the current diffview. You can also use `:tabclose`.
- `:DiffviewToggleFiles`: Toggle the file panel.
- `:DiffviewFocusFiles`: Bring focus to the file panel.
- `:DiffviewRefresh`: Update stats and entries in the file list of the current
  Diffview.

With a Diffview open and the default key bindings, you can cycle through changed
files with `<tab>` and `<s-tab>` (see configuration to change the key bindings).

### `:[range]DiffviewFileHistory [paths] [options]`

Opens a new file history view that lists all commits that affected the given
paths. This is a porcelain interface for git-log. Both `[paths]` and
`[options]` may be specified in any order, even interchangeably.

If no `[paths]` are given, defaults to the top-level of the working tree. The
top-level will be inferred from the current buffer when possible, otherwise the
cwd is used. Multiple `[paths]` may be provided and git pathspec is supported.

If `[range]` is given, the file history view will trace the line evolution of the
given range in the current file (for more info, see the `-L` flag in the docs).

Examples:

- `:DiffviewFileHistory`
- `:DiffviewFileHistory %`
- `:DiffviewFileHistory path/to/some/file.txt`
- `:DiffviewFileHistory path/to/some/directory`
- `:DiffviewFileHistory include/this and/this :!but/not/this`
- `:DiffviewFileHistory --range=origin..HEAD`
- `:DiffviewFileHistory --range=feat/example-branch`
- `:'<,'>DiffviewFileHistory`

## Configuration

<p>
<details>
<summary style='cursor: pointer'><b>Example config with default values</b></summary>

```lua
-- Lua
local actions = require("diffview.actions")

require("diffview").setup({
  diff_binaries = false,    -- Show diffs for binaries
  enhanced_diff_hl = false, -- See ':h diffview-config-enhanced_diff_hl'
  git_cmd = { "git" },      -- The git executable followed by default args.
  use_icons = true,         -- Requires nvim-web-devicons
  icons = {                 -- Only applies when use_icons is true.
    folder_closed = "",
    folder_open = "",
  },
  signs = {
    fold_closed = "",
    fold_open = "",
    done = "✓",
  },
  view = {
    -- Configure the layout and behavior of different types of views.
    -- Available layouts: 
    --  'diff1_plain'
    --    |'diff2_horizontal'
    --    |'diff2_vertical'
    --    |'diff3_horizontal'
    --    |'diff3_vertical'
    --    |'diff3_mixed'
    --    |'diff4_mixed'
    -- For more info, see ':h diffview-config-view.x.layout'.
    default = {
      -- Config for changed files, and staged files in diff views.
      layout = "diff2_horizontal",
    },
    merge_tool = {
      -- Config for conflicted files in diff views during a merge or rebase.
      layout = "diff3_horizontal",
      disable_diagnostics = true,   -- Temporarily disable diagnostics for conflict buffers while in the view.
    },
    file_history = {
      -- Config for changed files in file history views.
      layout = "diff2_horizontal",
    },
  },
  file_panel = {
    listing_style = "tree",             -- One of 'list' or 'tree'
    tree_options = {                    -- Only applies when listing_style is 'tree'
      flatten_dirs = true,              -- Flatten dirs that only contain one single dir
      folder_statuses = "only_folded",  -- One of 'never', 'only_folded' or 'always'.
    },
    win_config = {                      -- See ':h diffview-config-win_config'
      position = "left",
      width = 35,
      win_opts = {}
    },
  },
  file_history_panel = {
    log_options = {   -- See ':h diffview-config-log_options'
      single_file = {
        diff_merges = "combined",
      },
      multi_file = {
        diff_merges = "first-parent",
      },
    },
    win_config = {    -- See ':h diffview-config-win_config'
      position = "bottom",
      height = 16,
      win_opts = {}
    },
  },
  commit_log_panel = {
    win_config = {   -- See ':h diffview-config-win_config'
      win_opts = {},
    }
  },
  default_args = {    -- Default args prepended to the arg-list for the listed commands
    DiffviewOpen = {},
    DiffviewFileHistory = {},
  },
  hooks = {},         -- See ':h diffview-config-hooks'
  keymaps = {
    disable_defaults = false, -- Disable the default keymaps
    view = {
      -- The `view` bindings are active in the diff buffers, only when the current
      -- tabpage is a Diffview.
      ["<tab>"]      = actions.select_next_entry,         -- Open the diff for the next file
      ["<s-tab>"]    = actions.select_prev_entry,         -- Open the diff for the previous file
      ["gf"]         = actions.goto_file,                 -- Open the file in a new split in the previous tabpage
      ["<C-w><C-f>"] = actions.goto_file_split,           -- Open the file in a new split
      ["<C-w>gf"]    = actions.goto_file_tab,             -- Open the file in a new tabpage
      ["<leader>e"]  = actions.focus_files,               -- Bring focus to the file panel
      ["<leader>b"]  = actions.toggle_files,              -- Toggle the file panel.
      ["g<C-x>"]     = actions.cycle_layout,              -- Cycle through available layouts.
      ["[x"]         = actions.prev_conflict,             -- In the merge_tool: jump to the previous conflict
      ["]x"]         = actions.next_conflict,             -- In the merge_tool: jump to the next conflict
      ["<leader>co"] = actions.conflict_choose("ours"),   -- Choose the OURS version of a conflict
      ["<leader>ct"] = actions.conflict_choose("theirs"), -- Choose the THEIRS version of a conflict
      ["<leader>cb"] = actions.conflict_choose("base"),   -- Choose the BASE version of a conflict
      ["<leader>ca"] = actions.conflict_choose("all"),    -- Choose all the versions of a conflict
      ["dx"]         = actions.conflict_choose("none"),   -- Delete the conflict region
    },
    diff1 = { --[[ Mappings in single window diff layouts ]] },
    diff2 = { --[[ Mappings in 2-way diff layouts ]] },
    diff3 = {
      -- Mappings in 3-way diff layouts
      { { "n", "x" }, "2do", actions.diffget("ours") },   -- Obtain the diff hunk from the OURS version of the file
      { { "n", "x" }, "3do", actions.diffget("theirs") }, -- Obtain the diff hunk from the THEIRS version of the file
    },
    diff4 = {
      -- Mappings in 4-way diff layouts
      { { "n", "x" }, "1do", actions.diffget("base") },   -- Obtain the diff hunk from the BASE version of the file
      { { "n", "x" }, "2do", actions.diffget("ours") },   -- Obtain the diff hunk from the OURS version of the file
      { { "n", "x" }, "3do", actions.diffget("theirs") }, -- Obtain the diff hunk from the THEIRS version of the file
    },
    file_panel = {
      ["j"]             = actions.next_entry,         -- Bring the cursor to the next file entry
      ["<down>"]        = actions.next_entry,
      ["k"]             = actions.prev_entry,         -- Bring the cursor to the previous file entry.
      ["<up>"]          = actions.prev_entry,
      ["<cr>"]          = actions.select_entry,       -- Open the diff for the selected entry.
      ["o"]             = actions.select_entry,
      ["<2-LeftMouse>"] = actions.select_entry,
      ["-"]             = actions.toggle_stage_entry, -- Stage / unstage the selected entry.
      ["S"]             = actions.stage_all,          -- Stage all entries.
      ["U"]             = actions.unstage_all,        -- Unstage all entries.
      ["X"]             = actions.restore_entry,      -- Restore entry to the state on the left side.
      ["R"]             = actions.refresh_files,      -- Update stats and entries in the file list.
      ["L"]             = actions.open_commit_log,    -- Open the commit log panel.
      ["<c-b>"]         = actions.scroll_view(-0.25), -- Scroll the view up
      ["<c-f>"]         = actions.scroll_view(0.25),  -- Scroll the view down
      ["<tab>"]         = actions.select_next_entry,
      ["<s-tab>"]       = actions.select_prev_entry,
      ["gf"]            = actions.goto_file,
      ["<C-w><C-f>"]    = actions.goto_file_split,
      ["<C-w>gf"]       = actions.goto_file_tab,
      ["i"]             = actions.listing_style,        -- Toggle between 'list' and 'tree' views
      ["f"]             = actions.toggle_flatten_dirs,  -- Flatten empty subdirectories in tree listing style.
      ["<leader>e"]     = actions.focus_files,
      ["<leader>b"]     = actions.toggle_files,
      ["g<C-x>"]        = actions.cycle_layout,
      ["[x"]            = actions.prev_conflict,
      ["]x"]            = actions.next_conflict,
    },
    file_history_panel = {
      ["g!"]            = actions.options,          -- Open the option panel
      ["<C-A-d>"]       = actions.open_in_diffview, -- Open the entry under the cursor in a diffview
      ["y"]             = actions.copy_hash,        -- Copy the commit hash of the entry under the cursor
      ["L"]             = actions.open_commit_log,
      ["zR"]            = actions.open_all_folds,
      ["zM"]            = actions.close_all_folds,
      ["j"]             = actions.next_entry,
      ["<down>"]        = actions.next_entry,
      ["k"]             = actions.prev_entry,
      ["<up>"]          = actions.prev_entry,
      ["<cr>"]          = actions.select_entry,
      ["o"]             = actions.select_entry,
      ["<2-LeftMouse>"] = actions.select_entry,
      ["<c-b>"]         = actions.scroll_view(-0.25),
      ["<c-f>"]         = actions.scroll_view(0.25),
      ["<tab>"]         = actions.select_next_entry,
      ["<s-tab>"]       = actions.select_prev_entry,
      ["gf"]            = actions.goto_file,
      ["<C-w><C-f>"]    = actions.goto_file_split,
      ["<C-w>gf"]       = actions.goto_file_tab,
      ["<leader>e"]     = actions.focus_files,
      ["<leader>b"]     = actions.toggle_files,
      ["g<C-x>"]        = actions.cycle_layout,
    },
    option_panel = {
      ["<tab>"] = actions.select_entry,
      ["q"]     = actions.close,
    },
  },
})
```

</details>
</p>

### Hooks

The `hooks` table allows you to define callbacks for various events emitted from
Diffview. The available hooks are documented in detail in
`:h diffview-config-hooks`. The hook events are also available as User
autocommands. See `:h diffview-user-autocmds` for more details.

Examples:

```lua
hooks = {
  diff_buf_read = function(bufnr)
    -- Change local options in diff buffers
    vim.opt_local.wrap = false
    vim.opt_local.list = false
    vim.opt_local.colorcolumn = { 80 }
  end,
  view_opened = function(view)
    print(
      ("A new %s was opened on tab page %d!")
      :format(view:class():name(), view.tabpage)
    )
  end,
}
```

### Keymaps

The keymaps config is structured as a table with sub-tables for various
different contexts where mappings can be declared. In these sub-tables
key-value pairs are treated as the `{lhs}` and `{rhs}` of a normal mode
mapping. These mappings all use the `:map-arguments` `silent`, `nowait`, and
`noremap`. The implementation uses `vim.keymap.set()`, so the `{rhs}` can be
either a vim command in the form of a string, or it can be a lua function:

```lua
  view = {
    -- Vim command:
    ["a"] = "<Cmd>echom 'foo'<CR>",
    -- Lua function:
    ["b"] = function() print("bar") end,
  }
```

To disable any single mapping without disabling them all, set its value to
`false`:

```lua
  view = {
    -- Disable the default mapping for <tab>:
    ["<tab>"] = false,
  }
```

Most of the mapped file panel actions also work from the view if they are added
to the view maps (and vice versa). The exception is for actions that only
really make sense specifically in the file panel, such as `next_entry`,
`prev_entry`. Actions such as `toggle_stage_entry` and `restore_entry` work
just fine from the view. When invoked from the view, these will target the file
currently open in the view rather than the file under the cursor in the file
panel.

**For more details on how to set mappings for other modes, actions, and more see:**
- `:h diffview-config-keymaps`
- `:h diffview-actions`

## Restoring Files

If the right side of the diff is showing the local state of a file, you can
restore the file to the state from the left side of the diff (key binding `X`
from the file panel by default). The current state of the file is stored in the
git object database, and a command is echoed that shows how to undo the change.

## Tips and FAQ

- **Hide untracked files:**
  - `DiffviewOpen -uno`
- **Exclude certain paths:**
  - `DiffviewOpen -- :!exclude/this :!and/this`
- **Run as if git was started in a specific directory:**
  - `DiffviewOpen -C/foo/bar/baz`
- **Diff the index against a git rev:**
  - `DiffviewOpen HEAD~2 --cached`
  - Defaults to `HEAD` if no rev is given.
- **Q: How do I get the diagonal lines in place of deleted lines in
  diff-mode?**
  - A: Change your `:h 'fillchars'`:
    - (vimscript): `set fillchars+=diff:╱`
  - Note: whether or not the diagonal lines will line up nicely will depend on
    your terminal emulator. The terminal used in the screenshots is Kitty.
