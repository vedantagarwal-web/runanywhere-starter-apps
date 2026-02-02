# RunAnywhere React Native App - Setup Guide

This guide covers setting up and running the RunAnywhere React Native app on Android. It documents specific issues and workarounds encountered during setup.

## Prerequisites

- **Node.js** (v18 or later)
- **Android Studio** with:
  - Android SDK (API 34+)
  - NDK (side by side)
  - CMake
- **Git**
- **Physical ARM64 Android device** (required - see [Why ARM64 Device Required](#why-arm64-device-required))

## Initial Setup

### 1. Clone and Install Dependencies

```bash
cd runanywhere-starter-apps/react-native-app
npm install
```

### 2. Set JAVA_HOME

Use Android Studio's bundled JDK (JBR):

**Windows (PowerShell):**
```powershell
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"
```

**Windows (CMD):**
```cmd
set JAVA_HOME=C:\Program Files\Android\Android Studio\jbr
```

**macOS/Linux:**
```bash
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
```

### 3. Configure gradle.properties

Copy the example file:
```bash
cp android/gradle.properties.example android/gradle.properties
```

Ensure these settings in `android/gradle.properties`:
```properties
hermesEnabled=true
runanywhere.testLocal=false
runanywhere.rebuildCommons=false
```

> **Note:** `testLocal=false` uses pre-built native libraries. Set to `true` only if you have the SDK source code locally.

### 4. Download Missing Gradle Wrapper JAR

If `android/gradle/wrapper/gradle-wrapper.jar` is missing (gitignored):

```bash
curl -L -o android/gradle/wrapper/gradle-wrapper.jar \
  "https://github.com/gradle/gradle/raw/v8.14.1/gradle/wrapper/gradle-wrapper.jar"
```

### 5. Generate Debug Keystore

If `android/app/debug.keystore` is missing:

**Windows:**
```cmd
"C:\Program Files\Android\Android Studio\jbr\bin\keytool" -genkey -v ^
  -keystore android/app/debug.keystore ^
  -storepass android -alias androiddebugkey -keypass android ^
  -keyalg RSA -keysize 2048 -validity 10000 ^
  -dname "CN=Android Debug,O=Android,C=US"
```

**macOS/Linux:**
```bash
keytool -genkey -v \
  -keystore android/app/debug.keystore \
  -storepass android -alias androiddebugkey -keypass android \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -dname "CN=Android Debug,O=Android,C=US"
```

## Codegen Stub Files (React Native 0.83+ New Architecture)

React Native 0.83+ enforces the New Architecture. Some packages don't fully support codegen. Create stub files for them:

### react-native-nitro-modules

Create directory:
```bash
mkdir -p node_modules/react-native-nitro-modules/android/build/generated/source/codegen/jni
```

Create `NitroModulesSpec.h`:
```cpp
#pragma once
#include <ReactCommon/JavaTurboModule.h>
#include <ReactCommon/TurboModule.h>
#include <jsi/jsi.h>

namespace facebook::react {
JSI_EXPORT
std::shared_ptr<TurboModule> NitroModulesSpec_ModuleProvider(
    const std::string &moduleName,
    const JavaTurboModule::InitParams &params);
} // namespace facebook::react
```

Create `NitroModulesSpec-generated.cpp`:
```cpp
#include "NitroModulesSpec.h"

namespace facebook::react {
std::shared_ptr<TurboModule> NitroModulesSpec_ModuleProvider(
    const std::string &moduleName,
    const JavaTurboModule::InitParams &params) {
  return nullptr;
}
} // namespace facebook::react
```

Create `CMakeLists.txt`:
```cmake
cmake_minimum_required(VERSION 3.13)
set(CMAKE_VERBOSE_MAKEFILE on)

file(GLOB react_codegen_SRCS CONFIGURE_DEPENDS *.cpp)

add_library(
  react_codegen_NitroModulesSpec
  OBJECT
  ${react_codegen_SRCS}
)

target_include_directories(react_codegen_NitroModulesSpec PUBLIC .)

target_link_libraries(
  react_codegen_NitroModulesSpec
  fbjni
  jsi
  reactnative
)

target_compile_reactnative_options(react_codegen_NitroModulesSpec PRIVATE)
```

### react-native-sound

Create directory:
```bash
mkdir -p node_modules/react-native-sound/android/build/generated/source/codegen/jni
```

Create `RNSoundSpec.h`:
```cpp
#pragma once
#include <ReactCommon/JavaTurboModule.h>
#include <ReactCommon/TurboModule.h>
#include <jsi/jsi.h>

namespace facebook::react {
JSI_EXPORT
std::shared_ptr<TurboModule> RNSoundSpec_ModuleProvider(
    const std::string &moduleName,
    const JavaTurboModule::InitParams &params);
} // namespace facebook::react
```

Create `RNSoundSpec-generated.cpp`:
```cpp
#include "RNSoundSpec.h"

namespace facebook::react {
std::shared_ptr<TurboModule> RNSoundSpec_ModuleProvider(
    const std::string &moduleName,
    const JavaTurboModule::InitParams &params) {
  return nullptr;
}
} // namespace facebook::react
```

Create `CMakeLists.txt`:
```cmake
cmake_minimum_required(VERSION 3.13)
set(CMAKE_VERBOSE_MAKEFILE on)

file(GLOB react_codegen_SRCS CONFIGURE_DEPENDS *.cpp)

add_library(
  react_codegen_RNSoundSpec
  OBJECT
  ${react_codegen_SRCS}
)

target_include_directories(react_codegen_RNSoundSpec PUBLIC .)

target_link_libraries(
  react_codegen_RNSoundSpec
  fbjni
  jsi
  reactnative
)

target_compile_reactnative_options(react_codegen_RNSoundSpec PRIVATE)
```

> **Note:** These stubs are needed after each `npm install` as node_modules gets recreated.

## Why ARM64 Device Required

The RunAnywhere SDK includes pre-built native libraries (`librunanywherecore.so`) compiled **only for ARM64 (arm64-v8a)**.

**This means:**
- x86/x86_64 Android emulators will NOT work
- You need a physical ARM64 Android device (most modern Android phones/tablets)

**Error you'll see on emulator:**
```
dlopen failed: library "librunanywherecore.so" not found
```

## Running on Physical Device

### 1. Enable Developer Options & USB Debugging

On your Android device:
1. Go to **Settings > About Phone**
2. Tap **Build Number** 7 times
3. Go back to **Settings > Developer Options**
4. Enable **USB Debugging**

### 2. Connect Device

```bash
adb devices
```

Should show your device:
```
List of devices attached
XXXXXX    device
```

### 3. Set Up Port Forwarding

Required for Metro bundler to communicate with the device:
```bash
adb reverse tcp:8081 tcp:8081
```

### 4. Start Metro Bundler

```bash
npm start
```

### 5. Build and Run

```bash
npx react-native run-android
```

Or build manually:
```bash
cd android
./gradlew assembleDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## Building Release APK

For a standalone APK that doesn't need Metro:

```bash
cd android
./gradlew assembleRelease
```

Output: `android/app/build/outputs/apk/release/app-release.apk`

Install:
```bash
adb install -r app/build/outputs/apk/release/app-release.apk
```

## Package-Specific Configuration

### react-native.config.js

Some packages are platform-restricted. Current configuration:

```javascript
module.exports = {
  dependencies: {
    'react-native-live-audio-stream': {
      platforms: { ios: null },  // Android only
    },
    'react-native-audio-recorder-player': {
      platforms: { ios: null, android: null },  // Disabled
    },
    'react-native-sound': {
      platforms: { ios: null },  // Android only
    },
    'react-native-tts': {
      platforms: { ios: null },  // Android only
    },
  },
};
```

### metro.config.js

Basic configuration for this project:

```javascript
const { getDefaultConfig, mergeConfig } = require('@react-native/metro-config');

const config = {
  resolver: {
    unstable_enableSymlinks: true,
  },
};

module.exports = mergeConfig(getDefaultConfig(__dirname), config);
```

## Troubleshooting

### "hermesEnabled" property not found
Ensure `android/gradle.properties` exists and contains `hermesEnabled=true`.

### Codegen CMake errors
Create the stub files as documented in [Codegen Stub Files](#codegen-stub-files-react-native-083-new-architecture).

### "Unable to load script" on device
1. Ensure Metro is running: `npm start`
2. Ensure port forwarding: `adb reverse tcp:8081 tcp:8081`
3. Check Metro status: `curl http://localhost:8081/status`

### Grey screen after app launch
Metro bundler may have stopped. Restart it:
```bash
npm start
```

### Package not linked errors
Check `react-native.config.js` to ensure the package isn't disabled for your platform.

### AsyncStorage NativeModule is null
Install the missing package:
```bash
npm install @react-native-async-storage/async-storage
```
Then rebuild the app.

## Development Workflow

| Change Type | Action Required |
|-------------|-----------------|
| JavaScript/TypeScript | Save file - hot reloads automatically |
| Assets (images, sounds) | Save file - hot reloads automatically |
| Native code (Java/C++) | Rebuild app |
| New npm packages | Rebuild app |
| gradle.properties changes | Rebuild app |

## Quick Reference Commands

```bash
# Start Metro bundler
npm start

# Run on connected device (debug)
npx react-native run-android

# Build release APK
cd android && ./gradlew assembleRelease

# Install APK manually
adb install -r path/to/app.apk

# View device logs
adb logcat | grep -i "ReactNative\|runanywhere"

# Port forwarding for physical device
adb reverse tcp:8081 tcp:8081

# Check connected devices
adb devices

# Force stop and relaunch app
adb shell am force-stop com.runanywhereaI
adb shell am start -n com.runanywhereaI/.MainActivity
```
