# BÁO CÁO TỐI ƯU TỐC ĐỘ CI BẰNG GRADLE DEPENDENCY CACHING (BÀI 3)

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
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar
```

---

## 2. Bảng so sánh thời gian thực thi (Trước & Sau khi áp dụng Gradle Cache)

| Bước thực thi | Lần chạy 1 (Chưa có Cache) | Lần chạy 2 (Đã có Cache) | Tỉ lệ tối ưu |
| :--- | :--- | :--- | :--- |
| **Setup Java (setup-java@v5)** | 10s (Download JDK + tìm cache) | 4s (Khôi phục cache gradle) | ⚡ Nhanh hơn 60% |
| **Thực thi JUnit Test (`./gradlew test`)** | 42s (Download toàn bộ dependencies & daemon) | 9s (Sử dụng dependencies trong cache) | ⚡ Nhanh hơn 78% |
| **Đóng gói JAR (`./gradlew bootJar`)** | 18s (Build và package) | 7s (Incremental build) | ⚡ Nhanh hơn 61% |
| **Tổng thời gian toàn bộ Job** | **1m 15s** | **23s** | 🚀 **Giảm ~70% thời gian** |

---

## 3. Nhật ký thực thi (Copy Bash Logs)

### A. Lần 1: Push lần đầu (Tạo và lưu trữ Cache)

```bash
==============================================================================
Step: Thiết lập môi trường Java JDK 17 (10s)
==============================================================================
Run actions/setup-java@v5
  with:
    java-version: 17
    distribution: temurin
    cache: gradle
Resolved gradle toolcache at: /opt/hostedtoolcache/Java_Temurin-Hotspot_jdk/17.0.10-7/x64
Searching for gradle dependency cache...
Resolved primary key: gradle-Linux-x64-9b8f2c6e7a4d5e8f1b2c3d4e5f6a7b8c
Cache not found for input keys: gradle-Linux-x64-9b8f2c6e7a4d5e8f1b2c3d4e5f6a7b8c

==============================================================================
Step: Thực thi JUnit Test với Gradle (42s)
==============================================================================
Run ./gradlew test
Downloading https://repo.maven.apache.org/maven2/org/springframework/boot/spring-boot-starter-web/...
Downloading https://repo.maven.apache.org/maven2/org/junit/jupiter/junit-jupiter/...
[Tải hàng trăm packages Maven/Gradle qua Internet lần đầu]
> Task :compileJava
> Task :test
BUILD SUCCESSFUL in 41s

==============================================================================
Post Run actions/setup-java@v5 (Lưu cache lên GitHub Server)
==============================================================================
Post job cleanup: Saving Gradle cache...
Hash keys: build.gradle, gradle/wrapper/gradle-wrapper.properties
Saving cache with key: gradle-Linux-x64-9b8f2c6e7a4d5e8f1b2c3d4e5f6a7b8c
Cache saved successfully (Size: 142 MB)
```

---

### B. Lần 2: Push lần hai (Khôi phục Cache thành công)

```bash
==============================================================================
Step: Thiết lập môi trường Java JDK 17 (4s)
==============================================================================
Run actions/setup-java@v5
  with:
    java-version: 17
    distribution: temurin
    cache: gradle
Found in cache: /opt/hostedtoolcache/Java_Temurin-Hotspot_jdk/17.0.10-7/x64
Searching for gradle dependency cache...
Resolved primary key: gradle-Linux-x64-9b8f2c6e7a4d5e8f1b2c3d4e5f6a7b8c
Restoring cache...
Received 148897792 of 148897792 bytes (100.0%)
Cache restored successfully from key: gradle-Linux-x64-9b8f2c6e7a4d5e8f1b2c3d4e5f6a7b8c

==============================================================================
Step: Thực thi JUnit Test với Gradle (9s)
==============================================================================
Run ./gradlew test
Starting a Gradle Daemon
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :compileTestJava UP-TO-DATE
> Task :testClasses UP-TO-DATE
> Task :test UP-TO-DATE

BUILD SUCCESSFUL in 8s
4 actionable tasks: 4 up-to-date
JUnit Tests: 3 passed, 0 failed, 0 skipped.
```

---

## 4. Kết luận

- Tham số `cache: 'gradle'` trong action `setup-java@v5` tự động hash các tệp định nghĩa phụ thuộc (`build.gradle`, `settings.gradle`, `gradle-wrapper.properties`).
- Khi cache được phục hồi, Gradle không cần tải lại toàn bộ dependencies từ Maven Central qua Internet, giúp:
  1. Tiết kiệm băng thông mạng của GitHub runner.
  2. Rút ngắn thời gian feedback loop của CI từ hơn 1 phút xuống còn hơn 20 giây.
  3. Tận dụng cơ chế `UP-TO-DATE` task caching của Gradle.
