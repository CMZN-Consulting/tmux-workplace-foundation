# Tmux Workplace Foundation

This repository is a small workplace that you copy and start: your terminal splits into two side-by-side areas called seats, each running its own shell in a folder of its own. You describe the workplace in a few short text files, and [Tmux Workplace](https://github.com/CMZN-Consulting/tmux-workplace) (🏢 below) turns those files into a running tmux session, tmux being a program that shows several terminals inside one.

```text
  Left: a shell in seats/left        Right: a shell in seats/right
+----------------------------------+----------------------------------+
| $                                | $                                |
|                                  |                                  |
|                                  |                                  |
+----------------------------------+----------------------------------+
[Home] 0:Main*
```

Inside tmux, a session holds windows, which it shows one at a time, and a window holds panes. 🏢 calls them office, room and seat: the office Home is the session, the room Main is its window, and Left and Right are its two panes. The bar at the bottom names the session and its windows.

## The start

You need Linux on x86_64 with bash 5.2, jq 1.7, tmux 3.5a or later, git, curl, tar and the usual system programs (`setpriv`, `env`, `mkdir`, `cp`, `rm`, `sha256sum`, `stat`). `bash --version`, `jq --version` and `tmux -V` show which versions you have. Copy the workplace and enter it:

```sh
git clone https://github.com/CMZN-Consulting/tmux-workplace-foundation my-workplace
cd my-workplace
```

🏢 reads the text files with a program called the Typify reader. These commands download its release 1.0.0 into the folder `reader`, check that the download is exactly the published file, unpack it, and put it on your `PATH` for this terminal. The last command prints a line that begins with `1.0.0`:

```sh
mkdir reader
curl -fL -o reader/typify.tar.gz https://github.com/CMZN-Consulting/typify/releases/download/v1.0.0/typify-linux-x86_64.tar.gz
echo '88b1e82b0f560a5d237fb1433b502190bbcde04b867cf1544e2727a3189dc32a  reader/typify.tar.gz' | sha256sum --check --strict
tar -xpzf reader/typify.tar.gz -C reader
export PATH="$PWD/reader/typify/bin:$PATH"
typify --version
```

Now plan the workplace, build it, start it and ask how it is:

```sh
./workplace plan
./workplace build
./workplace up
./workplace status
```

- `plan` reads the files under `config` and writes what it found to `config/PLAN`. The first time, `./workplace` also fetches 🏢 itself into the folder `frame`, at the one version that this repository names.
- `build` checks the plan and writes what `up` needs to `config/BUILD`.
- `up` starts the tmux session with its room and seats.
- `status` says how each part is. Among its lines are `ok      session/home`, `ok      seat/Left` and `ok      seat/Right`, and one line that begins with `attach`.

To see the workplace, run the rest of the `attach` line in a terminal. It looks like this, with your own folder in place of the first dots and a name made from it in place of the second:

```text
tmux -S '…/my-workplace/config/BUILD/RUN/config-…' attach
```

Inside tmux, press Ctrl-b, let go, and then press an arrow key to move between seats. Press Ctrl-b, let go, and then d to leave tmux while the workplace keeps running.

## Three examples

Run the examples in order, each after the one before, in the folder `my-workplace` and in the terminal where you ran the start, after leaving tmux. To see a result, run the `attach` line again.

Each example changes a few files and then runs the same commands. `plan` comes first: if it prints an error, stop there, fix the file and run `plan` again, and the workplace keeps running as it was. Then `down` and `unbuild` stop the workplace and clear the old build, and `build`, `up` and `status` start it again.

### A third seat

A seat takes two files and one line. The first file names the seat's folder and holds the line `agent : none`, which says that no kind of agent works in the seat: a kind, declared in a file of its own, would describe a program that works there by itself and what it may see. The second file names the seat's command. The line adds the seat to the room. Here a third seat, Middle, joins the room Main and gets its own folder, which `.keep` holds in git if you commit it:

```sh
mkdir -p seats/middle 'config/programs/OFFICE[home]/ROOM[main]/SEAT[middle]/run'
touch seats/middle/.keep
printf '%s\n' 'agent : none' 'directory : seats/middle' > 'config/programs/OFFICE[home]/ROOM[main]/SEAT[middle]/DIRECTORIES.typify'
printf '%s\n' 'command : ["bash"]' > 'config/programs/OFFICE[home]/ROOM[main]/SEAT[middle]/run/START.typify'
printf '%s\n' 'seat : Middle = home / main / middle' >> config/programs/STRUCTURE.typify
./workplace plan
./workplace down
./workplace unbuild
./workplace build
./workplace up
./workplace status
```

`status` now also shows `ok      seat/Middle`, and the room is split in three.

### Another command for a seat

A seat runs the command in its `run/START.typify`. Here Right runs `top`, which lists the machine's busiest programs, in place of a shell:

```sh
printf '%s\n' 'command : ["top"]' > 'config/programs/OFFICE[home]/ROOM[main]/SEAT[right]/run/START.typify'
./workplace plan
./workplace down
./workplace unbuild
./workplace build
./workplace up
./workplace status
```

`status` still shows `ok      seat/Right`, and that seat now shows `top`. When a seat's program ends, for example when you press q in `top`, the seat keeps its pane, `status` shows it as `gone` instead of `ok`, and `./workplace restart` starts it again.

### A second room

A room is one more line in `config/programs/STRUCTURE.typify`. Here a room Side joins the office, Middle's line names Side in place of Main, and Middle's folder of programs under `config/programs` moves to the new room's folder; its working folder `seats/middle` stays where it is. tmux shows the new room as a second window, which the bar at the bottom lists beside Main:

```sh
mkdir -p 'config/programs/OFFICE[home]/ROOM[side]'
mv 'config/programs/OFFICE[home]/ROOM[main]/SEAT[middle]' 'config/programs/OFFICE[home]/ROOM[side]/'
printf '%s\n' \
  'office : home = Home' \
  'room : home / main = Main : even-horizontal' \
  'room : home / side = Side : even-horizontal' \
  'seat : Left = home / main / left' \
  'seat : Right = home / main / right' \
  'seat : Middle = home / side / middle' \
  > config/programs/STRUCTURE.typify
./workplace plan
./workplace down
./workplace unbuild
./workplace build
./workplace up
./workplace status
```

`status` shows `ok      seat/Middle` as before. Inside tmux, press Ctrl-b, let go, and then n to move to the next window, Side, where Middle now sits alone.

## The stop

When you are done, stop the workplace and clear its build:

```sh
./workplace down
./workplace unbuild
```

`down` ends the tmux session and its seats. `unbuild` removes `config/BUILD`. 🏢 also keeps a small folder of its own state for the workplace in your user's state folder, and `unbuild` leaves it there.

## What is in this repository

| File                                                                     | Its role                                                                                   |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| `workplace`                                                              | the command you run: it fetches 🏢 at the one version it names and hands it your words     |
| `config/workplace.json`                                                  | names the three language files under `config/layers` that this workplace uses              |
| `config/tmux.conf`                                                       | tmux's settings for this workplace; empty, so tmux's own defaults hold                     |
| `config/layers/Structure.typify`                                         | 🏢's language for offices, rooms and seats, copied exactly from the version named          |
| `config/layers/Agents.typify`                                            | 🏢's language for a seat's folder and its kind of agent, copied exactly                    |
| `config/layers/Host.typify`                                              | 🏢's language for a seat's command, copied exactly                                         |
| `config/programs/STRUCTURE.typify`                                       | the office Home, its room Main and the seats Left and Right                                |
| `config/programs/OFFICE[home]/ROOM[main]/SEAT[left]/DIRECTORIES.typify`  | Left's folder, `seats/left`, and no kind of agent                                          |
| `config/programs/OFFICE[home]/ROOM[main]/SEAT[left]/run/START.typify`    | Left's command, `bash`                                                                     |
| `config/programs/OFFICE[home]/ROOM[main]/SEAT[right]/DIRECTORIES.typify` | Right's folder, `seats/right`, and no kind of agent                                        |
| `config/programs/OFFICE[home]/ROOM[main]/SEAT[right]/run/START.typify`   | Right's command, `bash`                                                                    |
| `seats/left/.keep`                                                       | keeps Left's folder in git, which carries no empty folder                                  |
| `seats/right/.keep`                                                      | keeps Right's folder in git                                                                |
| `README.md`                                                              | this page                                                                                  |
| `.github/workflows/ci.yml`                                               | the checks GitHub runs: the start on two tmux versions, this page's commands, and the rest |
| `package.json`                                                           | names the formatter, a program that keeps these files' layout tidy, and its two commands   |
| `bun.lock`                                                               | the exact versions of the formatter and its tools, written by Bun                          |
| `.formatterignore`                                                       | tells the formatter to leave the copied language files as they are                         |
| `.gitignore`                                                             | keeps what the commands write, the fetched 🏢 and the reader out of git                    |
| `LICENSE`                                                                | the licence of this repository                                                             |

The folders `config/PLAN`, `config/BUILD`, `frame` and `reader` appear when you run the commands above, and git ignores them.

The seats' folders lie outside `config`, because 🏢 refuses a seat's folder inside the configuration. The version of 🏢 stands in `workplace`; the reader's release and the sum that checks its download stand in this page's download commands. The formatter runs under Bun, a JavaScript runtime, and only checks the files' layout: `bun install` and then `bun run format:check`. Running the workplace needs neither.
