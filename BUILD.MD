## Introduction

Wanted to build TDLib for Android yourself, but encountered issues following the official build instructions? So did I...

That's why I'm offering you a more detailed guide for building TDLib for Android from scratch*

**100% build success is not guaranteed*

## Operating System

The Telegram team recommends using Linux or macOS for building. On Windows, you may encounter some issues during the build process (especially with the `build-openssl.sh` file), so I strongly recommend installing WSL *(Windows Subsystem for Linux)* and Ubuntu distribution. You can find numerous installation guides online, so I'll be brief. In PowerShell, enter:

```bash
# Get a list of available distributions
wsl --list --online
# Install Ubuntu (recommended, but you can choose any other)
wsl --install Ubuntu
```

After this, you'll be asked to enter a username and password (the password is invisible when typing - keep this in mind). Then you'll enter the Linux terminal itself (you can recognize it by the green text at the beginning of the line). To access Linux in the future, simply type `wsl` in the terminal.

## Build Process

1. Let's get started:

```bash
# Navigate to home directory
cd
# Update package list, requires account password
sudo apt update
# Update applications
sudo apt upgrade -y 
```

2. Download TDLib:

```bash
# Clone the official repository
git clone https://github.com/tdlib/td.git
# Navigate to project folder
cd td/example/android
```

3. Install required build tools:

```bash
# This command will download all necessary tools to the system
sudo apt install gperf perl php-cli sed tar coreutils make openjdk-11-jdk openjdk-11-jdk-headless unzip
# Verify everything is installed correctly
./check-environment.sh 
# If the response is an empty line - proceed
```

4. Install Java:

```bash
# Java 17 is recommended for TDLib
sudo apt install openjdk-17-jdk
# Check version
java -version 
# If the above command shows a version other than 17, you need to select it manually:
sudo update-alternatives --config java
```

5. Download Android SDK:

```bash
# Download cmdline-tools
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
# Create SDK folder
# IMPORTANT: it must be located strictly inside td/example/android
mkdir -p SDK/cmdline-tools
# Navigate to created folder
cd SDK
# Extract the downloaded archive
unzip ../commandlinetools-linux-11076708_latest.zip -d cmdline-tools
# Navigate to extracted archive folder
cd cmdline-tools
# Rename cmdline-tools folder inside cmdline-tools to latest
mv cmdline-tools latest
# Configure environment variables
export ANDROID_SDK_ROOT=$HOME/td/example/android/SDK 
export PATH=$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$PATH
# Navigate inside the downloaded folder
cd $ANDROID_SDK_ROOT/cmdline-tools/latest/bin
# Accept license agreement
./sdkmanager --licenses 
# Install required components 
./sdkmanager "ndk;23.2.8568313" "cmake;3.22.1" "build-tools;34.0.0" "platforms;android-34"
# Make sure everything is installed
./sdkmanager --list
```

6. Download and build OpenSSL:

```bash
./build-openssl.sh
```

7. Build TDLib itself:

```bash
./build-tdlib.sh
# IMPORTANT: there should be no "warnings" in the output after the build completes
```

## Obtaining Files

A new `tdlib` folder should be created in the `td/example/android/` directory.

If you used WSL, you can access Linux folders using the left panel in File Explorer (if there's no "Linux" item, enter `\\wsl$` in the address bar at the top). Then navigate to: `home\User\td\example\android\tdlib`

Inside the `tdlib` folder, there will be several folders and 2 archives: `tdlib.zip` and `tdlib-debug.zip`. You most likely won't need `tdlib-debug.zip`, so copy only `tdlib.zip` somewhere on your computer.

The entire `tdlib` folder can be deleted. You can also delete the `build-` folders in the parent directory to free up disk space (they will be recreated when you run `./build-tdlib.sh` again).

## Integrating TDLib into Android Project

You can add the library to your project in different ways. I'll describe 3 main methods:

### Method 1: Direct Addition to Project Files

The simplest way is to copy files directly into your app module.

**Step 1:** Copy Java files

1. Open your project folder in File Explorer *(right-click on app → Open in → Explorer)*
2. Navigate to `app/src/main/java/`
3. Create folder structure: `org/drinkless/tdlib/`
4. Copy all TDLib `.java` files to `app/src/main/java/org/drinkless/tdlib/`

**Step 2:** Copy .so libraries

1. In the same `app/src/main/` folder, create a `jniLibs/` folder (if it doesn't exist)
2. Copy architecture folders and .so files there
3. Make sure the `jniLibs/` folder now contains `arm64-v8a/`, `armeabi-v7a/`, `x86/`, and `x86_64/` folders, with `libtdjni.so` file inside each

**Step 3:** Configure `build.gradle`

Open `app/build.gradle.kts` (Kotlin DSL):

```kotlin
android {
    // ... other settings

    sourceSets {
        getByName("main") {
            jniLibs.srcDirs("src/main/jniLibs")
        }
    }
}
```

Or `app/build.gradle` (Groovy):

```groovy
android {
    // ... other settings

    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/jniLibs']
        }
    }
}
```

**Step 4:** Sync

Click *Sync Project with Gradle Files* in Android Studio.

### Method 2: Integration as Project Module

Creating a separate library module inside your project.

**Step 1:** Create library module

1. In Android Studio: *File → New → New Module*
2. Select *Android Library*
3. Specify:
    - Module name: `tdlib`
    - Package name: `org.drinkless.tdlib`
4. Click *Finish*

**Step 2:** Copy files to module

1. Java files:
    - Delete auto-generated files from `tdlib/src/main/java/`
    - Copy all TDLib `.java` files to `tdlib/src/main/java/org/drinkless/tdlib/`

2. .so libraries:
    - Create `tdlib/src/main/jniLibs/`
    - Copy architecture folders there

**Step 3:** Configure module build.gradle

Open `tdlib/build.gradle.kts` (Kotlin DSL):

```kotlin
plugins {
    id("com.android.library")
}

android {
    namespace = "org.drinkless.tdlib"
    compileSdk = 34

    defaultConfig {
        minSdk = 16
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    sourceSets {
        getByName("main") {
            jniLibs.srcDirs("src/main/jniLibs")
        }
    }
}

dependencies {
    implementation("androidx.annotation:annotation:1.9.1")
}
```

Or `tdlib/build.gradle` (Groovy):

```groovy
plugins {
    id 'com.android.library'
}

android {
    namespace 'org.drinkless.tdlib'
    compileSdk 34

    defaultConfig {
        minSdk 16
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/jniLibs']
        }
    }
}

dependencies {
    implementation 'androidx.annotation:annotation:1.9.1'
}
```

**Step 4:** Check settings.gradle

Make sure the module is included in `settings.gradle.kts` (Kotlin DSL):

```kotlin
include(":app")
include(":tdlib")
```

Or `settings.gradle` (Groovy):

```groovy
include ':app'
include ':tdlib'
```

**Step 5:** Connect module in app

Open `app/build.gradle.kts` (Kotlin DSL):

```kotlin
dependencies {
    implementation(project(":tdlib"))
    // ... other dependencies
}
```

Or `app/build.gradle` (Groovy):

```groovy
dependencies {
    implementation project(':tdlib')
    // ... other dependencies
}
```

**Step 6:** Sync

Click *Sync Project with Gradle Files*.

### Method 3: Creating Maven Library

The most "professional" way — create a separate library.

#### Option A: Local Maven Library

**Step 1:** Create separate library project

1. *File → New → New Project*
2. Select *No Activity*
3. Specify name (for example, `TDLib-Android`)
4. Click *Finish*

**Step 2:** Create library module

1. *File → New → New Module*
2. Select *Android Library*
3. Specify:
    - Module name: `tdlib`
    - Package name: `org.drinkless.tdlib`
4. Click *Finish*

**Step 3:** Copy files

Copy `.java` and `.so` files as in Method 2.

**Step 4:** Configure publishing

Open `tdlib/build.gradle.kts` (Kotlin DSL):

```kotlin
plugins {
    id("com.android.library")
    id("maven-publish")
}

android {
    namespace = "org.drinkless.tdlib"
    compileSdk = 34

    defaultConfig {
        minSdk = 16
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    sourceSets {
        getByName("main") {
            jniLibs.srcDirs("src/main/jniLibs")
        }
    }

    publishing {
        singleVariant("release") {
            withSourcesJar()
            withJavadocJar()
        }
    }
}

dependencies {
    implementation("androidx.annotation:annotation:1.9.1")
}

afterEvaluate {
    publishing {
        publications {
            create<MavenPublication>("release") {
                from(components["release"])

                groupId = "com.yourname"
                artifactId = "tdlib-android"
                version = "1.8.58"
            }
        }
    }
}
```

Or for Groovy `tdlib/build.gradle`:

```groovy
plugins {
    id 'com.android.library'
    id 'maven-publish'
}

android {
    namespace 'org.drinkless.tdlib'
    compileSdk 34

    defaultConfig {
        minSdk 16
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/jniLibs']
        }
    }

    publishing {
        singleVariant('release') {
            withSourcesJar()
            withJavadocJar()
        }
    }
}

dependencies {
    implementation 'androidx.annotation:annotation:1.9.1'
}

afterEvaluate {
    publishing {
        publications {
            release(MavenPublication) {
                from components.release

                groupId = 'com.yourname'
                artifactId = 'tdlib-android'
                version = '1.8.58'
            }
        }
    }
}
```

**Step 5:** Publish locally

Open terminal *(Alt + F12)* in the library project and enter:

```bash
./gradlew :tdlib:publishToMavenLocal
```

The library will be published to `~/.m2/repository/com/yourname/tdlib-android/1.8.58/`

**Step 6:** Connect in main project

Open your main project (where you develop the application).

`settings.gradle.kts` (Kotlin DSL):

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        mavenLocal() // Add for local libraries
    }
}
```

`settings.gradle` (Groovy):

```groovy
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        mavenLocal() // Add for local libraries
    }
}
```

`app/build.gradle.kts` (Kotlin DSL):

```kotlin
dependencies {
    implementation("com.yourname:tdlib-android:1.8.58") // Created library name
}
```

`app/build.gradle` (Groovy):

```groovy
dependencies {
    implementation 'com.yourname:tdlib-android:1.8.58' // Created library name
}
```

**Step 7:** Sync

Click *Sync Project with Gradle Files*.

#### Option B: Publishing to JitPack (for public access)

Follow the same steps 1-4 from Option A, then:

**Step 5:** Prepare for GitHub publication

First, create necessary files in the project root:

1. Create `.gitignore` file:

```gitignore
*.iml
.gradle
/local.properties
/.idea
.DS_Store
/build
/captures
.externalNativeBuild
.cxx
```

2. Create `jitpack.yml` file:

```text
jdk:
- openjdk17
  before_install:
- sdk install java 17.0.1-open
- sdk use java 17.0.1-open
  install:
- ./gradlew :tdlib:publishToMavenLocal
```

3. Update `settings.gradle.kts`:

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "TDLib-Android"
include(":tdlib")
```

**Step 6:** Initialize Git repository

Open Terminal *(Alt + F12)* in Android Studio:

```bash
git init
git add .
git commit -m "Initial commit"
```

**Step 7:** Create GitHub repository

1. Go to [GitHub](github.com)
2. Click *+* → *New repository*
3. Fill in:
   * Repository name: *anything*
   * Description: *anything*
   * Public (required for JitPack)
4. Click *Create* repository

**Step 8:** Push to GitHub

GitHub will show you commands. Execute them in Terminal:

```bash
git remote add origin https://github.com/yourusername/tdlib-android.git
git branch -M main
git push -u origin main
```

Replace *yourusername* with your actual GitHub username.

**Step 9:** Create release on GitHub

1. Open your repository on GitHub
2. Click *Releases*
3. Click *Create a new release*
4. Click *Publish release*

**Step 10:** Build on JitPack

1. Open [Jitpack](jitpack.io)
2. Sign in with GitHub
3. Paste your repository URL in the search field
4. Click *Look up*
5. You'll see a list of releases. Find your version and click *Get it*
6. JitPack will start building

**Step 11:** Use in any project

After successful build, the library is ready to use!

In any Android project, add to `settings.gradle.kts` (Kotlin DSL):

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

And in `app/build.gradle.kts` (Kotlin DSL):

```kotlin
dependencies {
    implementation("com.github.yourusername:tdlib-android:1.8.58")
}
```
Or for `app/build.gradle` (Groovy):

```groovy
dependencies {
    implementation 'com.github.yourusername:tdlib-android:1.8.58'
}
```
