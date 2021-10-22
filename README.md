# discord-ext-menus-views

A simple layer over [discord.ext.menus](https://github.com/Rapptz/discord-ext-menus) using the new `discord.ui` components in [discord.py](https://github.com/Rapptz/discord.py) v2.0.

For those who do not want to rewrite their code to use the new framework, this library makes it easy to switch to views by changing just a couple of lines of code.

## Installing

Make sure you are using [discord.py](https://github.com/Rapptz/discord.py) v2.0 or above, probably from the master branch. Install this library from git:

```sh
pip install git+https://github.com/oliver-ni/discord-ext-menus-views
```

For maximum compatibility, this library does not list discord.py as a dependency. You must provide it yourself.

## Usage

This library provides two classes, `ViewMenu` and `ViewMenuPages`, which replace `Menu` and `MenuPages`, respectively. These classes aim to be as close as possible to the original classes, but use views (buttons) instead of reactions.

Most of the work is done for you; simply:

1. Replace `Menu` and `MenuPages` with `ViewMenu` and `ViewMenuPages`.
2. In `send_initial_message`, replace your `ctx.send` (or similar) with a `self.send_with_view` call.
   - This step may not apply if you're using `MenuPages` and do not have your own `send_initial_message`.

In most cases, this should already work. However, you probably want to take advantage of the `Interaction` object, which is now the second argument passed into the callback (replacing the reaction payload). You can do as you please with it.

**By default, the library will automatically defer the interaction (for maximal compatibility with ext.menus).** If you would like to turn this off, you can set the `auto_defer` kwarg in the initializer to False.

Here's a simple example:

```py
from discord.ext import menus
from discord.ext.menus.views import ViewMenu

class MyMenu(ViewMenu):
    async def send_initial_message(self, ctx, channel):
        return await self.send_with_view(channel, f"Hello {ctx.author}")

    @menus.button("\N{THUMBS UP SIGN}")
    async def on_thumbs_up(self, interaction):
        await interaction.followup.send(f"Thanks {self.ctx.author}!", ephemeral=True)

    @menus.button("\N{THUMBS DOWN SIGN}")
    async def on_thumbs_down(self, interaction):
        await interaction.followup.send(f"That's not nice {self.ctx.author}...", ephemeral=True)

    @menus.button("\N{BLACK SQUARE FOR STOP}\ufe0f")
    async def on_stop(self, payload):
        self.stop()
```

An example of `ViewMenuPages`:

```py
from discord.ext import menus
from discord.ext.menus.views import ViewMenuPages

class MySource(menus.ListPageSource):
    def __init__(self, data):
        super().__init__(data, per_page=4)

    async def format_page(self, menu, entries):
        offset = menu.current_page * self.per_page
        return '\n'.join(f'{i}. {v}' for i, v in enumerate(entries, start=offset))

# somewhere else:
pages = ViewMenuPages(source=MySource(range(1, 100)), clear_reactions_after=True)
await pages.start(ctx)
```
