Vim(NeoVim)でScalaを書きたいなと思ったので、手順をまとめておく。少し大変だった。

# 各種用語

| 用語   | 意味                                                                                                                         |
| ------ | ---------------------------------------------------------------------------------------------------------------------------- |
| Metals | Scalaの言語サーバー。[公式ページ](https://scalameta.org/metals/)には「Scala language server with rich IDE features」とある。 |


# Dockerコンテナの起動
今回はUbuntu 22.04イメージを使用する。

```
docker run --name vim-scala -it ubuntu:22.04 bash
```

# Neovimのインストール
Vimで設定する情報があまりなかったので、Neovimでやることにする。時代はNeoVimなのかしら。

```
apt update && apt upgrade
apt install neovim
```

正常にインストールされたかを確認

```
root@be0a823e5999:/# nvim --version
NVIM v0.6.1
Build type: Release
LuaJIT 2.1.0-beta3
Compiled by team+vim@tracker.debian.org

Features: +acl +iconv +tui
See ":help feature-compile"

   system vimrc file: "$VIM/sysinit.vim"
  fall-back for $VIM: "/usr/share/nvim"

Run :checkhealth for more info
```

バージョンが古いので、別の方法でインストールする。2023/5/20時点で、nvim-metalsを使うには、v.0.9.0以上でないといけない。

https://github.com/neovim/neovim/wiki/Installing-Neovim#appimage-universal-linux-package

```
apt remove neovim
apt install curl
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim.appimage
chmod u+x nvim.appimage
./nvim.appimage
```

うまくいかなかった。

```
root@be0a823e5999:/# ./nvim.appimage
dlopen(): error loading libfuse.so.2

AppImages require FUSE to run. 
You might still be able to extract the contents of this AppImage 
if you run it with the --appimage-extract option. 
See https://github.com/AppImage/AppImageKit/wiki/FUSE 
for more information
```

その場合は以下のようにするように書かれていたので、従う。

```
./nvim.appimage --appimage-extract
./squashfs-root/AppRun --version
```

バージョンは0.9.0なのでOK

```
root@be0a823e5999:/# ./squashfs-root/AppRun --version
NVIM v0.9.0
Build type: RelWithDebInfo
LuaJIT 2.1.0-beta3
Compilation: /usr/bin/gcc-10 -O2 -g -Og -g -Wall -Wextra -pedantic -Wno-unused-parameter -Wstrict-prototypes -std=gnu99 -Wshadow -Wconversion -Wvla -Wdouble-promotion -Wmissing-noreturn -Wmissing-format-attribute -Wmissing-prototypes -fno-common -Wno-unused-result -Wimplicit-fallthrough -fdiagnostics-color=always -fstack-protector-strong -DUNIT_TESTING -DINCLUDE_GENERATED_DECLARATIONS -D_GNU_SOURCE -I/__w/neovim/neovim/.deps/usr/include/luajit-2.1 -I/usr/include -I/__w/neovim/neovim/.deps/usr/include -I/__w/neovim/neovim/build/src/nvim/auto -I/__w/neovim/neovim/build/include -I/__w/neovim/neovim/build/cmake.config -I/__w/neovim/neovim/src -I/usr/include -I/__w/neovim/neovim/.deps/usr/include -I/__w/neovim/neovim/.deps/usr/include -I/__w/neovim/neovim/.deps/usr/include -I/__w/neovim/neovim/.deps/usr/include -I/__w/neovim/neovim/.deps/usr/include -I/__w/neovim/neovim/.deps/usr/include

   system vimrc file: "$VIM/sysinit.vim"
  fall-back for $VIM: "/__w/neovim/neovim/build/nvim.AppDir/usr/share/nvim"

Run :checkhealth for more info
```

ついでに、インストールしたディレクトリを移動し、リンクを貼っておく。

```
mv squashfs-root /
ln -s /squashfs-root/AppRun /usr/bin/nvim
nvim
```

mvに失敗した。もともとルートディレクトリにいたのでmvの必要はなかった。

```
root@be0a823e5999:/# mv squashfs-root /
mv: 'squashfs-root' and '/squashfs-root' are the same file
```

リンクを貼った後、nvimコマンドでneovimを起動できた。

# nvim-metalsのインストール
以下のnvim-metalsのREADMEに従ってインストールや設定をしていく。

https://github.com/scalameta/nvim-metals

まず以下に従い、Coursierをインストールする。

https://get-coursier.io/docs/cli-installation

```
curl -fL "https://github.com/coursier/launchers/raw/master/cs-x86_64-pc-linux.gz" | gzip -d > cs
chmod +x cs
./cs setup
```

いろいろ聞かれたが、全部Yにした。

```
root@be0a823e5999:/# ./cs setup
Checking if a JVM is installed
  No JVM found, should we try to install one? [Y/n] Y
  Should we update ~/.profile? [Y/n] Y
Some shell configuration files were updated. It is recommended to close this terminal once the setup command is done, and open a new one for the changes to be taken into account.

Checking if ~/.local/share/coursier/bin is in PATH
  Should we add ~/.local/share/coursier/bin to your PATH via ~/.profile? [Y/n] Y

Checking if the standard Scala applications are installed
  Installed ammonite
  Installed cs
  Installed coursier
  Installed scala
  Installed scalac
  Installed scala-cli
  Installed sbt
  Installed sbtn
  Installed scalafmt
```

そしたらいよいよnvim-metalsをインストールする。

一旦HOMEに移動する。

```
cd ~
```

設定ファイルを作成し、MetalsのGitHubにあったサンプルの設定ファイルをコピペする。

```
mkdir -p ~/.config/nvim
nvim ~/.config/nvim/init.lua
```

init.luaに以下の内容をコピペする。

参照元：https://github.com/scalameta/nvim-metals/discussions/39

```lua
-------------------------------------------------------------------------------
-- These are example settings to use with nvim-metals and the nvim built-in
-- LSP. Be sure to thoroughly read the `:help nvim-metals` docs to get an
-- idea of what everything does. Again, these are meant to serve as an example,
-- if you just copy pasta them, then should work,  but hopefully after time
-- goes on you'll cater them to your own liking especially since some of the stuff
-- in here is just an example, not what you probably want your setup to be.
--
-- Unfamiliar with Lua and Neovim?
--  - Check out https://github.com/nanotee/nvim-lua-guide
--
-- The below configuration also makes use of the following plugins besides
-- nvim-metals, and therefore is a bit opinionated:
--
-- - https://github.com/hrsh7th/nvim-cmp
--   - hrsh7th/cmp-nvim-lsp for lsp completion sources
--   - hrsh7th/cmp-vsnip for snippet sources
--   - hrsh7th/vim-vsnip for snippet sources
--
-- - https://github.com/wbthomason/packer.nvim for package management
-- - https://github.com/mfussenegger/nvim-dap (for debugging)
-------------------------------------------------------------------------------
local api = vim.api
local cmd = vim.cmd
local map = vim.keymap.set

----------------------------------
-- PLUGINS -----------------------
----------------------------------
cmd([[packadd packer.nvim]])
require("packer").startup(function(use)
  use({ "wbthomason/packer.nvim", opt = true })

  use({
    "hrsh7th/nvim-cmp",
    requires = {
      { "hrsh7th/cmp-nvim-lsp" },
      { "hrsh7th/cmp-vsnip" },
      { "hrsh7th/vim-vsnip" },
    },
  })
  use({
    "scalameta/nvim-metals",
    requires = {
      "nvim-lua/plenary.nvim",
      "mfussenegger/nvim-dap",
    },
  })
end)

----------------------------------
-- OPTIONS -----------------------
----------------------------------
-- global
vim.opt_global.completeopt = { "menuone", "noinsert", "noselect" }

-- LSP mappings
map("n", "gD",  vim.lsp.buf.definition)
map("n", "K",  vim.lsp.buf.hover)
map("n", "gi", vim.lsp.buf.implementation)
map("n", "gr", vim.lsp.buf.references)
map("n", "gds", vim.lsp.buf.document_symbol)
map("n", "gws", vim.lsp.buf.workspace_symbol)
map("n", "<leader>cl", vim.lsp.codelens.run)
map("n", "<leader>sh", vim.lsp.buf.signature_help)
map("n", "<leader>rn", vim.lsp.buf.rename)
map("n", "<leader>f", vim.lsp.buf.format)
map("n", "<leader>ca", vim.lsp.buf.code_action)

map("n", "<leader>ws", function()
  require("metals").hover_worksheet()
end)

-- all workspace diagnostics
map("n", "<leader>aa", vim.diagnostic.setqflist)

-- all workspace errors
map("n", "<leader>ae", function()
  vim.diagnostic.setqflist({ severity = "E" })
end)

-- all workspace warnings
map("n", "<leader>aw", function()
  vim.diagnostic.setqflist({ severity = "W" })
end)

-- buffer diagnostics only
map("n", "<leader>d", vim.diagnostic.setloclist)

map("n", "[c", function()
  vim.diagnostic.goto_prev({ wrap = false })
end)

map("n", "]c", function()
  vim.diagnostic.goto_next({ wrap = false })
end)

-- Example mappings for usage with nvim-dap. If you don't use that, you can
-- skip these
map("n", "<leader>dc", function()
  require("dap").continue()
end)

map("n", "<leader>dr", function()
  require("dap").repl.toggle()
end)

map("n", "<leader>dK", function()
  require("dap.ui.widgets").hover()
end)

map("n", "<leader>dt", function()
  require("dap").toggle_breakpoint()
end)

map("n", "<leader>dso", function()
  require("dap").step_over()
end)

map("n", "<leader>dsi", function()
  require("dap").step_into()
end)

map("n", "<leader>dl", function()
  require("dap").run_last()
end)

-- completion related settings
-- This is similiar to what I use
local cmp = require("cmp")
cmp.setup({
  sources = {
    { name = "nvim_lsp" },
    { name = "vsnip" },
  },
  snippet = {
    expand = function(args)
      -- Comes from vsnip
      vim.fn["vsnip#anonymous"](args.body)
    end,
  },
  mapping = cmp.mapping.preset.insert({
    -- None of this made sense to me when first looking into this since there
    -- is no vim docs, but you can't have select = true here _unless_ you are
    -- also using the snippet stuff. So keep in mind that if you remove
    -- snippets you need to remove this select
    ["<CR>"] = cmp.mapping.confirm({ select = true }),
    -- I use tabs... some say you should stick to ins-completion but this is just here as an example
    ["<Tab>"] = function(fallback)
      if cmp.visible() then
        cmp.select_next_item()
      else
        fallback()
      end
    end,
    ["<S-Tab>"] = function(fallback)
      if cmp.visible() then
        cmp.select_prev_item()
      else
        fallback()
      end
    end,
  }),
})

----------------------------------
-- LSP Setup ---------------------
----------------------------------
local metals_config = require("metals").bare_config()

-- Example of settings
metals_config.settings = {
  showImplicitArguments = true,
  excludedPackages = { "akka.actor.typed.javadsl", "com.github.swagger.akka.javadsl" },
}

-- *READ THIS*
-- I *highly* recommend setting statusBarProvider to true, however if you do,
-- you *have* to have a setting to display this in your statusline or else
-- you'll not see any messages from metals. There is more info in the help
-- docs about this
-- metals_config.init_options.statusBarProvider = "on"

-- Example if you are using cmp how to make sure the correct capabilities for snippets are set
metals_config.capabilities = require("cmp_nvim_lsp").default_capabilities()

-- Debug settings if you're using nvim-dap
local dap = require("dap")

dap.configurations.scala = {
  {
    type = "scala",
    request = "launch",
    name = "RunOrTest",
    metals = {
      runType = "runOrTestFile",
      --args = { "firstArg", "secondArg", "thirdArg" }, -- here just as an example
    },
  },
  {
    type = "scala",
    request = "launch",
    name = "Test Target",
    metals = {
      runType = "testTarget",
    },
  },
}

metals_config.on_attach = function(client, bufnr)
  require("metals").setup_dap()
end

-- Autocmd that will actually be in charging of starting the whole thing
local nvim_metals_group = api.nvim_create_augroup("nvim-metals", { clear = true })
api.nvim_create_autocmd("FileType", {
  -- NOTE: You may or may not want java included here. You will need it if you
  -- want basic Java support but it may also conflict if you are using
  -- something like nvim-jdtls which also works on a java filetype autocmd.
  pattern = { "scala", "sbt", "java" },
  callback = function()
    require("metals").initialize_or_attach(metals_config)
  end,
  group = nvim_metals_group,
})
```

nvimを起動したら、エラーが出た。

```
Error detected while processing /root/.config/nvim/init.lua:
E5113: Error while calling lua chunk: vim/_editor.lua:0: /root/.config/nvim/init.lua..nvim_exec2() called at /root/.config/nvim/init.lua:0: Vim(packadd):E919: Directory not found in 'packpath': "pack/*/o
pt/packer.nvim"
stack traceback:
        [C]: in function 'nvim_exec2'
        vim/_editor.lua: in function 'cmd'
        /root/.config/nvim/init.lua:30: in main chunk
Press ENTER or type command to continue
```

たぶんpackerが入っていないのが原因なので、以下に従って入れる。

https://github.com/wbthomason/packer.nvim

```
apt install git
git clone --depth 1 https://github.com/wbthomason/packer.nvim\
 ~/.local/share/nvim/site/pack/packer/start/packer.nvim
```

nvimを起動したら、またなんか出た。

```
Error detected while processing /root/.config/nvim/init.lua:
E5113: Error while calling lua chunk: /root/.config/nvim/init.lua:130: module 'cmp' not found:
        no field package.preload['cmp']
        no file './cmp.lua'
        no file '/__w/neovim/neovim/.deps/usr/share/luajit-2.1.0-beta3/cmp.lua'
        no file '/usr/local/share/lua/5.1/cmp.lua'
        no file '/usr/local/share/lua/5.1/cmp/init.lua'
        no file '/__w/neovim/neovim/.deps/usr/share/lua/5.1/cmp.lua'
        no file '/__w/neovim/neovim/.deps/usr/share/lua/5.1/cmp/init.lua'
        no file './cmp.so'
        no file '/usr/local/lib/lua/5.1/cmp.so'
        no file '/__w/neovim/neovim/.deps/usr/lib/lua/5.1/cmp.so'
        no file '/usr/local/lib/lua/5.1/loadall.so'
stack traceback:
        [C]: in function 'require'
        /root/.config/nvim/init.lua:130: in main chunk
Press ENTER or type command to continue
```

nvim-cmpプラグインは入っているはずなんだけどな。行き詰まったのでChatGPTさんに聞いてみた。

```
Based on the provided init.lua configuration, it seems that the cmp plugin is being used, but it's not being loaded correctly. To resolve the error, you need to ensure that the cmp plugin is installed and configured properly.

Here are the steps to fix the issue:

1. Install cmp plugin: Verify that you have the cmp plugin installed. You can do this by checking your plugin manager configuration. In your case, you're using Packer, so make sure that wbthomason/packer.nvim is installed correctly.

2. Load cmp plugin before init.lua: In your init.lua file, move the line that loads the cmp plugin (require("hrsh7th/nvim-cmp")) above the line that initializes Packer (require("packer").startup(function(use) ... end)). This ensures that cmp is loaded before your configuration runs.

3. Rebuild plugins: After making the changes, rebuild your plugins by running :PackerCompile and then :PackerSync inside Neovim to ensure that all plugins, including cmp, are installed and loaded correctly.

Once you've made these modifications and rebuilt your plugins, the module 'cmp' not found error should be resolved.
```

:PackeCompileと:PackerSyncというコマンドがあるようなので、nvimでそれを叩いてみる。packer使いからしたら当たり前なのだろうけど、ご勘弁。

```
               packer.nvim - finished in 0.912s
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ✓ Installed hrsh7th/cmp-nvim-lsp
 ✓ Installed hrsh7th/cmp-vsnip
 ✓ Installed hrsh7th/vim-vsnip
 ✓ Installed scalameta/nvim-metals
 ✓ Installed nvim-lua/plenary.nvim
 ✓ Installed mfussenegger/nvim-dap
 ✓ Installed hrsh7th/nvim-cmp

 Press 'q' to quit
 Press '<CR>' to show more info
 Press 'd' to show the diff
 Press 'r' to revert an update
```

インストールされたっぽい。もう一度nvimを起動したら、エラーは出なかった。

ではscalaファイルを開いてみよう。

```
nvim hello.scala
```

内容は何でも良いが、今回は以下のようにした。

```scala
object Main extends App {
  println("Hello, world!")
}
```

保存してもう一度開いたら、以下のようにWelcomeされた。

```
[nvim-metals] Welcome to nvim-metals!
[nvim-metals] It looks like you don't have Coursier installed, which you need to install Metals.
[nvim-metals] 
[nvim-metals] You can find instructions on how to install it here: https://get-coursier.io/docs/cli-installationIt looks like you don't have Metals installed yet.
[nvim-metals] 
[nvim-metals] You can do this using `:MetalsInstall`.
[nvim-metals] 
[nvim-metals] If you need to set a specific version, you can set it in your settings table.
Press ENTER or type command to continue
```

Coursierはインストールしたんだけどな。と思ったら、PATHが通ってなかったようなので、~/.profileを再読込する。

```
. ~/.profile
```

そしてもう一度nvimを起動。

```
[nvim-metals] Welcome to nvim-metals!
[nvim-metals] It looks like you don't have Metals installed yet.
[nvim-metals] 
[nvim-metals] You can do this using `:MetalsInstall`.
[nvim-metals] 
[nvim-metals] If you need to set a specific version, you can set it in your settings table.
Press ENTER or type command to continue
```

Coursierについては言われなくなった。:MetalsInstallコマンドを叩くように言われたので、従う。

そしたら、補完されるようになった。いいね。

![nvim-metals](img/nvim-metals_2023-05-20%2015-29-06.png)

最後にホームディレクトリをきれいにしておく。もともと別のディレクトリでやれば良かったのだけれど。

```
rm -rf hello.scala .metals
```

# init.luaの追記
このままだと行数の表示などがされていないので、以下の設定を追記しておいた。

```lua
vim.opt["number"] = true
vim.opt["expandtab"] = true
vim.opt["tabstop"] = 4
vim.opt["shiftwidth"] = 4
vim.opt["encoding"] = "utf-8"
vim.opt["fileencoding"] = "utf-8"
vim.opt["cursorline"] = true
vim.opt["signcolumn"] = "yes"

vim.api.nvim_create_augroup( 'html', {} )
vim.api.nvim_create_autocmd( {'BufEnter', 'BufWinEnter'}, {
  group = 'html',
  pattern = '*.html',
  command = 'setlocal tabstop=2 shiftwidth=2'
})
```

ついでにファジーファインダーも入れておく。

https://github.com/ibhagwan/fzf-lua

以下の日本語の説明を見て入れた。

https://zenn.dev/botamotch/articles/a41052477342d5

:PackerInstallコマンドというものもあるようなので、そちらを叩いてみる。:PackerSyncとかとの違いはよく分かっていない。

```
               packer.nvim - finished in 0.852s
 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ✓ Installed kyazdani42/nvim-web-devicons
 ✓ Installed ibhagwan/fzf-lua

 Press 'q' to quit
 Press '<CR>' to show more info
 Press 'd' to show the diff
 Press 'r' to revert an update
```

fzf自体も入れておかないといけない。

```
apt install fzf
```

~/workフォルダを作って、hello.scalaを作成して、nvim内で:MetalsInstallをした。この状態で`:FzfLua files`とすると、以下のように表示される。一部文字化けしてるけどいったんスルーする。

![fzflua](img/fzflua_2023-05-20%2016-08-04.png)

# コンテナイメージ化
この設定のNeoVimをどこでも使えるように、コンテナイメージ化したい。Dockerfileを書くのが面倒だったので、いま作ったコンテナをそのままコンテナイメージにしようと思う。

コンテナからexitした後、以下を実行する

```
docker stop vim-scala
docker commit vim-scala vim-scala:1.0
```

1.3GBにもなってしまった。Dockerfileを使ったり、Ubuntuではなくalphineを使ったりすれば軽量化できるのかもしれないが、いったんこのままでいこう。

```
$ docker images | grep vim-scala
vim-scala                                                        1.0       e7ec0f53a48a   About a minute ago   1.3GB
```

以下のように起動する。

```
docker run --rm -it --workdir /tmp -v $(pwd):/tmp vim-scala:1.0 nvim
```