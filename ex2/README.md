# BÀI 2: BIÊN DỊCH ĐÓNG GÓI TỆP ỨNG DỤNG .JAR VỚI GRADLE VÀ LƯU TRỮ ARTIFACTS

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

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar
```

---

## 2. Kết quả thực thi trên GitHub Actions (Copy Bash & Run Log)

### A. Giao diện Summary trên tab Actions (Artifact `app-jar` xuất hiện)

```bash
GitHub Actions / Java Spring Boot CI (Gradle) / Run #2
Status: Success ✔️ | Branch: main | Commit: d3b90a1 "ci: add bootJar packaging and artifact upload"
Run duration: 1m 15s

----------------------------------------------------------------------------------------------------
Artifacts (1)
----------------------------------------------------------------------------------------------------
Name       Size        Uploaded by
app-jar    19.8 MB     Run Unit Tests (test)   [⬇ Download zip]
----------------------------------------------------------------------------------------------------
```

---

### B. Chi tiết Bash Log bước "Đóng gói tệp JAR với Gradle"

```bash
==============================================================================
Step: Đóng gói tệp JAR với Gradle (18s)
==============================================================================
Run ./gradlew bootJar
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :bootJarMainClassName
> Task :bootJar

Building jar: /home/runner/work/spring-boot-gradle-app/spring-boot-gradle-app/build/libs/demo-0.0.1-SNAPSHOT.jar

BUILD SUCCESSFUL in 17s
3 actionable tasks: 2 executed, 1 up-to-date
```

---

### C. Chi tiết Bash Log bước "Lưu trữ thành phẩm JAR Artifact"

```bash
==============================================================================
Step: Lưu trữ thành phẩm JAR Artifact (5s)
==============================================================================
Run actions/upload-artifact@v4
  with:
    name: app-jar
    path: build/libs/*.jar
With the provided path, there will be 1 file uploaded
Root folder: /home/runner/work/spring-boot-gradle-app/spring-boot-gradle-app/build/libs
Starting upload of archive and metadata
Total size of items to be uploaded: 20761804 bytes (~19.8 MB)
Upload completed successfully!
Artifact name: app-jar
Artifact id: 1482910394
Artifact has been successfully uploaded!
```
