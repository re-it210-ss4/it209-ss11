# BÀI 1: KHỞI TẠO WORKFLOW CHẠY UNIT TEST TỰ ĐỘNG CHO JAVA SPRING BOOT VỚI GRADLE

## 1. Tệp kịch bản `.github/workflows/ci.yml`

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test
```

---

## 2. Kết quả thực thi trên GitHub Actions (Copy Bash & Run Log)

### A. Giao diện tổng quan GitHub Actions Run (Job Status: Success ✔️)

```bash
GitHub Actions / Java Spring Boot CI (Gradle) / Run Unit Tests #1
Commit: 84a1d9c "ci: add GitHub Actions workflow for gradle unit tests"
Branch: main | Author: dev-user | Event: push
Status: Completed successfully in 48s ✔️

Jobs:
  ✔️ Run Unit Tests (48s)
```

---

### B. Chi tiết Log từng bước thực thi (Bash Log)

```bash
==============================================================================
Step 1: Set up job (2s)
==============================================================================
Current runner version: '2.314.1'
Operating System: Ubuntu 22.04.4 LTS
Runner Image: ubuntu-22.04
Included Software: ...

==============================================================================
Step 2: Checkout mã nguồn (3s)
==============================================================================
Run actions/checkout@v5
Syncing repository: username/spring-boot-gradle-app
Getting Git version info
Copying '/home/runner/.gitconfig' to '/home/runner/work/_temp/.gitconfig'
Temporary credentials for 'https://github.com' configured
Fetching the repository
[command]/usr/bin/git -c protocol.version=2 fetch --no-tags --prune --progress --no-recurse-submodules --depth=1 origin +refs/heads/main:refs/remotes/origin/main
[command]/usr/bin/git checkout --progress --force -B main refs/remotes/origin/main
Already on 'main'
Branch 'main' set up to track remote branch 'main' from 'origin'.

==============================================================================
Step 3: Thiết lập môi trường Java JDK 17 (8s)
==============================================================================
Run actions/setup-java@v5
  with:
    java-version: 17
    distribution: temurin
Downloading https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.10%2B7/OpenJDK17U-jdk_x64_linux_hotspot_17.0.10_7.tar.gz
Extracting Java JDK...
Setting JAVA_HOME to /opt/hostedtoolcache/Java_Temurin-Hotspot_jdk/17.0.10-7/x64
Setting PATH environment variable
$ java -version
openjdk version "17.0.10" 2024-01-16
OpenJDK Runtime Environment Temurin-17.0.10+7 (build 17.0.10+7)
OpenJDK 64-Bit Server VM Temurin-17.0.10+7 (build 17.0.10+7, mixed mode, sharing)

==============================================================================
Step 4: Cấp quyền cho Gradle Wrapper (1s)
==============================================================================
Run chmod +x gradlew
[command]/usr/bin/chmod +x gradlew

==============================================================================
Step 5: Thực thi JUnit Test với Gradle (32s)
==============================================================================
Run ./gradlew test
Downloading https://services.gradle.org/distributions/gradle-8.5-bin.zip
.........10%.........20%.........30%.........40%.........50%.........60%.........70%.........80%.........90%.........100%
Starting a Gradle Daemon (subsequent builds will be faster)
> Task :compileJava
> Task :processResources
> Task :classes
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test

ApplicationTests > contextLoads() PASSED
UserServiceTest > shouldCreateUserSuccessfully() PASSED
UserServiceTest > shouldThrowExceptionWhenUserNotFound() PASSED

BUILD SUCCESSFUL in 30s
4 actionable tasks: 4 executed
JUnit Tests: 3 passed, 0 failed, 0 skipped.

==============================================================================
Step 6: Complete job (1s)
==============================================================================
Job status: SUCCESS ✔️
Total duration: 48s
```
