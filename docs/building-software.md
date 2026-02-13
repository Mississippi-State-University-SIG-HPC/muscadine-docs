# Building Software
Build systems automate the process of compiling source code into executables or libraries.  
They manage dependencies, compiler flags, and installation steps, making builds portable and repeatable.

---

## General Flow
**Source Code → Configuration → Build Files → Compilation Installation**

### Source Code
- Contains `.c`, `.cpp`, `.h`, and related source files.
- Includes metadata files that describe how to build (e.g., `CMakeLists.txt`, `configure.ac`, `Makefile.am`).
### Configuration
- Detects system settings, compilers, and libraries.
- Generates platform-specific build files (like Makefiles or Visual Studio projects).

### Build
- Uses generated build files to compile and link the code.

### Installation
- Copies binaries, headers, and other files to system directories.

---

## Example: CMake
**Configuration → Build → Install**

```bash
# create build dir
mkdir build; cd build

# Step 1: Configure (generate build system files)
cmake .. -DCMAKE_INSTALL_PREFIX=/path/to/prefix

# (Optional) Use ccmake for more transparency
ccmake ..

# Step 2: Build
make -j

# Step 3: Install
make install
```
```{note}
If you're unsure what variables you can tweak in a cmake project, you can launch a terminal user interface by using `ccmake` in place of `cmake`
```

**Files involved:**
- `CMakeLists.txt`: Describes targets, dependencies, and options.
- `build/`: Directory containing generated Makefiles or project files.

**Typical flow:**

**CMakeLists.txt → CMake generates Makefiles → make builds → make install**

---

## Example: GNU Autotools
**Bootstrap → Configure → Build → Install**

```bash
# Step 1: Generate configuration scripts (for maintainers)
autoreconf -i

# Step 2: Configure (generate Makefiles)
./configure --prefix=/path/to/prefix

# Step 3: Build
make -j$(nproc)

# Step 4: Install
make install
```

**Files involved:**
- `configure.ac`: Defines project metadata and system checks.
- `Makefile.am`: High-level Makefile template.
- `Makefile.in` and `configure`: Generated intermediate files.

**Typical flow:**

```text
configure.ac + Makefile.am → autoreconf → configure → Makefiles → make → make install
```

---

## Comparison Summary

|Step|CMake|Autotools|
|--:|---|---|
|Input file(s)|`CMakeLists.txt`|`configure.ac`, `Makefile.am`|
|Configuration tool|`cmake`|`autoreconf`, `configure`|
|Output files|Makefiles or project files|`Makefile`|
|Build command|`cmake --build` / `make`|`make`|
|Install command|`cmake --install` / `make install`|`make install`|
|Portability focus|Cross-platform (Windows/Linux/macOS)|Primarily Unix-like systems|