# Chapter 23 — Compiling Programs

## Quick Reference

| Command         | Description                                        |
| --------------- | -------------------------------------------------- |
| `./configure`   | Detect build environment, generate Makefile        |
| `make`          | Build targets defined in Makefile                  |
| `make install`  | Install built binaries to system directories       |
| `make clean`    | Remove compiled objects and binaries               |
| `make -j N`     | Build using N parallel jobs                        |
| `gcc`           | GNU C Compiler                                     |
| `g++`           | GNU C++ Compiler                                   |

---

## The Compilation Pipeline

```
source code (.c)
      ↓  compiler (gcc)
object files (.o)    ← compiled modules
      ↓  linker (ld)
executable binary    ← final program
```

- **Compiler** — translates source into machine code (`.o` object files)
- **Linker** — combines object files + shared libraries into a final executable
- **Header files** (`.h`) — describe interfaces between modules; included at compile time
- **Libraries** (`/lib`, `/usr/lib`) — shared code reused across programs (`.so` = shared, `.a` = static)

**Compiled vs. interpreted:**

| Compiled (C, C++, Rust, Go) | Interpreted (Python, Perl, Ruby, shell) |
| --------------------------- | --------------------------------------- |
| Faster execution            | Faster development cycle                |
| Translated once             | Translated on every run                 |
| Build step required         | Run directly                            |

---

## Setting Up Build Tools on Fedora

```bash
# Install the full development toolchain group
sudo dnf group install "Development Tools"

# Or install individually
sudo dnf install gcc gcc-c++ make cmake meson ninja-build

# Install build dependencies for a package in the repos
sudo dnf builddep packagename        # e.g. dnf builddep vim

# Check if gcc is present
which gcc
gcc --version
```

---

## The Standard Build Sequence (Autotools)

Most source tarballs from GNU and similar projects follow this pattern:

```bash
# 1. Download and unpack
wget https://ftp.gnu.org/gnu/diction/diction-1.11.tar.gz
tar xzf diction-1.11.tar.gz

# Inspect the tarball before unpacking (good habit)
tar tzvf diction-1.11.tar.gz | head

# 2. Enter source directory
cd diction-1.11

# 3. Read the docs first
less README
less INSTALL

# 4. Configure — detects your system, generates Makefile
./configure

# 5. Build
make

# 6. Install (to /usr/local/bin by default)
sudo make install

# Verify
which diction
man diction
```

> If `./configure` fails with errors, you're missing a dependency. Read the error carefully — it names the missing library or tool. Install it with `dnf` and re-run `./configure`.

---

## ./configure

Shell script that probes your system and generates a `Makefile` tailored to it.

```bash
./configure                             # defaults (installs to /usr/local)
./configure --prefix=$HOME/.local       # install to home dir (no sudo needed)
./configure --help                      # list all available options
./configure --enable-feature            # enable optional feature
./configure --disable-feature           # disable something
./configure --with-library=/path        # specify library location
```

**Key output files it generates:**
- `Makefile` — the build instructions `make` reads
- `config.h` — system-specific definitions included in source
- `config.log` — detailed log if something goes wrong (check this on failures)

---

## make

Reads `Makefile` and builds only what's out of date (newer source than object). Extremely efficient for large projects.

```bash
make                        # build default target (usually 'all')
make install                # install to prefix (usually /usr/local)
make clean                  # delete compiled objects and binaries
make distclean              # clean + remove configure-generated files (reset to tarball state)
make uninstall              # remove installed files (if supported)
make check                  # run test suite (if included)
make -n                     # dry run: show what would be done without doing it
make -j$(nproc)             # parallel build using all CPU threads
make -j$(nproc) install     # parallel build then install
```

**Parallel builds on your i9-12900K (24 threads):**
```bash
make -j24                   # use all threads — dramatically faster on large projects
make -j$(nproc)             # portable: auto-detects thread count
```

### How make decides what to rebuild

`make` compares timestamps: if a source file is newer than its compiled output, it recompiles. If you touch a source file, make rebuilds that module and anything that depends on it.

```bash
touch getopt.c      # pretend getopt.c was modified
make                # rebuilds getopt.o and relinks the final binary
```

### Makefile anatomy (simplified)

```makefile
CC = gcc                          # variable
CFLAGS = -O2 -Wall

target: dependency1 dependency2   # target: what it depends on
	$(CC) $(CFLAGS) -o target dependency1 dependency2   # recipe (TAB-indented)

diction: diction.o misc.o sentence.o
	$(CC) -o diction diction.o misc.o sentence.o

diction.o: diction.c diction.h
	$(CC) -c diction.c
```

> Makefile recipes **must** be indented with a real tab character, not spaces.

---

## Install Locations

| Location          | Purpose                                          |
| ----------------- | ------------------------------------------------ |
| `/usr/local/bin`  | Default for `make install` — manually built apps |
| `/usr/local/lib`  | Manually installed libraries                     |
| `$HOME/.local/bin`| User-only install (`./configure --prefix=~/.local`) |
| `/usr/src`        | Source maintained by your distro                 |
| `~/src`           | Conventional location for your own source builds |

**User-only install (no sudo):**
```bash
./configure --prefix=$HOME/.local
make -j$(nproc)
make install
# binary lands in ~/.local/bin — make sure it's in your PATH
```

---

## Source Tree Structure

```
project-1.11/
├── README          # description and overview — read first
├── INSTALL         # build instructions
├── COPYING         # license
├── NEWS            # changelog
├── configure       # autoconf-generated configure script
├── configure.in    # configure source (autoconf input)
├── Makefile.in     # Makefile template (configure fills it in)
├── *.c             # C source files (one per module)
├── *.h             # header files (interfaces between modules)
└── *.o             # object files (appear after make)
```

---

## Modern Build Systems

The book covers Autotools (`./configure` + `make`), which is still common, but many modern projects use:

### CMake
```bash
mkdir build && cd build
cmake ..                        # configure (generates build files)
cmake --build . -j$(nproc)      # build
sudo cmake --install .          # install
```

### Meson + Ninja (common on Fedora/GNOME projects)
```bash
meson setup build               # configure
cd build
ninja -j$(nproc)                # build (ninja is faster than make)
sudo ninja install
```

---

## Practical Tips for Fedora

```bash
# Find missing build deps by name from error messages
sudo dnf install pkgconfig      # for pkg-config errors
sudo dnf install glibc-devel    # for missing system headers

# Search for a header file's package
dnf provides '*/regex.h'
dnf provides '*/stdio.h'

# Use checkinstall to build an RPM instead of raw make install
sudo dnf install checkinstall
sudo checkinstall make install  # creates and installs an RPM you can dnf remove later

# Speed up repeated builds with ccache
sudo dnf install ccache
export CC="ccache gcc"
export CXX="ccache g++"
make -j$(nproc)

# Strip debug symbols to reduce binary size (optional)
strip /usr/local/bin/programname
```

---

## Quick Reference: Build Something from Source

```bash
# Full workflow
wget https://example.com/project-1.0.tar.gz
tar xzf project-1.0.tar.gz
cd project-1.0
less README && less INSTALL
./configure --prefix=$HOME/.local
make -j$(nproc)
make check                      # optional: run tests
sudo make install               # or: make install (if using --prefix=~/.local)
```
