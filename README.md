# Monopoly (working title)

A property-trading board game on Roblox, built by five friends. This README is the development guide: how to get set up, how we work day to day, and the rules that keep us from stepping on each other.

For what we're building and why, see the **game plan doc**. For what to work on, see the **[issues](../../issues)**.

> Things marked `TODO` need filling in by whoever sets that piece up.

---

## How it all fits together

- **Code lives in this repo.** We write Luau in VS Code, and **Rojo** syncs it into Roblox Studio.
- **The game lives in one Roblox place**, owned by our Roblox Group. TODO: Group link · Place ID `TODO`
- **Everyone develops in their own local copy of that place.** You sync your code into *your* copy, test it, and open a pull request. Nobody syncs code into the shared place.
- **Team Create on the shared place is only for building things by hand** in Studio: the map, models, lighting.

```
 your files (VS Code) ──rojo serve──▶ your local copy of the place ──Play──▶ you test it
        │
        └── git push ──▶ pull request ──▶ review ──▶ main
```

---

## One-time setup

Do these once. If you get stuck, fix this README so the next person doesn't.

### 1. Install the tools

**Roblox Studio:** download from [create.roblox.com](https://create.roblox.com) and sign in.

**Rokit** installs our tools at the exact versions pinned in `rokit.toml`, so everyone runs the same thing.

macOS / Linux:
```bash
curl -sSf https://raw.githubusercontent.com/rojo-rbx/rokit/main/scripts/install.sh | bash
```
Windows (PowerShell):
```powershell
Invoke-RestMethod https://raw.githubusercontent.com/rojo-rbx/rokit/main/scripts/install.ps1 | Invoke-Expression
```
Restart your terminal afterward.

**VS Code** with these extensions:
- **Rojo** (by evaera)
- **Luau Language Server** (by JohnnyMorganz) for autocomplete and type checking
- **StyLua** (by JohnnyMorganz) for formatting
- **Selene** (by Kampfkarren) for linting

Turn on **Format on Save** in VS Code settings.

### 2. Clone and install

```bash
git clone https://github.com/ck4135/monopoly.git
cd monopoly
rokit install       # installs Rojo, Wally, StyLua, Selene at the pinned versions
wally install       # installs Luau packages into Packages/
rojo plugin install # installs the Rojo plugin into Roblox Studio
```

### 3. Make your local copy of the place

1. Open Studio, go to **My Experiences → Group Experiences**, and open our place.
2. **File → Save to File As** and save it as `local.rbxl` inside the repo folder. It's gitignored, so it never gets committed.
3. Close that window. From now on you open `local.rbxl` for coding, not the shared place.

### 4. Check it works

1. Open `local.rbxl` in Studio.
2. In the repo folder, run `rojo serve`.
3. In Studio, open the **Rojo** plugin and click **Connect**.
4. Press **Play**. You should see the starter messages in the Output window.

If that works, you're set up.

---

## Daily workflow

```bash
git checkout main
git pull
git checkout -b feat/short-description   # e.g. feat/roll-dice, fix/rent-double, art/placeholder-pieces
rojo serve
```

1. Open `local.rbxl` in Studio and click **Connect** in the Rojo plugin.
2. Write code in VS Code. Every save syncs into Studio instantly.
3. Press **Play** to test.
   - **Multiplayer:** in the **Test** tab, set **Clients and Servers** to 2 or more players and click **Start**. Studio runs a local server plus one window per player.
4. Commit small and often, then push and open a pull request:
   ```bash
   git add -A
   git commit -m "Roll two dice and move the piece"
   git push -u origin feat/short-description
   ```
5. Link the issue in the PR (`Closes #12`), get one approval, then **squash and merge**.
6. Delete the branch and start the next one from a fresh `main`.

### Refreshing your local copy

Your code always comes from your files, but anything built by hand in Studio (map, models, lighting) lives in the shared place. When someone says in the chat that they changed it:

1. Open the shared place from **Group Experiences**.
2. **File → Save to File As**, overwrite `local.rbxl`.
3. Reconnect Rojo. Your code comes straight back.

---

## The rules

These are the few things that actually break stuff if we get them wrong.

1. **Never run `rojo serve` into the shared place.** Rojo overwrites whatever place is open. In the shared place, that's everyone's code.
2. **Never edit synced scripts inside Studio.** Rojo overwrites them on the next sync, and the change never reaches Git. Edit files in VS Code.
3. **Team Create is for building, not coding.** Map, models, and lighting changes happen in the shared place. Say so in the chat when you change something big.
4. **Nothing goes to `main` without a pull request and one approval.**
5. **The server decides everything.** Clients ask the server to do things; they never change money, ownership, or turns themselves.

---

## Project structure

TODO: confirm once the Rojo project is set up.

```
src/
  server/    → ServerScriptService       (match server, runs the real game)
  client/    → StarterPlayerScripts      (UI and input, per player)
  shared/    → ReplicatedStorage/Shared  (code both sides use)
  engine/    → ReplicatedStorage/Engine  (pure game rules, no Roblox APIs, fully tested)
Packages/    → ReplicatedStorage/Packages (Wally packages, gitignored)
gen-ai/      → standards every AI tool follows
```

`default.project.json` maps these folders into the game. If you add a folder that needs to sync, add it there.

---

## Code style

- Put `--!strict` at the top of every file.
- Format with **StyLua**, lint with **Selene**. Before pushing:
  ```bash
  stylua src
  selene src
  ```
  CI runs both on every pull request, so a failing check means one of these needs fixing.
- Use the `task` library (`task.wait`, `task.spawn`, `task.delay`), never the old `wait`, `spawn`, or `delay`.
- Game rules go in `src/engine` as plain modules with tests. Keep Roblox APIs out of that folder.

---

## Tracking work

We use **GitHub issues** with **epics**.

- Each **epic** (label `epic`) is one area of Phase 1: Board, Turns and movement, Jail, Trading, and so on.
- Real work is a **sub-issue** of an epic. Open the epic and click **Create sub-issue**, or use the **Task** template and attach it with **Add existing issue**.
- Give every sub-issue one type label: `code`, `art`, `design`, `content`, or `research`.
- Keep sub-issues small: a few hours to a couple of days. Bigger? Split it.
- **Assign yourself** before you start so nobody doubles up. Anyone can pick up anything whose blockers are done.

---

## Using AI

AI is welcome. A few rules keep it useful:

- Every AI tool reads the **`gen-ai/`** folder (via `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`, and `.cursor/rules/`). If an AI keeps making the same mistake, add one line to the matching file in `gen-ai/standards/`.
- **You own the code you open a PR for**, whether or not AI wrote it. Read it before asking for review.
- Keep AI PRs small. A 2,000-line PR doesn't get a real review.
- **Studio MCP:** if you let an AI control Studio, only ever connect it to **your `local.rbxl`**, never the shared place. Use it to playtest and check things; code changes still go through files.

---

## Publishing

There's one place for now, so **whatever gets published is what players see**. It stays private until we launch.

- Only people with the **Publish experiences** permission publish: TODO (names).
- Publish from a build of `main`, not from someone's branch.
- Studio keeps version history, so a bad publish can be rolled back from the Creator Dashboard.
- **Before we save any player data or go public, add a separate PROD place**, so testing never touches real players' saves.

---

## Troubleshooting

| Problem | Fix |
| --- | --- |
| Rojo plugin won't connect | Check `rojo serve` is running and the plugin shows the same port (default 34872). Restart both. |
| My changes don't show up in Studio | Make sure you're in `local.rbxl` and Rojo says **Connected**. Check the Output window for sync errors. |
| Studio says "Failed to fetch place info" | Quit Studio fully (Cmd+Q / close from the taskbar) and reopen it directly. Check [status.roblox.com](https://status.roblox.com). Sign out and back in. |
| Can't see the place in Group Experiences | Make sure you accepted the Group invite and have the **Developer** role. |
| `rokit install` command not found | Restart your terminal after installing Rokit. |
| CI fails on formatting | Run `stylua src`, commit, push again. |
| Someone's map change is missing | Refresh your local copy (see above). |

---

## Links

- Game plan doc: TODO
- Roblox Group: TODO
- Place: TODO
- [Rojo docs](https://rojo.space/docs)
- [Roblox Creator Docs](https://create.roblox.com/docs)
- [Luau docs](https://luau.org)
