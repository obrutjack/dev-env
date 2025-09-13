-----

### 在 Apple Silicon Mac 上設定 Apache Kafka 開發與使用環境

這是一份簡單明瞭的指南，引導你如何在搭載 **Apple Silicon 晶片**的 Mac 上，建立一個完整的 **Apache Kafka 開發環境**。

-----

### 1\. 準備開發工具

請先在你的 Mac 上安裝以下軟體：

  * **[Visual Studio Code](https://code.visualstudio.com/#:~:text=Download,-for%20macOS)**
      * 這將會是我們主要的程式開發環境。
  * **[OrbStack](https://orbstack.dev/download/stable/latest/arm64)**
      * OrbStack 是一個輕量且高效能的 Docker Desktop 替代品，非常適合在 Mac 上使用。
      * **提醒：** OrbStack 對於個人與開源專案免費，但商業用途需購買授權。

-----

### 2\. 安裝 VS Code 延伸套件

啟動 Visual Studio Code 後，請安裝以下幾個重要的延伸套件：

  * **[Extension Pack for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)**
    > 安裝後，你可以透過其內建的「Getting Started」教學來安裝 JDK。在 **"Get your runtime ready"** 區塊點選 **Install JDK**，並下載你需要的 JDK 版本。
    > **提醒：** Apache Kafka 建議使用 **JDK 17 或以上**的版本。
  * **[YAML Language Support](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)**
  * **[Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)**

-----

### 3\. 建立專案與下載原始碼

開啟你的**終端機 (Terminal)**，並依照以下步驟設定你的專案目錄：

```bash
# 建立 Apache Kafka 專案目錄
mkdir apache-kafka

# 建立用於 Docker 的環境目錄
mkdir -p apache-kafka/dev-env

# 切換到專案目錄
cd apache-kafka

# 複製 Kafka 官方原始碼
git clone https://github.com/apache/kafka.git
```

-----

### 4\. 編譯 Kafka 原始碼

進入 Kafka 專案目錄，並使用 **Gradle** 編譯，以產生後續服務所需的 `.jar` 檔案：

```bash
# 切換到 kafka 原始碼目錄
cd kafka

# 編譯 kafka 專案
./gradlew jar
```

-----

### 5\. 啟動 Kafka 服務

請將你的 `docker-compose.yaml` 檔案放置到 `apache-kafka/dev-env` 目錄中，然後執行以下指令來啟動服務：

```bash
# 切換到 docker-compose 檔案所在的目錄
cd ../dev-env

# 啟動 Kafka 與 Zookeeper 容器
docker compose up -d
```

服務成功啟動後，你可以在瀏覽器中開啟 `localhost:8080` 來查看 Kafka UI。

-----

### 6\. 開發與更新流程說明

#### 程式碼同步與 Volume 設定

為了讓你的本機程式碼修改能夠同步到容器中，你的 `docker-compose.yaml` 檔案必須設定 **Volume 掛載**。以下是相關的設定範例：

```yaml
volumes:
      - ../kafka/core:/opt/kafka/core  # 掛載編譯後的 Kafka Core
      - ../kafka/clients:/opt/kafka/clients # 掛載編譯後的 Kafka Clients
```

#### 開發流程邏輯

1.  **啟動服務：** 第一次執行 `docker-compose up -d` 時，Docker 會根據 `docker-compose.yaml` 啟動容器，此時 Kafka 運行的是映像檔內建的程式碼。
2.  **編譯程式碼：** 當你在 VS Code 中修改了 Kafka 原始碼，執行 `./gradlew jar` 會將你的新程式碼編譯成新的 `.jar` 檔案。
3.  **檔案同步：** 由於 **Volume** 的設定，你的新 `.jar` 檔案會自動同步到容器中，覆蓋舊的檔案。
4.  **重啟服務以生效：** 雖然檔案已同步，但運行的服務仍在舊程式碼上。你必須**重啟** Kafka 服務，它才會載入新的 `.jar` 檔案。

-----

### 7\. 重啟服務

在完成程式碼修改並重新編譯後，使用以下指令來重啟 **Kafka 容器**：

```bash
docker-compose up -d --force-recreate kafka
```

**資料來源：** [Gemini]
