# 🔐 GGHACKS — Hybrid Android Security Engineering

A full-stack Android application demonstrating advanced native integration:
JNI/C++ modules with libcurl + OpenSSL, compile-time string obfuscation,
multi-ABI NDK packaging, and a custom login/authentication panel.

Built with Java (Android SDK), C++17 (NDK), Gradle (Kotlin DSL), and CMake.

> *"Engineering is not about what you build — it's about how you build it."*

---

## Philosophy

This project showcases **systems-level Android engineering** through the lens of a
real application. The architecture demonstrates:

- **Hybrid native/Java architecture** — C++ JNI module communicating with Android
  Java layer through clean interface boundaries
- **Multi-channel networking** — libcurl (native C++), OkHttp (Java), Retrofit
  (Java) — three independent HTTP stacks in one app
- **Security through obscurity at the compiler level** — compile-time XOR string
  encryption using C++14 `constexpr` templates
- **Multi-ABI native packaging** — prebuilt libraries for both `arm64-v8a` and
  `armeabi-v7a`, linked via CMake
- **Professional build system** — Gradle Kotlin DSL, CMake NDK integration, proper
  dependency management with version pinning

---

## Architecture

```
┌───────────────────────────────────────────────────────┐
│                 MainActivity.java                      │
│                                                       │
│  ┌─────────┐  ┌──────────┐  ┌──────────────────────┐ │
│  │ Login UI│  │  OkHttp  │  │ JNI Bridge            │ │
│  │ Panel   │  │ Upload   │  │ System.loadLibrary()  │ │
│  │         │  │ (ZIP)    │  └──────────┬───────────┘ │
│  └────┬────┘  └────┬─────┘             │             │
│       │            │                   │             │
│  ┌────┴────────────┴───────────────────┴───────────┐ │
│  │         HTTP Endpoints                           │ │
│  │  Auth POST  ·  File Upload  ·  Status Download  │ │
│  └─────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────┘
                        │
                   JNI Boundary
                        │
┌───────────────────────────────────────────────────────┐
│              libinject32.so  (C++ NDK)                │
│                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │  libcurl     │  │  OpenSSL     │  │  OBFUSCATION │ │
│  │  HTTP POST   │  │  TLS 1.2     │  │  Compile-time│ │
│  │  Client      │  │  Encryption  │  │  XOR Strings │ │
│  └──────────────┘  └──────────────┘  └─────────────┘ │
└───────────────────────────────────────────────────────┘
```

---

## Code Examples

### JNI Bridge with Native HTTP (C++)

```cpp
// Direct libcurl HTTP POST from native code with OpenSSL TLS
extern "C" JNIEXPORT jint JNICALL
Java_com_DeFexGGxANDLUA_hacks_MainActivity_executeScript(JNIEnv* env, jobject) {
    CURL* curl = curl_easy_init();
    curl_easy_setopt(curl, CURLOPT_URL, "https://<host>/endpoint.php");
    curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 0L);

    // Read Android system properties natively
    char build_id[PROP_VALUE_MAX];
    __system_property_get("ro.build.id", build_id);

    // POST with custom headers
    struct curl_slist* headers = NULL;
    headers = curl_slist_append(headers, "Content-Type: application/x-www-form-urlencoded");
    curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);

    std::string post = "username=andlua&password=" + std::string(build_id);
    curl_easy_setopt(curl, CURLOPT_POSTFIELDS, post.c_str());

    CURLcode res = curl_easy_perform(curl);
    curl_easy_cleanup(curl);
    return (res == CURLE_OK) ? 1 : 0;
}
```

### Compile-Time String Obfuscation (C++14)

```cpp
// Adam Yaxley's compile-time XOR obfuscator.
// Strings are encrypted at compile time, decrypted on first use,
// and automatically zeroed on scope exit via RAII.

class obfuscated_string {
    constexpr auto encrypt(char c, size_t i) const {
        return c ^ key(i);  // XOR with position-dependent key
    }
public:
    constexpr obfuscated_string(const char* s) : data{encrypt(s, 0)...} {}
    auto decrypt() { /* runtime XOR reversal */ }
};

#define AY_OBFUSCATE(str) \
    (obfuscated_string<sizeof(str)>{str}.decrypt())

// Usage: all sensitive strings are opaque in the binary
auto url = AY_OBFUSCATE("https://api.example.com/auth");
```

### Multi-ABI CMake Build

```cmake
# Links prebuilt libcurl + OpenSSL for both ABIs
cmake_minimum_required(VERSION 3.22.1)
project("inject32")

add_library(inject32 SHARED main.cpp)

# Per-ABI prebuilt library paths
set(CURL_LIB_ARM64 ${CMAKE_SOURCE_DIR}/curl/curl-android-arm64-v8a/lib/libcurl.a)
set(CURL_LIB_ARMV7 ${CMAKE_SOURCE_DIR}/curl/curl-android-armeabi-v7a/lib/libcurl.a)

if(${ANDROID_ABI} STREQUAL "arm64-v8a")
    target_link_libraries(inject32 ${CURL_LIB_ARM64} ${OPENSSL_ARM64})
else()
    target_link_libraries(inject32 ${CURL_LIB_ARMV7} ${OPENSSL_ARMV7})
endif()
```

### Android UI with Material Design (Java)

```java
// Modern Android Material UI with constraint layout, custom gradients,
// video background, and ripple animations.
public class MainActivity extends AppCompatActivity {
    static { System.loadLibrary("inject32"); }  // JNI bridge
    private native int executeScript();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Video background
        VideoView video = findViewById(R.id.videoView);
        video.setVideoURI(Uri.parse("android.resource://" + getPackageName()
                           + "/" + R.raw.video));

        // Custom Material Button with ripple
        Button loginBtn = findViewById(R.id.login_button);
        loginBtn.setOnClickListener(v -> login());
    }
}
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| UI | Android XML (ConstraintLayout), Material Design |
| Java Logic | Android SDK 29-34, OkHttp 4.9, Retrofit 2.9 + Gson |
| Native (C++) | NDK r25+, libcurl 7.x, OpenSSL 1.1.x |
| Build | Gradle 8.2 (Kotlin DSL), CMake 3.22 |
| String Protection | Compile-time XOR obfuscation (`OBFUSCATION.h`) |
| ABIs | `arm64-v8a`, `armeabi-v7a` |
| IDE | Android Studio Hedgehog+ |

---

## File Structure

```
app/
├── build.gradle.kts          # SDK 29-34, NDK + CMake, OkHttp/Retrofit deps
├── CMakeLists.txt             # NDK build config, prebuilt lib linking
└── src/main/
    ├── AndroidManifest.xml    # Permissions + launcher activity
    ├── java/.../MainActivity.java     # UI, login, HTTP, file ops
    ├── cpp/
    │   ├── CMakeLists.txt     # Native build
    │   ├── main.cpp           # JNI entry + libcurl HTTP POST
    │   └── Component/
    │       ├── HTTP/httplib.h           # cpp-httplib v0.14.3 (9377 lines)
    │       └── ENCRYPTION/OBFUSCATION.h # Compile-time XOR strings
    ├── curl/                  # Prebuilt libcurl + OpenSSL for both ABIs
    └── res/                   # Layouts, drawables, icons, video, fonts
```

---

*"Good code is invisible. Great code is undeniable."*
