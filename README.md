# UITPad (AI Notepad)

**UITPad** is a desktop text and code editor aimed at **C++** workflows. It gives you a familiar editor window with syntax highlighting, multiple open files, optional spell checking, light/dark theming, and built-in **build and run** support using common toolchains. An optional **AI assistant** (via API key) can answer questions, explain code, and help with generation from inside the app.

---

## What it does

| Area | Behavior |
|------|----------|
| **Editing** | Multi-document editing with tabs; open, save, and track modified files. |
| **C++ highlighting** | Real-time syntax coloring driven by rule files (keywords, types, operators, preprocessor tokens). |
| **Spell check** | Dictionary-based checking (language can follow app settings). |
| **Compile / run** | Detects and uses **GCC (g++)**, **MSVC (`cl.exe`)**, or **Clang**, with selectable C++ standards (C++11 through C++23). Output appears in a dedicated output panel. |
| **AI** | Optional **CodeMind** assistant: chat, explain selection, complete code, generate from `//` comments. See [Using the AI assistant](#using-the-ai-assistant) below. |
| **UI** | Qt Widgets interface; theme management for a consistent look. |

For a deeper walkthrough of how actions flow through the code (open file, highlighting, etc.), see [`FLOW_DETAILS.md`](FLOW_DETAILS.md).

---

## Using the AI assistant

UITPad talks to models through **[OpenRouter](https://openrouter.ai/)** (HTTP API). The app is configured to use **`gpt-4o-mini`** and sends requests to `https://openrouter.ai/api/v1/chat/completions`. The system prompt targets **C++** help and **French** replies.

### 1. Get an API key

1. Create an account on OpenRouter if needed.  
2. Create an API key at [openrouter.ai/keys](https://openrouter.ai/keys) (keys often look like `sk-or-v1-...`).

### 2. Save the key in UITPad

Open **AI → AI Settings…**, paste your key, then **Save**. The key is stored locally via Qt (`QSettings`, organization **CodeMind**, application **AIAssistant**). UITPad needs **network access** to call the API.

### 3. Open the chat panel

- **AI → Toggle AI Chat**, or **Ctrl+Shift+C**, or the **AI Chat** button on the toolbar.  
- A dock titled **Assistant IA** appears on the right (you can show or hide it anytime).

### 4. What you can do

| Action | How |
|--------|-----|
| **Ask a question** | Type in the chat input and press **Send** (or Enter). Plain text is sent as a normal chat question. |
| **Generate code from a prompt** | Start your message with **`//`** (two slashes). The rest of the line is treated as a code-generation prompt (e.g. `// binary search in C++`). |
| **Explain selected code** | Select code in the editor, then **AI → Explain Selection** or **Ctrl+Shift+E**. The chat panel opens and the explanation appears there. |
| **Complete code** | **AI → Complete Code** or **Ctrl+Shift+Space** sends all text **from the start of the file up to the cursor** and asks the model to continue from there. |
| **Generate from comment** | Put the cursor on a line (often a `//` comment describing the code you want), then **AI → Generate from Comment** or **Ctrl+Shift+G** — that **entire line** is used as the generation prompt. |

Responses may include markdown code blocks; the chat UI can show code with copy/insert options where supported.

### 5. Troubleshooting

- If you see a message about configuring the API key first, open **AI → AI Settings…** and save a non-empty key.  
- Errors from the network or API (invalid key, quota, etc.) are shown in the chat as error messages.

Implementation details live in [`Source Files/AIAssistant.cpp`](Source%20Files/AIAssistant.cpp) and the **AI** menu in [`Source Files/MainWindow.cpp`](Source%20Files/MainWindow.cpp).

---

## Tech stack

- **Language:** C++17  
- **GUI & platform:** [Qt](https://www.qt.io/) **5 or 6** — `Widgets` and `Network` modules  
- **Build system:** [CMake](https://cmake.org/) 3.16+  
- **Resources:** Qt `.qrc` files for icons/syntax word lists and dictionary data  

External tools (not bundled): a C++ compiler on your PATH or configured for detection if you use **Build / Run**; an API key if you use the **AI** features.

---

## Prerequisites

1. **Qt** — Install Qt 5.15+ or Qt 6.x with the **Qt Widgets** and **Qt Network** components.  
2. **CMake** — 3.16 or newer.  
3. **C++ compiler** — e.g. MSVC (Visual Studio), MinGW g++, or Clang, for compiling *your* projects from UITPad (optional if you only edit files).  
4. **C++ toolchain for building UITPad itself** — same as above, plus Qt’s development files.

On Windows, Qt is often installed via the [Qt Online Installer](https://www.qt.io/download); point CMake at your Qt prefix (see below).

---

## How to build and run

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd UITPad
```

### 2. Set Qt for CMake

CMake must find Qt. Typical approaches:

- **Environment:** set `QTDIR` to your Qt kit path (the folder that contains `lib/cmake/Qt6` or `lib/cmake/Qt5`), **or**  
- **CMake variable:** pass `-DCMAKE_PREFIX_PATH="C:/Qt/6.x.x/msvc2019_64"` (adjust to your install).

If you use Qt Creator, it usually sets this when you open the project.

### 3. Configure and build

**Out-of-source build (recommended):**

```bash
cmake -B build -DCMAKE_PREFIX_PATH="<path-to-qt>"
cmake --build build --config Release
```

On Windows with Visual Studio generator, the executable is often under `build/Release/UITPad.exe` (or `build/Debug/...` for Debug).

**Single-configuration generators (Ninja, Unix Makefiles):**

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH="<path-to-qt>"
cmake --build build
```

### 4. Run

- From the build tree, run the `UITPad` executable (`.exe` on Windows).  
- Or open the project in **Qt Creator**: open `CMakeLists.txt`, select a kit, then **Run**.

---

## Project layout (overview)

| Path | Role |
|------|------|
| `Source Files/` | Application implementation (main window, editor, compiler, AI, highlighters, …) |
| `Header Files/` | Headers for the above |
| `Form Files/` | Qt Designer `.ui` files |
| `Resources Files/` | `.qrc` assets, syntax word lists, dictionary |

---

## Contributing

Issues and pull requests are welcome. Please build with the CMake/Qt versions above and keep changes focused and consistent with the existing code style.
