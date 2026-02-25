# SlimeVR-Server Installation & Build Guide

This file documents the full setup process used to compile and run SlimeVR Server and GUI on Windows 11. Follow the steps in order. Most commands are executed from a PowerShell terminal.

---

## 1. Prerequisites

- Git
- Java 17+ (JDK)
- Node.js 16.9+ with npm
- pnpm package manager
- Microsoft Edge WebView2 (Windows) or webkit2gtk (Linux)
- Rust (via rustup)


## 2. Clone repository

```powershell
cd <folder where you keep projects>

# clone with submodules
git clone --recursive https://github.com/SlimeVR/SlimeVR-Server.git
cd SlimeVR-Server
```

## 3. Java setup

Download a JDK 17 distribution (Temurin recommended) and extract locally.

```powershell
# example using download link; adapt version if necessary
Invoke-WebRequest \
  https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.8+7/OpenJDK17U-jdk_x64_windows_hotspot_17.0.8_7.zip \
  -OutFile jdk17.zip
Expand-Archive -LiteralPath jdk17.zip -DestinationPath jdk17

# configure environment
setx JAVA_HOME "E:\0_project\SlimeVR\code\SlimeVR-Server\jdk17\jdk-17.0.8+7"
# update current session
$env:JAVA_HOME='E:\0_project\SlimeVR\code\SlimeVR-Server\jdk17\jdk-17.0.8+7'
$env:PATH = "$env:JAVA_HOME\bin;" + $env:PATH
java -version  # should report 17.x
```

## 4. Build Java backend

```powershell
# build desktop server jar (skip android tasks if toolchain missing)
cd e:\0_project\SlimeVR\code\SlimeVR-Server
./gradlew :server:desktop:shadowJar -x :server:android:compileDebugJavaWithJavac

# result jar at:
# server\desktop\build\libs\slimevr.jar
```

To run the server:

```powershell
cd server\desktop\build\libs
java -jar slimevr.jar run
```

(Optional) run `./gradlew run` in project root to start server in IDE.
### Running GUI & server together

Once the backend JAR and GUI dependencies are built, you can start both components:

```powershell
# open two terminals

# terminal 1: launch server
cd e:\0_project\SlimeVR\code\SlimeVR-Server\server\desktop\build\libs
java -jar slimevr.jar run

# terminal 2: start GUI in dev mode (from project root or gui folder)
cd e:\0_project\SlimeVR\code\SlimeVR-Server\gui
pnpm run dev
```

The GUI window will appear automatically (or visit http://localhost:5173 in a browser if you prefer). When using the production executable:

```powershell
# after building the GUI with `pnpm run tauri build`
cd e:\0_project\SlimeVR\code\SlimeVR-Server\gui\target\release
.\slimevr.exe
```

These commands should be executed from the indicated directories.

## 5. pnpm & GUI setup

Install `pnpm` globally via npm:

```powershell
npm install -g pnpm
pnpm -v    # verify
```

Install GUI dependencies and start development mode:

```powershell
cd e:\0_project\SlimeVR\code\SlimeVR-Server\gui
pnpm i
pnpm run dev   # launches Tauri window; also accessible at http://localhost:5173
```

To build production executable:

```powershell
pnpm run tauri build
# output: target/release/slimevr.exe
```

## 6. Rust toolchain issues and workarounds

The repository includes a `rust-toolchain.toml` file; the default channel `1.89` downloaded from a faulty mirror caused checksum errors. To work around:

1. Update global rustup server to official and upgrade stable:
   ```powershell
   setx RUSTUP_DIST_SERVER "https://static.rust-lang.org"
   setx RUSTUP_UPDATE_ROOT "https://static.rust-lang.org/rustup"
   # restart session or set env for current run
   $env:RUSTUP_DIST_SERVER="https://static.rust-lang.org";
   $env:RUSTUP_UPDATE_ROOT="https://static.rust-lang.org/rustup";
   rustup update stable
   ```

2. Edit `rust-toolchain.toml` to use `stable` channel and remove components that trigger downloads:
   ```toml
   [toolchain]
   channel = "stable"
   profile = "default"
   components = ["rustc", "cargo", "clippy", "rustfmt"]
   ```

## 7. Additional notes

- Skip the android module or install Android SDK/NDK if you need to build the Android app.
- Code formatting:
  - Java/Kotlin: `./gradlew spotlessCheck` / `spotlessApply`
  - GUI: `pnpm run lint` / `pnpm run lint:fix` / `pnpm run format`.
- SolarXR protocol generation: `cd solarxr-protocol && ./generate-flatbuffer.ps1` (Windows).

---

This `install.MD` contains all the commands used during the setup process. Adjust paths and versions as needed for your environment.

## 8. package exe file

cd e:\0_project\SlimeVR\code\SlimeVR-Server
.\gradlew.bat :server:desktop:shadowJar -x :server:android:compileDebugJavaWithJavac

cd e:\0_project\SlimeVR\code\SlimeVR-Server\gui
pnpm run tauri build

Copy-Item "E:\0_project\SlimeVR\code\SlimeVR-Server\server\desktop\build\libs\slimevr.jar" "D:\Users\liyang\AppData\Local\slimevr\slimevr.jar" -Force