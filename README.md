# RunAnywhere Starter Apps

Example applications demonstrating the RunAnywhere SDK for on-device AI across multiple platforms.

## Apps Included

| Platform | Directory | Features |
|----------|-----------|----------|
| **Flutter** | `flutter-app/` | LLM Chat, STT, TTS, Voice Assistant |
| **React Native** | `react-native-app/` | LLM Chat, STT, TTS, Voice Assistant |
| **Kotlin/Android** | `kotlin-app/` | LLM Chat, STT, TTS, Voice Assistant |

## Prerequisites

- **Flutter**: Flutter 3.10+ with Dart 3.0+
- **React Native**: Node.js 18+, React Native 0.83+
- **Kotlin**: Android Studio Hedgehog+, JDK 17+

All apps require:
- Physical device (arm64) recommended for best performance
- ~540MB storage for AI models (LLM + STT + TTS)

## Quick Start

### Flutter

```bash
cd flutter-app
flutter pub get
flutter run
```

### React Native

```bash
cd react-native-app
npm install
npx react-native run-android  # or run-ios
```

### Kotlin/Android

```bash
cd kotlin-app
./gradlew assembleDebug
./gradlew installDebug
```

## Models

Each app downloads these models on first run:

| Type | Model | Size |
|------|-------|------|
| LLM | SmolLM2 360M Q8_0 | ~400MB |
| STT | Whisper Tiny English | ~75MB |
| TTS | Piper US English | ~65MB |

## Tutorials

Full tutorial series available at [runanywhere.ai/blog](https://runanywhere.ai/blog):

- Part 1: Chat with LLMs
- Part 2: Speech-to-Text with Whisper
- Part 3: Text-to-Speech with Piper
- Part 4: Voice Pipeline with VAD

## Resources

- [RunAnywhere Documentation](https://docs.runanywhere.ai)
- [SDK Repository](https://github.com/RunanywhereAI/runanywhere-sdks)

## License

MIT License - See LICENSE file for details.
