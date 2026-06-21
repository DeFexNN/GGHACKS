# GGHACKS — DeFexHacksLoginPanel

Professional overview

GGHACKS is a hybrid Android native + Java project integrating JNI C/C++ modules, custom networking (libcurl + OpenSSL), and UI resources. It demonstrates systems-level Android engineering: multi-ABI native builds, native-bridge patterns, and careful binary/resource management.

Why this repo matters for employers

- Native performance and security: NDK/NDK-build and JNI glue for latency-sensitive tasks.
- Cross-ABI packaging: arm64-v8a and armeabi-v7a handled via per-ABI libraries and Gradle/ndk config.
- Networking & TLS: packaged libcurl + OpenSSL show experience integrating native TLS stacks and handling certs/streams.
- Resource-driven UI: optimized drawables and media handling for responsive UX.

Tech stack

- Java/Kotlin (Android app shell)
- C/C++ (NDK native modules, JNI)
- libcurl, OpenSSL (native networking)
- Gradle, Android Studio, Android NDK

Core features

- JNI native modules for performance-critical operations (app/src/main/cpp)
- Multi-ABI prebuilt libraries for faster CI/dev cycles (app/src/main/cpp/*/lib)
- Custom networking stack using libcurl+OpenSSL for robust TLS-enabled connections
- Resource bundling: optimized images and packaged media resources

Highlighted files & callouts

- app/src/main/cpp/**/libcrypto.a, libssl.a — Prebuilt OpenSSL artifacts (consider rebuilding from source or moving to LFS/releases). Why important: proves ability to integrate and troubleshoot native TLS implementations.
- app/src/main/cpp/** — JNI glue and native implementations. Note: native code includes careful pointer checks and memory management; point this out in walkthroughs during interviews.
- app/src/main/res/raw/video.mp4 — Example of media packaging and optimization.

Code focus areas to discuss in interviews

- JNI boundary design: safe parameter marshaling, minimal JNI calls for perf, and thread attachment strategies.
- TLS integration: custom certificate handling and error propagation from native to Java layers.
- Cross-ABI packaging: strategy for prebuilt binaries vs. building in CI; reproducible builds recommendation.

Build & run (developer)

1. Install Android Studio and NDK (r21+ recommended). Set sdk/ndk paths in local.properties.
2. Open the project in Android Studio and let Gradle sync.
3. Build native modules: Gradle handles ndk build automatically, or run ./gradlew assembleDebug.
4. Run on a device (prefer arm64 for full features).

Security & maintenance notes

- Prebuilt OpenSSL blobs should be audited or rebuilt from source to ensure versions and patches are tracked.
- Large binaries are recommended to be moved to Git LFS or released as separate artifacts to keep the repo light.
- Use private testing environments for any sensitive network or authentication flows.

Interview talking points (short)

- "I architected JNI modules to minimize crossing overhead while keeping native code testable and memory-safe."
- "I handled OpenSSL integration across ABIs and automated packaging in CI to ensure reproducible builds."

Contact

For a walkthrough or pair-programming demo of native modules, open an issue or contact the author.