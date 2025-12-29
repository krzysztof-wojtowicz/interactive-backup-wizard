# Interactive Backup Wizard

**Interactive Backup Wizard** is a C-based command-line tool designed to manage interactive, real-time file backups on Linux systems. It utilizes the Linux `inotify` API to monitor filesystem events and automatically synchronizes changes from source directories to target locations.

The application uses a multi-process architecture to handle multiple backup tasks concurrently without blocking the user interface.

## 🚀 Features

* **Real-Time Synchronization:** Continuously monitors source directories for changes (creation, modification, deletion, moves) and mirrors them to the target immediately.
* **Recursive Backup:** Handles deep directory structures efficiently.
* **Smart Restore:** An optimized restore function that only copies files that are missing or have changed (based on modification time and size), rather than copying the entire directory blindly.
* **Symlink Handling:** Correctly copies symbolic links. If an absolute link points inside the source directory, it is adjusted to point to the corresponding location in the target directory.
* **Concurrency:** Each backup task runs in its own child process, allowing the main CLI to remain responsive.
* **Space Handling:** Supports paths with spaces using bash-style quoting (e.g., `"my folder/file.txt"`).
* **Logging:** Detailed activity logging is saved to `workers.log`.

## 📂 Project Structure

The project is organized as follows:

```text
.
├── Makefile          # Build script
├── README.md         # Project documentation
└── src               # Source code and headers
    ├── command_handler.c # Logic for CLI commands (add, end, restore, etc.)
    ├── dict.c            # Linked list dictionary for tracking active processes
    ├── main.c            # Entry point and main event loop
    ├── parser.c          # Command line argument parser
    ├── signal_handler.c  # Signal handling logic
    ├── utils.c           # General utility functions
    ├── watchers.c        # Inotify wrapper and monitoring logic
    └── worker.c          # Backup worker logic (copying and monitoring)

```

## 🛠️ Build Instructions

This project requires a Linux environment (due to the usage of `inotify` headers).

1. **Clone the repository:**
```bash
git clone <url> sop-backup
cd sop-backup
```

2. **Compile the project:**
Run the `make` command in the root directory.
```bash
make
```

## 💻 Usage

Start the program by running the binary:

```bash
./sop-backup
```

Once inside the interactive shell (`>`), you can use the following commands:

### 1. Start a Backup (`add`)

Starts a continuous backup from source to target.

```bash
add <source_path> <target_path> [target_path_2 ...]
```

* Creates the target directory if it doesn't exist.
* Performs an initial recursive copy.
* Starts a background worker to watch for changes.
* **Note:** If the target directory already exists, it must be empty.

### 2. Stop a Backup (`end`)

Stops the background monitoring process for a specific source-target pair. The files in the target directory remain untouched.

```bash
end <source_path> <target_path> [target_path_2 ...]
```

### 3. List Active Backups (`list`)

Displays all currently running backup processes and their paths.

```bash
list
```

### 4. Restore Data (`restore`)

Restores files from a backup location to the source.

```bash
restore <source_path> <target_path>
```

* **Optimized:** Only copies files that are different (size/mtime) or missing in the source.
* **Blocking:** The shell waits until the restoration is complete.
* **Cleanup:** Deletes files in the source that do not exist in the backup.

### 5. Exit (`exit`)

Terminates all worker processes, cleans up memory, and closes the program gracefully.

```bash
exit
```

## ⚙️ Technical Details

* **Architecture:** The `main` process handles user input and orchestrates tasks. When `add` is called, it `forks` a new worker process. This worker utilizes `inotify` to listen for filesystem events (`IN_CREATE`, `IN_DELETE`, `IN_MOVED_TO`, etc.) and applies them to the target.
* **Signal Handling:** Proper handling of `SIGINT` and `SIGTERM` ensures that all child processes are killed gracefully before the main program exits.
* **Permissions:** File permissions and modification times (`atime`, `mtime`) are preserved during copy and restore operations.

## 📝 Logs

The program generates a `workers.log` file in the execution directory. This file contains detailed logs from all child processes, including file copy operations and detected events.

---

**Author:** Krzysztof Wójtowicz
**Course:** Operating Systems (Systemy Operacyjne) @ MiNI WUT
