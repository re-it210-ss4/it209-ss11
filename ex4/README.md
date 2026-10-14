# BÀI 4: KHAI BÁO GITHUB SECRETS VÀ CẤU HÌNH ĐĂNG NHẬP CONTAINER REGISTRY

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

      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

---

## 2. Xác thực cấu hình GitHub Secrets (Copy Bash & Web UI)

### A. Kiểm tra danh sách Repository Secrets qua GitHub CLI (`gh secret list`)

```bash
$ gh secret list --repo username/spring-boot-gradle-app

NAME              UPDATED
REGISTRY_TOKEN    about 2 minutes ago
```

### B. Giao diện Web: Settings -> Secrets and variables -> Actions

```bash
----------------------------------------------------------------------------------------------------
Repository secrets
Secrets are environment variables that are encrypted. Anyone with collaborator access to this 
repository can use these secrets for Actions.

[New repository secret]

Name               Updated
REGISTRY_TOKEN     2 minutes ago                                              [Update]  [Remove]
----------------------------------------------------------------------------------------------------
```

---

## 3. Kết quả thực thi trên GitHub Actions (Copy Bash Log)

### Chi tiết log bước "Đăng nhập vào GitHub Container Registry" (Login Succeeded)

```bash
==============================================================================
Step: Đăng nhập vào GitHub Container Registry (3s)
==============================================================================
Run docker/login-action@v3
  with:
    registry: ghcr.io
    username: dev-user
    password: ***
    logout: true
Logging into ghcr.io...
[command]/usr/bin/docker login -u dev-user --password-stdin ghcr.io
WARNING! Your password will be stored unencrypted in /home/runner/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded!
Post job cleanup: Logging out from registry ghcr.io
[command]/usr/bin/docker logout ghcr.io
Removing login credentials for ghcr.io
```

---

## 4. Hướng dẫn các bước tạo Personal Access Token & Secret trên GitHub

1. **Tạo GitHub Personal Access Token (classic)**:
   - Truy cập **GitHub** -> Nhấn vào Avatar góc trên cùng bên phải -> Chọn **Settings**.
   - Cuộn xuống dưới cùng cột bên trái, chọn **Developer settings** -> **Personal access tokens** -> **Tokens (classic)**.
   - Nhấn **Generate new token** -> **Generate new token (classic)**.
   - Đặt Note (ví dụ: `GHCR Push Token`), tích chọn scope:
     - `write:packages` (Ghi và tải gói lên GitHub Packages / GHCR).
     - `read:packages` (Tự động được tích kèm).
     - `delete:packages` (Tùy chọn).
   - Nhấn **Generate token** và sao chép mã token (dạng `ghp_...`).

2. **Cấu hình Secret trong Repository**:
   - Truy cập Repository của bạn trên GitHub.
   - Vào tab **Settings** -> Mục **Security** -> chọn **Secrets and variables** -> **Actions**.
   - Tại phần **Repository secrets**, nhấn nút **New repository secret**.
   - **Name**: Nhập chính xác `REGISTRY_TOKEN`.
   - **Secret**: Dán mã token vừa tạo vào.
   - Nhấn **Add secret**.
