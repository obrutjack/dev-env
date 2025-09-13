這是一篇介紹如何在 MacOS with Apple Silicon 安裝要進行 Apache Kafka 開發作業的簡單指引

Step 1. 我們會需要在 Mac OS 上安裝以下套件：
# [Visual Studio Code](https://code.visualstudio.com/#:~:text=Download,-for%20macOS)
## 會以這個軟體為主要開發環境.
# [OrbStack] (https://orbstack.dev/download/stable/latest/arm64)
## 選擇這個是因為簡單方便, 唯一要注意的是不能在商業上使用.

Step 2. 安裝完以上軟體後, 請執行 Visual Studio Code 並安裝以下延伸套件:
# [Extension Pack for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)
## 安裝完 Extension Pack for Java 後可以透過 Walkthrough: Get Started with JAVA Development 來安裝 JDK 
### 請在 Get your runtime ready 點選 Install JDK 後下載你需要的 JDK 版本並安裝在 Mac OS 上 (Apache Kafka 建議 JDK 17 以上)
# [YAML Language Support](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)
# [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)

Step 3. 開啟 Terminal 建立要進行開發的目錄
```
# mkdir apache-kafka
# mkdir -p apache-kafka/dev-env
```

Step 4. 切換到該目錄後對官方原始碼庫進行複製到 Mac OS 上
```
# cd apache-kafka
# git clone https://github.com/apache/kafka.git
```

Step 5. 切換到 apache-kafka 原始碼目錄後即可進行編譯產生 JAR 檔
```
# cd kafka 
# ./gradlew jar 編譯 kafka
```

Step 6. 請將附檔 docker-compose.yaml 放入到 apache-kafka/dev-env 的目錄內就可以使用以下指令啟用 kafka
```
# cd apache-kafka/dev-env
# docker compose up -d
```

Step 7. 開啟 Safari, 連線到 localhost:8080 就可以看到 Kafka UI

最後說明:

# 檔案 docker-compose.yaml 裡面要修改掛在的 kafka core/clients volume 來自編譯的目錄
```=
volumes:
      - ../kafka/core:/opt/kafka/core  # 舉例: 掛載你的編譯核心
      - ../kafka/clients:/opt/kafka/clients # 舉例: 掛載你的編譯客戶端
```

# 整個開發流程的邏輯是這樣的：
1. 啟動服務： 第一次執行 docker-compose up -d 時，Docker 會根據 docker-compose.yaml 啟動 Zookeeper 和 Kafka 容器。此時，容器內的 Kafka 會使用其映像檔內建的程式碼來運行。
2. 編譯程式碼： 當你在 VS Code 中修改了 Kafka 的原始碼後，你執行 ./gradlew jar。這個命令會把你的新程式碼編譯成新的 .jar 檔案，並儲存在你主機的 build/libs 資料夾中。
3. 容器檔案同步： 由於 volumes 的設定 (../core:/opt/kafka/core)，你的新 .jar 檔案會自動同步到容器內的 /opt/kafka/core 資料夾，覆蓋掉舊的檔案。
4. 重啟服務以生效： 雖然檔案已經同步了，但正在運行的 Kafka 服務仍然使用舊的程式碼。你必須重啟Kafka 服務，才能讓它載入新的 .jar 檔案。

# 修改完成式並重新編譯完後, 可以使用下面指令重啟 kafka
```
# docker-compose up -d --force-recreate kafka
```

# 資料來源:
<Gemini>
