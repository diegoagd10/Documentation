# Advanced Linux Commands & Tools

A curated collection of modern, powerful alternatives to standard Linux commands that make working in the terminal faster, more visual, and more efficient.

---

## File System, Disk Usage, and Information

### ncdu
**Description:** Interactive disk usage analyzer - a better, visual version of df and du  
**Installation:** https://dev.yorhel.nl/ncdu  
**Install:** `sudo apt install ncdu` (Ubuntu/Debian) or `brew install ncdu` (macOS)

### duff
**Description:** Prettier version of df showing drive usage with better visuals  
**Installation:** https://github.com/elmindreda/duff  
**Install:** `sudo apt install duff` (Ubuntu/Debian)

### stat
**Description:** Shows detailed information about files, including filesystem-specific data with `-f` option  
**Built-in:** Standard Linux command

### lshw
**Description:** Displays complete hardware information. Filter with `-C` option (e.g., `lshw -C cpu`)  
**Installation:** https://github.com/lyonel/lshw  
**Install:** `sudo apt install lshw` (Ubuntu/Debian)

### shred
**Description:** Securely overwrites files multiple times before deletion to prevent data recovery
**Built-in:** Standard Linux command

### tomb
**Description:** Command-line tool for creating encrypted file containers - secure storage for sensitive files
**Installation:** https://github.com/dyne/Tomb
**Install:** `sudo apt install tomb` (Ubuntu/Debian)
**Quick Start:**
```bash
# Create tomb and key
tomb dig -s 100 ~/my_secrets.tomb
tomb forge ~/my_secrets.tomb.key
tomb lock ~/my_secrets.tomb -k ~/my_secrets.tomb.key

# Open/mount tomb (default: /media/my_secrets/)
tomb open ~/my_secrets.tomb -k ~/my_secrets.tomb.key

# Add files
cp files.txt /media/my_secrets/

# Close/encrypt tomb
tomb close my_secrets

# List mounted tombs
tomb list
```
**Security Notes:**
- Store key file separately (e.g., USB drive)
- Without password, files are unrecoverable
- Backup both `.tomb` and `.tomb.key` files regularly

---

## Searching, Filtering, and Monitoring

### ripgrep (rg)
**Description:** Rust-powered search tool, faster alternative to grep for finding errors in logs, functions, or files  
**Installation:** https://github.com/BurntSushi/ripgrep  
**Install:** `sudo apt install ripgrep` or `brew install ripgrep`

### fd
**Description:** User-friendly alternative to find with better defaults (recursive, case-insensitive, colorized)  
**Installation:** https://github.com/sharkdp/fd  
**Install:** `sudo apt install fd-find` or `brew install fd`

### fzf (FuzzyFinder)
**Description:** Interactive filtering tool for lists, pipes, and command history  
**Installation:** https://github.com/junegunn/fzf  
**Install:** `git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && ~/.fzf/install`

### glances
**Description:** All-in-one monitoring dashboard with stats, can run as API or web server  
**Installation:** https://github.com/nicolargo/glances  
**Install:** `pip install glances` or `sudo apt install glances`

### iotop
**Description:** Top-style real-time display of processes using the most disk I/O  
**Installation:** https://github.com/Tomas-M/iotop  
**Install:** `sudo apt install iotop` (Ubuntu/Debian)

### dstat
**Description:** Combined timeline view of CPU, RAM, disk, network, and memory statistics  
**Installation:** https://github.com/dagwieers/dstat  
**Install:** `sudo apt install dstat` (Ubuntu/Debian)

### watch
**Description:** Reruns any command at set intervals (e.g., `watch -n 0.5 nvidia-smi`)  
**Built-in:** Standard Linux command

### progress
**Description:** Monitors status and progress of commands like cp, mv, dd, rsync in real-time  
**Installation:** https://github.com/Xfennec/progress  
**Install:** `sudo apt install progress` (Ubuntu/Debian)

---

## Connectivity and Networking

### mosh
**Description:** Mobile shell - alternative to SSH with roaming support (maintains connection when switching networks)  
**Installation:** https://mosh.org/  
**Install:** `sudo apt install mosh` or `brew install mosh`

### mtr
**Description:** Network diagnostic tool tracking latency and packet loss hop-by-hop in real-time  
**Installation:** https://github.com/traviscross/mtr  
**Install:** `sudo apt install mtr` (Ubuntu/Debian)

### dog
**Description:** Colorful DNS lookup tool, better alternative to dig with DNS-over-TLS support and JSON output  
**Installation:** https://github.com/ogham/dog  
**Install:** `brew install dog` or download from releases page

### termshark
**Description:** Terminal UI for tshark - user-friendly interface for packet capture analysis  
**Installation:** https://github.com/gcla/termshark  
**Install:** `sudo apt install termshark` or download from releases

### lsof
**Description:** Lists open files and processes using specific ports (e.g., `lsof -i :22`)  
**Built-in:** Standard Linux command

### ifdata
**Description:** Cleaner network interface information display (part of moreutils)  
**Installation:** Part of moreutils package  
**Install:** `sudo apt install moreutils`

### rsync
**Description:** Smart file transfer tool that syncs only differences, resumes broken transfers, works over SSH  
**Built-in:** Standard Linux command  
**Website:** https://rsync.samba.org/

---

## Directory Navigation and File Management

### ranger
**Description:** Terminal-based file manager with vim keybindings, bulk renames, and previews  
**Installation:** https://github.com/ranger/ranger  
**Install:** `sudo apt install ranger` or `pip install ranger-fm`

### zoxide (z)
**Description:** Smarter cd alternative that learns frequently/recently used directories  
**Installation:** https://github.com/ajeetdsouza/zoxide  
**Install:** `curl -sS https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | bash`

### exa
**Description:** Modern replacement for ls with better colors, tree view, and icons  
**Installation:** https://github.com/ogham/exa  
**Install:** `sudo apt install exa` or `brew install exa`  
**Note:** Consider using `eza` (fork) as exa is no longer maintained: https://github.com/eza-community/eza

### bat
Description: Cat clone with syntax highlighting, line numbers, and git integration
Installation: https://github.com/sharkdp/bat
Install: sudo apt install bat or brew install bat
Note: On Ubuntu/Debian, the command is batcat due to name conflict

### vidir
**Description:** Edit directory names in a text editor (part of moreutils)  
**Installation:** Part of moreutils package  
**Install:** `sudo apt install moreutils`

---

## System Utilities, Processes, and Tasks

### systemd-analyze
**Description:** Shows boot-up performance analysis  
**Usage:** `systemd-analyze blame` - shows which services take longest  
**Usage:** `systemd-analyze critical-chain` - highlights dependency bottlenecks  
**Built-in:** Part of systemd

### procs
**Description:** Modern replacement for ps with better formatting, sorting by CPU, and tree structure  
**Installation:** https://github.com/dalance/procs  
**Install:** `brew install procs` or download from releases

### lazydocker
**Description:** Interactive terminal UI for Docker management  
**Installation:** https://github.com/jesseduffield/lazydocker  
**Install:** `curl https://raw.githubusercontent.com/jesseduffield/lazydocker/master/scripts/install_update_linux.sh | bash`

### taskwarrior
**Description:** Command-line task management tool  
**Installation:** https://taskwarrior.org/  
**Install:** `sudo apt install taskwarrior` or `brew install task`

---

## Programming, Data Handling, and Archives

### ipcalc
**Description:** Command-line subnet calculator for CIDR notation  
**Installation:** https://gitlab.com/ipcalc/ipcalc  
**Install:** `sudo apt install ipcalc` (Ubuntu/Debian)

### wormhole (magic-wormhole)
**Description:** Peer-to-peer, end-to-end encrypted file transfer tool  
**Installation:** https://github.com/magic-wormhole/magic-wormhole  
**Install:** `pip install magic-wormhole`

### ts
**Description:** Adds timestamps to output (part of moreutils)  
**Installation:** Part of moreutils package  
**Install:** `sudo apt install moreutils`

### errno
**Description:** Explains what error numbers mean (part of moreutils)  
**Installation:** Part of moreutils package  
**Install:** `sudo apt install moreutils`

### vipe
**Description:** Insert text editor in Unix pipeline to edit data mid-stream (part of moreutils)  
**Installation:** Part of moreutils package  
**Install:** `sudo apt install moreutils`

### unp
**Description:** Universal unpacker - extracts any archive type automatically  
**Installation:** https://github.com/mitsuhiko/unp  
**Install:** `sudo apt install unp` (Ubuntu/Debian)

### jq
**Description:** Command-line JSON processor for querying and transforming JSON data  
**Installation:** https://github.com/jqlang/jq  
**Install:** `sudo apt install jq` or `brew install jq`

---

## Terminal Recording and AI

### asciinema
**Description:** Records terminal sessions as text-based cast files that can be replayed  
**Installation:** https://asciinema.org/  
**Install:** `sudo apt install asciinema` or `pip install asciinema`

### agg (asciinema-agg)
**Description:** Converts asciinema recordings to animated GIFs  
**Installation:** https://github.com/asciinema/agg  
**Install:** `cargo install --git https://github.com/asciinema/agg`

### ttyd
**Description:** Share your terminal over the web - lightweight web server for broadcasting terminal sessions via browser
**Installation:** https://github.com/tsl0922/ttyd
**Install:** `brew install ttyd` (macOS), `sudo snap install ttyd --classic` (Linux), or download from releases

### fabric
**Description:** CLI tool for interacting with AI
**Installation:** https://github.com/danielmiessler/fabric
**Install:** Follow installation instructions on GitHub

### ollama
**Description:** Run AI models locally and create custom agents
**Installation:** https://ollama.ai/
**Install:** `curl -fsSL https://ollama.com/install.sh | sh`

### whisper (OpenAI Whisper)
**Description:** Automatic speech recognition (ASR) system from OpenAI for transcribing audio files
**Installation:** https://github.com/openai/whisper
**Dependencies:** Requires `ffmpeg` and `setuptools-rust`
**Install:**
```bash
# Install dependencies
sudo apt install ffmpeg
pip install setuptools-rust

# Install whisper
pip install -U openai-whisper
```

---

## Quick Reference: Moreutils Package

Several useful tools come bundled in the **moreutils** package:
- `ifdata` - Clean network interface info
- `vidir` - Edit directories in text editor
- `ts` - Add timestamps to output
- `errno` - Explain error numbers
- `vipe` - Edit pipeline data in text editor

**Install moreutils:** `sudo apt install moreutils` (Ubuntu/Debian) or `brew install moreutils` (macOS)

---

## Installation Tips

### General Installation Methods:
1. **Debian/Ubuntu:** `sudo apt install <package-name>`
2. **macOS:** `brew install <package-name>`
3. **Cargo (Rust):** `cargo install <package-name>`
4. **Python:** `pip install <package-name>`
5. **From source:** Clone repository and follow build instructions

**Note:** Some tools may have different package names on different systems. Check the official installation pages for the most up-to-date instructions.
