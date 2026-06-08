# DevSecOps Homework 2

本專案為「系統開發與作業安全實務 Practices of DevSecOps」作業二。

GitHub Repository：

https://github.com/Practicesdevsecops/DevSecOps-HW2

## 專案簡介

本專案使用 Java 與 Spring Boot 建立一個簡單的 REST API，並透過 GitHub Actions 建立自動化 DevSecOps Pipeline。

每次程式碼 push 到 GitHub 的 `main` branch 時，Pipeline 會自動執行：

1. 自動化測試
2. Maven 打包
3. Dependency vulnerability scan
4. Secret scan
5. 上傳測試、JAR 與安全掃描報告

## 使用技術

* Java 17
* Spring Boot 3.5.14
* Maven
* JUnit 5
* MockMvc
* GitHub Actions
* Trivy

## 應用程式功能

本專案提供以下 REST API：

`GET /hello`

回傳結果：

`{"message":"Hello DevSecOps"}`

## 本機執行

啟動應用程式：

`mvn spring-boot:run`

啟動後可透過以下網址存取：

`http://localhost:8080/hello`

也可以使用 curl：

`curl http://localhost:8080/hello`

## 執行自動化測試

`mvn clean test`

本專案包含兩個測試：

* `DevsecopsHw2ApplicationTests`：確認 Spring Boot Application Context 能正常啟動。
* `HelloControllerTest`：確認 `/hello` 回傳 HTTP 200，且 JSON 中的 message 為 `Hello DevSecOps`。

正常結果：

`Tests run: 2, Failures: 0, Errors: 0, Skipped: 0`

## Maven 打包

`mvn clean package`

成功後會產生：

`target/devsecops-hw2-0.0.1-SNAPSHOT.jar`

## DevSecOps Pipeline

Workflow 檔案位於：

`.github/workflows/ci.yml`

Pipeline 包含三個 Jobs。

### 1. Build and Test

* Checkout source code
* 設定 Java 17
* 執行 Maven automated tests
* 將 Spring Boot 應用程式打包成 JAR
* 上傳 JAR 與 Maven Surefire 測試報告

### 2. Trivy Dependency Vulnerability Scan

* 掃描 Maven dependencies
* 檢查 HIGH 與 CRITICAL vulnerabilities
* 產生 `trivy-vulnerability-report.txt`
* 將報告上傳為 GitHub Actions artifact
* 發現 HIGH 或 CRITICAL vulnerability 時，Pipeline 會失敗

### 3. Trivy Secret Scan

* 掃描 repository 中可能存在的 password、API key、token 或其他 credentials
* 產生 `trivy-secret-report.txt`
* 將報告上傳為 GitHub Actions artifact
* 發現 hardcoded secret 時，Pipeline 會失敗

## Pipeline 驗證過程

### 第一次執行：成功

原始程式與測試內容一致，因此以下工作均成功：

* Build and Test
* Trivy Dependency Vulnerability Scan
* Trivy Secret Scan

### 第二次執行：故意失敗

將 `HelloControllerTest` 的預期訊息故意從：

`Hello DevSecOps`

修改成：

`Wrong Message`

實際 API 回傳內容與測試預期內容不同，因此 GitHub Actions 在自動化測試階段偵測到錯誤，`Build and Test` Job 失敗。

由於兩個 Trivy Jobs 依賴 `Build and Test`，因此後續安全掃描被停止。

### 第三次執行：修正後成功

將測試期待值修正回：

`Hello DevSecOps`

修正後，兩個自動化測試、應用程式打包以及兩項安全掃描均再次成功。

## Dependency Vulnerability 修補

Trivy 曾在 `tomcat-embed-core 10.1.54` 中偵測到：

* HIGH：3
* CRITICAL：3
* Total：6

掃描報告指出修補版本為 `10.1.55`。

因此在 `pom.xml` 中加入：

`<tomcat.version>10.1.55</tomcat.version>`

修補後重新執行測試與 Trivy 掃描，結果為：

`Vulnerabilities: 0`

確認已修補所有偵測到的 HIGH 與 CRITICAL dependency vulnerabilities。

## GitHub Actions Artifacts

每次成功執行 Pipeline 後，會產生三份 Artifacts：

### build-and-test-results

包含：

* Spring Boot JAR
* Maven Surefire test reports

### trivy-vulnerability-report

包含：

* `trivy-vulnerability-report.txt`

### trivy-secret-report

包含：

* `trivy-secret-report.txt`

Artifacts 保存期限設定為 14 天。

## 最終安全閘門

目前 Pipeline 會阻擋以下問題：

* 自動化測試失敗
* 發現 HIGH 或 CRITICAL dependency vulnerability
* 發現 hardcoded secret

因此 Pipeline 可以在程式碼進入後續流程前，自動發現功能錯誤與安全問題。
