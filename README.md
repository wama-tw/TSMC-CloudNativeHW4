# TSMC-CloudNativeHW4

## Docker 相關說明

### 如何透過 docker build 打包你的應用程式

#### 指令說明

```bash
# 建立 Docker 映像檔
docker build -t . <image-name>:<tag>

# 為映像檔加上額外的標籤
docker tag <source-image>:<tag> <target-image>:<tag>

# 推送映像檔到 Docker Hub
docker push <image-name>:<tag>

# 實際使用範例
docker build -t wamatw/2025cloud:1.0 .
docker tag wamatw/2025cloud:1.0 wamatw/2025cloud:latest
docker push wamatw/2025cloud:latest
```

#### 自動化 Docker 打包與推送流程說明

這段程式碼會在 CI/CD 流程中自動執行以下步驟：

##### 1. 設定標籤變數 `TAG`

```yaml
- name: Set IMAGE_TAG env
  id: vars
  run: echo "TAG=$(date +%Y%m%d-%H%M%S)" >> $GITHUB_OUTPUT
```

* 建立一個時間戳記（格式為 `YYYYMMDD-HHMMSS`）作為 Docker 映像的標籤（tag）。
* 選擇以時間作為標記，方便辨認不同版本以及新舊的同時，也能快速 release 不同版本
* 儲存在變數 `steps.vars.outputs.TAG` 中，用來唯一識別每一次打包版本。

---

##### 2. 建立 Docker 映像檔

```yaml
- name: Build Docker image
  run: |
    docker build . -t ${{ secrets.DOCKER_USERNAME }}/2025cloud:${{ steps.vars.outputs.TAG }}
```

* 在目前目錄（`.）`下執行 `docker build`，根據 `Dockerfile` 打包應用程式。
* 使用剛剛產生的 `TAG` 當作版本標籤。
* 映像名稱為 `DOCKER_USERNAME/2025cloud:時間戳記TAG`。

---

##### 3. 推送映像檔到 Docker Hub

```yaml
- name: Push Docker image
  run: |
    docker push ${{ secrets.DOCKER_USERNAME }}/2025cloud:${{ steps.vars.outputs.TAG }}
    docker tag ${{ secrets.DOCKER_USERNAME }}/2025cloud:${{ steps.vars.outputs.TAG }} ${{ secrets.DOCKER_USERNAME }}/2025cloud:latest
    docker push ${{ secrets.DOCKER_USERNAME }}/2025cloud:latest
```

* 將剛剛建好的映像檔推送到 Docker Hub。
* 同時也將這個版本標記為 `latest`，表示這是最新的版本，方便部署使用。
* 這裡會執行兩次推送：一次是時間標籤的版本，一次是 `latest` 版本。

### 如何透過 docker run 運行你 Container Image 

```bash
docker run wamatw/2025cloud:latest
```

### 驗證運行狀態
有出現 `Hello, Docker Hub` 即為運行成功
```bash
Unable to find image 'wamatw/2025cloud:latest' locally
latest: Pulling from wamatw/2025cloud
Digest: sha256:72b601ae8b0395f7aedf83c6368d55570e4b1209266ce161919ff58ae2aeb091
Status: Downloaded newer image for wamatw/2025cloud:latest
Hello, Docker Hub
```

### 專案自動化產生 Container Image 的邏輯，以及 Tag 的選擇邏輯

#### 自動化產生 Container Image 邏輯

專案使用 GitHub Actions 實現自動化 CI/CD 流程，主要包含以下步驟：

1. **觸發條件**：
   - 當程式碼推送到主分支時
   - 當建立新的 Pull Request 時
   - 當手動觸發工作流程時

2. **自動化流程**：
   - 檢出程式碼
   - 設定 Docker 登入憑證
   - 產生時間戳記標籤
   - 建立 Docker 映像檔
   - 推送至 Docker Hub

#### Tag 選擇邏輯

標籤策略採用雙重標記機制：

1. **時間戳記標籤**：
   - 格式：`YYYYMMDD-HHMMSS`
   - 優點：
     - 提供精確的版本追蹤
     - 方便回溯特定時間點的版本
     - 確保每個版本都是唯一的

2. **Latest 標籤**：
   - 每次建立新的映像檔時，同時標記為 `latest`
   - 優點：
     - 方便部署最新版本
     - 符合 Docker 生態系統的常見實踐
     - 簡化部署流程

這種雙重標記策略既保證了版本追蹤的精確性，又提供了便利的部署選項。