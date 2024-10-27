# 臺灣天氣預報應用

## 概述

這個專案是一個使用 Vue 3 開發的臺灣天氣預報應用。它利用中央氣象署的開放資料 API 來獲取並顯示各縣市的天氣預報資訊。

## demo 網址

[vue3-yungching-interview-assignment](https://vue3-yungching-interview-assignment.vercel.app/)

## 使用技術

- Vue 3
- Vite
- SCSS
- 中央氣象署開放資料 API

## 功能特點

- 提供未來 36 小時的天氣預報，包括天氣狀況、溫度、降雨機率等
- 可顯示全臺各縣市的天氣預報
- 可選擇特定縣市，只查看該縣市之天氣預報
- 表格皆有做分頁功能
- 提供我的最愛功能，點選 checkbox 後，可批次將天氣預報資訊儲存下來
- 我的最愛頁面之內容皆可編輯跟刪除
- 本網站採用響應式網頁設計 (RWD)

## 專案設置

```sh
nvm use 18.17.0
npm ci
```

### 編譯並啟動開發伺服器

```sh
npm run dev
```

### 編譯並壓縮用於正式環境

```sh
npm run build
```

### 使用 ESLint 進行程式碼檢查

```sh
npm run lint
```

## 數據來源

本應用使用的天氣預報數據由[中央氣象署開放資料平台](https://opendata.cwa.gov.tw/)提供。

## 注意事項

此專案為了方便驗收成果，有將環境變數上傳至 github。
正常操作流程下，應從中央氣象署獲取有效的 API 金鑰，並自行將其設置在專案的環境變數中。
