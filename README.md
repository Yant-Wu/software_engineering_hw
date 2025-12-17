# 健康生活管家 Software Design Document (SDD)

## 1. Introduction

- **Purpose**
  本文件為健康生活管家應用程式的軟體設計文件（SDD），旨在詳細闡述系統架構、功能模組、資料庫設計、技術棧、非功能需求、測試策略與部署運維，作為開發、測試、維護的技術參考基準。

- **Scope**  
  系統提供 AI 智能健康飲食管理、食物掃描辨識與營養分析、個人化菜單推薦與購物清單生成、每日飲食追蹤與建議、運動與睡眠整合、提醒與通知等；涵蓋使用者資料管理、餐食資料庫、圖像辨識與 NLP、行為分析、報表與目標管理。

- **References**
  - 公開營養資料來源（如衛福部食物營養成分資料集）  
  - 作業系統與框架官方文件（Android/iOS、Node.js、Python、PostgreSQL）

## 2. System Overview

- **System Description**  
  健康生活管家是一款以 AI 為核心的行動應用程式，透過使用者的健康資料與飲食行為進行分析，提供個人化餐單、即時掃描辨識食物營養、每日飲食追蹤與建議、並生成改善建議與目標管理。系統包含行動端 App、後端服務、資料庫、AI 模型、第三方 API 整合，支援多語與本地化。

- **Design Goals**  
  - 精準個人化推薦（餐單、營養比例、改善建議）  
  - 高效食物辨識與營養計算  
  - 資料隱私與安全保護  
  - 高可用性與可擴展性  
  - 直覺易用的使用者介面  
  - 清晰任務模組化與可維護性

- **Architecture Summary**  
  採用分層式微服務架構：  
  - 客戶端（iOS/Android）：UI/UX、拍照掃描、同步、通知  
  - API Gateway：驗證、路由、速率限制  
  - 應用服務：使用者管理、餐單推薦、掃描辨識、追蹤與建議、報表  
  - AI 服務：圖像辨識、營養推論、個人化推薦模型  
  - 資料層：關聯式資料庫（PostgreSQL/MySQL）、文件儲存
  - 整合：第三方食材 API、推播服務

```mermaid
flowchart LR
  A["Mobile App (iOS/Android)"] --> B[API Gateway]
  B --> C1[User Service]
  B --> C2[Diet & Menu Service]
  B --> C3[Scan & Recognition Service]
  B --> C4[Tracking & Advice Service]
  B --> C5[Report Service]
  C2 --> D1[(Relational DB)]
  C3 --> D2[(Object Storage)]
  C2 --> D3[AI Recommendation]
  C3 --> D4[AI Vision]
  B --> E[Third-party APIs]
  B --> F[Auth/Rate Limit]
```

## 3. Architectural Design

- **Component Breakdown**  
  - 使用者管理（User Service）：帳號、個資、健康目標、過敏史、偏好、設定  
  - 餐食資料庫（Diet DB Service）：食材、營養成分、熱量、分類  
  - AI 推薦（AI Recommendation Service）：依個人健康狀況與目標生成菜單  
  - 食物掃描辨識（Vision Service）：相機掃描、影像分類、NLP  
  - 追蹤與建議（Tracking & Advice）：每日飲食記錄、趨勢分析、行為建議  
  - 通知與提醒（Notification）：餐時提醒、目標達成
  - 報表（Report）：週/月報表、KPI 指標、目標進度  
  - 系統管理（Admin）：權限、審計、設定、監控

- **Technology Stack**  
  - 前端：Flutter/React Native；支援 iOS/Android  
  - 後端：Node.js（NestJS/Express）或 Python（FastAPI）
  - AI：Python、PyTorch/TensorFlow；LLM/NLP 模型
  - 資料庫：PostgreSQL、Redis
  - 基礎設施：Docker、Kubernetes、CI/CD、API Gateway
  - 監控：Prometheus、Grafana、ELK、Sentry

```mermaid
graph TD
  subgraph Client
    M[Mobile UI]
  end
  subgraph Backend
    G[Gateway]
    U[User svc]
    D[Diet svc]
    R[Recommend svc]
    V[Vision svc]
    T[Tracking svc]
    N[Notification svc]
    P[Report svc]
  end
  subgraph Data
    DB[(PostgreSQL)]
    CA[(Redis)]
  end
  M-->G
  G-->U
  G-->D
  G-->R
  G-->V
  G-->T
  G-->N
  G-->P
  U-->DB
  D-->DB
  T-->DB
  R-->CA
```

- **Functional Requirements**
  - Task 1.1（用戶資料輸入介面）
  - Task 1.2（建立健康餐食資料庫）
  - Task 1.3（個性化菜單推薦）
  - Task 1.4（掃描食物辨識營養成分）
  - Task 1.5（追蹤與建議）
  - Task 1.6（生成購物清單）
  - Task 1.7（每日飲食數據追蹤）
  - Task 1.8（提醒與通知）
  - Task 1.9（報表）
  - Task 1.10（目標管理）
- **Non-Functional Requirements**
  - Task 2.1（性能優化）
  - Task 2.2（安全性與隱私保護）
  - Task 2.3（使用者體驗與易用性）
  - Task 2.4（可維護性）
  - Task 2.5（可擴展性）

## 4. Detailed Design

### 4.1 使用者與健康資料模組

- 任務對應：**Task 1.1（用戶資料輸入介面）**
- 功能：建立使用者資料（飲食習慣、健康目標、過敏史、身體狀況）、偏好設定
- 輸入：使用者表單資料；身高、體重、年齡、性別、活動量、疾病史
- 輸出：使用者設定檔；BMI 計算結果；偏好標籤
- 流程：
  1. UI 表單輸入 → 基本驗證
  2. 後端計算 BMI，生成健康目標建議
  3. 儲存至資料庫 `users`, `user_profiles`, `preferences`
- 驗收標準：
  - 必填欄位驗證與錯誤提示完整
  - 成功儲存與同步跨裝置
  - 建議值計算正確（對照公式）
- 類別圖：

```mermaid
---
config:
  layout: elk
---
flowchart TD
    Start([使用者開始註冊]) --> Input[輸入基本資料]
    Input --> ValidateBasic{資料格式正確?}
    
    ValidateBasic -->|否| ShowError1[顯示錯誤訊息]
    ShowError1 --> Input
    
    ValidateBasic -->|是| InputHealth[輸入健康目標]
    InputHealth --> SelectGoal[選擇目標類型]
    SelectGoal --> InputActivity[輸入活動量]
    
    InputActivity --> InputAllergy[輸入過敏史]
    InputAllergy --> InputPref[設定飲食偏好]
    
    InputPref --> ValidateComplete{必填欄位完整?}
    ValidateComplete -->|否| ShowError2[提示缺少欄位]
    ShowError2 --> InputHealth
    
    ValidateComplete -->|是| CalcBMI[計算BMI]
    CalcBMI --> CalcTDEE[計算每日建議熱量]
    CalcTDEE --> GenRecommend[生成初步建議]
    
    GenRecommend --> SaveDB[(儲存至資料庫)]
    SaveDB --> SyncCheck{需要跨裝置同步?}
    
    SyncCheck -->|是| SyncCloud[同步至雲端]
    SyncCloud --> ShowSuccess[顯示成功訊息]
    
    SyncCheck -->|否| ShowSuccess
    ShowSuccess --> End([完成註冊])
```

### 4.2 餐食資料庫模組

- 任務對應：**Task 1.2（建立健康餐食資料庫）**
- 功能：維護食材、營養成分、熱量與分類
- 輸入：食材資料；外部營養資料集
- 輸出：可查詢之食材/營養 API
- 流程：
  1. 資料匯入與清洗
  2. 建立索引與分類標籤（過敏、素食、低醣等）
  3. 提供查詢端點（以名稱/條碼/成分）
- 驗收標準：
  - 資料覆蓋率 ≥ 95% 常見食材
  - 查詢延遲 ≤ 200ms（快取）
  - 萬用過敏標籤一致性

```mermaid
classDiagram
    class FoodDatabaseModule {
        +String dbVersion
        +searchFood(String query)
        +getFoodById(String id)
        +importData(File csvFile)
        +updateNutrients()
        +addFoodItem()
    }
    
    class Food {
        +String foodId
        +String name
        +String category
        +NutrientProfile nutrients
        +List tags
        +Image image
        +getCalories()
        +checkAllergy()
    }
    
    class NutrientProfile {
        +float calories
        +float protein
        +float fat
        +float carbohydrates
        +float fiber
        +float sodium
        +float sugar
        +getPerUnit()
    }
    
    class Ingredient {
        +String ingredientId
        +String name
        +float defaultAmount
        +String unit
    }
    
    class Dish {
        +String dishId
        +String name
        +List ingredients
        +NutrientProfile totalNutrients
        +String description
        +calculateNutrients()
    }
    
    class Tag {
        +String tagName
        +String tagType
    }
    
    class DataImporter {
        +validateCSV()
        +parseData()
        +cleanData()
        +batchInsert()
    }
    
    class QueryIndex {
        +createIndex()
        +optimizeQuery()
        +cacheResult()
    }
    
    FoodDatabaseModule --> Food
    FoodDatabaseModule --> DataImporter
    FoodDatabaseModule --> QueryIndex
    Food --> NutrientProfile
    Food --> Tag
    Dish --> Ingredient
    Dish --> NutrientProfile
    Ingredient --|> Food
```

### 4.3 AI 個人化推薦模組

- 任務對應：**Task 1.3（個性化菜單推薦）、Task 1.5（追蹤與建議）**
- 功能：依使用者目標與狀況生成菜單；持續優化建議
- 輸入：使用者檔案、偏好、歷史飲食記錄、營養資料庫
- 輸出：日/週菜單、營養配比、熱量目標達成度
- 流程：
  1. 特徵工程（健康目標、過敏、偏好、活動量）
  2. 模型推論（Rule-based + ML Ranking）
  3. 生成菜單
  4. 追蹤每日記錄，調整建議
- 驗收標準：
  - 達成目標的推薦正確性（A/B 測試提升 ≥ 10%）
  - 避免過敏食材
  - 使用者滿意度調查達標

```mermaid
sequenceDiagram
    participant User
    participant UI
    participant AIModule as AI推薦模組
    participant UserProfile as 用戶資料
    participant FoodDB as 餐食資料庫
    participant Algorithm as 推薦演算法
    
    User->>UI: 請求產生菜單
    UI->>AIModule: requestMenu()
    AIModule->>UserProfile: getUserData()
    UserProfile-->>AIModule: 健康目標、過敏史、偏好
    
    AIModule->>AIModule: calculateTDEE()
    Note over AIModule: 計算每日建議熱量
    
    AIModule->>FoodDB: queryAvailableFoods(constraints)
    Note over FoodDB: 篩選符合條件的餐點
    FoodDB-->>AIModule: 候選食物清單
    
    AIModule->>Algorithm: generateMenuCombinations()
    Note over Algorithm: 組合菜單並排序
    Algorithm->>Algorithm: rankByObjective()
    Algorithm-->>AIModule: 推薦菜單 (top 3)
    
    AIModule->>AIModule: validateAllergens()
    AIModule-->>UI: 個性化菜單
    UI-->>User: 顯示推薦菜單
    
    alt 使用者接受
        User->>UI: 確認使用菜單
        UI->>AIModule: saveMenuSelection()
        AIModule->>UserProfile: recordSelection()
    else 使用者調整
        User->>UI: 修改菜單項目
        UI->>AIModule: adjustMenu()
        AIModule->>Algorithm: regenerate()
    end
```

### 4.4 食物掃描與營養辨識模組

- 任務對應：**Task 1.4（掃描食物辨識營養成分）**
- 功能：影像/條碼掃描，辨識食物與營養
- 輸入：相機影像、條碼文字、包裝標示 OCR
- 輸出：辨識食物名稱、營養成分、份量估計
- 流程：
  1. 影像前處理 → 分類/偵測模型
  2. 條碼解析 → 產品資料匹配
  3. OCR → 營養標示抽取 → NLP 正規化
  4. 輸入資料庫並關聯菜餚/食材
- 驗收標準：
  - 辨識準確率 ≥ 90%（常見食物集）
  - 條碼解析成功率 ≥ 95%
  - OCR/NLP 誤差率 ≤ 5%

```mermaid
stateDiagram-v2
    [*] --> 待機狀態
    
    待機狀態 --> 請求權限: 開啟掃描
    請求權限 --> 權限被拒: 權限拒絕
    權限被拒 --> [*]
    
    請求權限 --> 準備拍攝: 權限允許
    準備拍攝 --> 拍攝中: 按下快門
    拍攝中 --> 影像處理: 拍攝完成
    
    影像處理 --> 網路檢查: 前處理完成
    網路檢查 --> 離線模式: 無網路
    離線模式 --> 手動輸入
    
    網路檢查 --> AI辨識: 有網路
    AI辨識 --> 高信心結果: 信心分數 >= 0.9
    AI辨識 --> 中信心結果: 信心分數 0.6-0.9
    AI辨識 --> 低信心結果: 信心分數 < 0.6
    
    高信心結果 --> 顯示結果
    中信心結果 --> 候選清單: 顯示選項
    候選清單 --> 顯示結果: 使用者選擇
    低信心結果 --> 手動輸入: 辨識失敗
    
    手動輸入 --> 顯示結果: 搜尋完成
    
    顯示結果 --> 調整份量: 檢視結果
    調整份量 --> 調整份量: 修改份量
    調整份量 --> 儲存紀錄: 確認儲存
    
    儲存紀錄 --> 完成狀態: 儲存成功
    完成狀態 --> [*]
    
    note right of AI辨識
        根據信心分數
        分為三個路徑
    end note
    
    note right of 調整份量
        使用者可持續
        調整直到滿意
    end note
```

### 4.5 購物清單與電商整合模組

- 任務對應：**Task 1.6（生成購物清單）**
- 功能：依菜單生成可購買之食材清單；庫存與替代品
- 輸入：菜單、家中庫存、偏好（品牌/價格）
- 輸出：購物清單（分店/電商）、替代建議
- 流程：
  1. 菜單→食材分解→合併週期需求
  2. 對照庫存與替代品表
  3. 產出清單並可一鍵加入電商購物車（API）
- 驗收標準：
  - 清單覆蓋率 ≥ 98%
  - 替代品建議符合過敏/偏好約束

```mermaid
---
config:
  layout: elk
---
flowchart TD
    Start([使用者選擇菜單]) --> ParseMenu[解析菜單內容]
    ParseMenu --> ExtractIngredients[提取所需食材]
    
    ExtractIngredients --> MergeItems[合併重複食材]
    MergeItems --> UnitConversion[單位標準化換算]
    
    UnitConversion --> CheckInventory{檢查庫存?}
    CheckInventory -->|是| LoadInventory[載入庫存資料]
    LoadInventory --> CompareStock[比對庫存量]
    CompareStock --> CalculateNeeded[計算實際需求量]
    CalculateNeeded --> Classify
    
    CheckInventory -->|否| Classify[智能分類食材]
    
    Classify --> CategoryLoop{處理所有類別?}
    CategoryLoop -->|否| AssignCategory[分配至適當類別]
    AssignCategory --> CheckSubstitute{需要替代品?}
    
    CheckSubstitute -->|是| FindAlternatives[查詢替代食材]
    FindAlternatives --> RankSubstitutes[依偏好排序]
    RankSubstitutes --> AddSuggestions[加入建議清單]
    AddSuggestions --> CategoryLoop
    
    CheckSubstitute -->|否| CategoryLoop
    
    CategoryLoop -->|是| GenerateList[生成購物清單]
    GenerateList --> UserReview[使用者檢視]
    
    UserReview --> UserAction{使用者操作}
    
    UserAction -->|編輯項目| ModifyItem[修改清單項目]
    ModifyItem --> UserReview
    
    UserAction -->|分享清單| ShareOption{選擇分享方式}
    ShareOption -->|連結| GenLink[生成分享連結]
    GenLink --> SetupSync[設定即時同步]
    SetupSync --> NotifyCollaborators[通知協作者]
    NotifyCollaborators --> SaveList
    
    ShareOption -->|匯出| ExportFormat{選擇匯出格式}
    ExportFormat -->|PDF| ExportPDF[匯出為PDF]
    ExportFormat -->|Excel| ExportExcel[匯出為Excel]
    ExportFormat -->|文字| ExportText[匯出為純文字]
    ExportPDF --> SaveList
    ExportExcel --> SaveList
    ExportText --> SaveList
    
    UserAction -->|連接電商| SelectEcommerce[選擇電商平台]
    SelectEcommerce --> APIConnect[連接API]
    APIConnect --> SyncPrice[同步價格資訊]
    SyncPrice --> AddToCart[加入購物車]
    AddToCart --> SaveList
    
    UserAction -->|確認完成| SaveList[(儲存購物清單)]
    SaveList --> End([完成])
```

### 4.6 飲食追蹤與建議模組

- 任務對應：**Task 1.7（每日飲食數據追蹤）、Task 1.5（改善建議）**
- 功能：記錄每日飲食、營養匯總、趨勢分析、健康提醒
- 輸入：手動記錄、掃描辨識結果、第三方裝置資料
- 輸出：每日/週/月營養與熱量報表；建議
- 流程：
  1. 記錄事件（早餐/午餐/晚餐/零食）
  2. 營養匯總與目標對齊
  3. 偏差分析 → 推送建議/提醒
- 驗收標準：
  - 記錄完成率與一致性
  - 建議對偏差具體有效（回歸/迴圈測試）
  - 通知推送可靠

```mermaid
classDiagram
    class DietTrackingModule {
        +String userId
        +Date trackingDate
        +recordMeal(MealRecord meal)
        +getDailyStats()
        +getWeeklyTrend()
        +generateSuggestions()
    }
    
    class MealRecord {
        +String recordId
        +String mealType
        +Date timestamp
        +List foods
        +Image photo
        +NutritionSummary nutrition
        +calculateTotal()
    }
    
    class NutritionSummary {
        +float totalCalories
        +float protein
        +float fat
        +float carbohydrates
        +float fiber
        +float sodium
        +compareWithGoal()
    }
    
    class DailyLog {
        +Date date
        +List meals
        +float waterIntake
        +float exercise
        +getDailyTotal()
        +checkGoalAchievement()
    }
    
    class TrendAnalyzer {
        +analyzeWeekly()
        +analyzeMonthly()
        +detectPatterns()
        +identifyIssues()
    }
    
    class SuggestionEngine {
        +List rules
        +generateAdvice()
        +prioritizeSuggestions()
        +checkUserPreferences()
    }
    
    class GoalManager {
        +HealthGoal currentGoal
        +float targetCalories
        +MacroRatio targetRatio
        +adjustGoal()
        +trackProgress()
    }
    
    class ReportGenerator {
        +generateDailyReport()
        +generateWeeklyReport()
        +generateMonthlyReport()
        +exportData()
    }
    
    DietTrackingModule --> MealRecord
    DietTrackingModule --> DailyLog
    DietTrackingModule --> TrendAnalyzer
    DietTrackingModule --> SuggestionEngine
    DietTrackingModule --> GoalManager
    DietTrackingModule --> ReportGenerator
    MealRecord --> NutritionSummary
    DailyLog --> MealRecord
    TrendAnalyzer --> DailyLog
    SuggestionEngine --> TrendAnalyzer
    ReportGenerator --> DailyLog
```

### 4.7 通知與提醒模組

- 任務對應：**Task 1.8（提醒與通知）**
- 功能：餐時提醒、補充水分、達標提示、購物提醒
- 輸入：使用者目標、行程、時區
- 輸出：App 內通知、推播
- 流程：
  1. 設定排程與條件
  2. 推播服務
  3. 點擊行動（開啟菜單/清單/記錄）
- 驗收標準：
  - 準時送達率 ≥ 99%
  - 不重複或騷擾（頻率控制與退訂）
  - 行動轉換率 KPI 追蹤

```mermaid
stateDiagram-v2
    [*] --> 未排程
    
    未排程 --> 已排程: 建立通知排程
    已排程 --> 等待觸發: 條件設定完成
    
    等待觸發 --> 檢查條件: 時間到達
    檢查條件 --> 準備發送: 條件滿足
    檢查條件 --> 等待觸發: 條件不滿足
    
    準備發送 --> 發送中: 開始推播
    發送中 --> 已送達: 推播成功
    發送中 --> 發送失敗: 網路錯誤/裝置離線
    
    發送失敗 --> 等待重試: 設定重試
    等待重試 --> 發送中: 重試時間到
    等待重試 --> 已取消: 超過重試次數
    
    已送達 --> 已讀取: 使用者點擊
    已送達 --> 已過期: 超過有效期
    
    已讀取 --> 已處理: 執行動作
    已讀取 --> 已關閉: 忽略通知
    
    已排程 --> 已取消: 使用者取消
    等待觸發 --> 已取消: 使用者取消
    準備發送 --> 已取消: 使用者取消
    
    已處理 --> [*]
    已關閉 --> [*]
    已過期 --> [*]
    已取消 --> [*]
    
    note right of 檢查條件
        條件包括：
        - 使用者偏好時間
        - 免打擾時段
        - 目標達成狀態
    end note
    
    note right of 發送失敗
        最多重試3次
        間隔時間遞增
    end note
```

### 4.8 報表與目標管理模組

- 任務對應：**Task 1.9（報表）、Task 1.10（目標管理）**
- 功能：週/月報表、KPI 指標、目標設定與進度追蹤
- 輸入：歷史記錄、目標、偏好
- 輸出：圖表（熱量、營養價值、達標率）、目標狀態、建議
- 流程：
  1. 聚合計算與指標生成
  2. 視覺化（圖表元件）
  3. 目標與提醒耦合
- 驗收標準：
  - 指標計算正確（交叉驗證）
  - 圖表載入 < 1s（快取）
  - 使用者可操作性良好

```mermaid
sequenceDiagram
    participant User
    participant UI
    participant ReportModule as 報表模組
    participant DataStore as 資料儲存
    participant ChartGen as 圖表生成器
    participant GoalTracker as 目標追蹤器
    participant Export as 匯出服務
    
    User->>UI: 請求查看報表
    UI->>ReportModule: generateReport(type, dateRange)
    
    ReportModule->>DataStore: queryMealRecords(dateRange)
    DataStore-->>ReportModule: 飲食記錄清單
    
    ReportModule->>DataStore: queryHealthMetrics(dateRange)
    DataStore-->>ReportModule: 健康指標數據
    
    ReportModule->>ReportModule: aggregateData()
    Note over ReportModule: 彙整統計數據
    
    ReportModule->>GoalTracker: getGoalProgress()
    GoalTracker->>GoalTracker: calculateProgress()
    GoalTracker-->>ReportModule: 目標達成度
    
    ReportModule->>ChartGen: createCharts(data)
    ChartGen->>ChartGen: generateLineChart()
    ChartGen->>ChartGen: generateBarChart()
    ChartGen->>ChartGen: generatePieChart()
    ChartGen-->>ReportModule: 圖表集合
    
    ReportModule->>ReportModule: formatReport()
    ReportModule-->>UI: 完整報表
    UI-->>User: 顯示報表內容
    
    alt 使用者匯出報表
        User->>UI: 點擊匯出
        UI->>ReportModule: exportReport(format)
        ReportModule->>Export: export(data, format)
        
        alt PDF格式
            Export->>Export: generatePDF()
            Export-->>ReportModule: PDF檔案
        else Excel格式
            Export->>Export: generateExcel()
            Export-->>ReportModule: Excel檔案
        else 圖片格式
            Export->>Export: generateImage()
            Export-->>ReportModule: 圖片檔案
        end
        
        ReportModule-->>UI: 下載連結
        UI-->>User: 提供下載
    end
    
    alt 調整目標
        User->>UI: 修改健康目標
        UI->>GoalTracker: updateGoal(newGoal)
        GoalTracker->>DataStore: saveGoal()
        DataStore-->>GoalTracker: 儲存成功
        GoalTracker-->>UI: 目標已更新
        UI-->>User: 顯示確認訊息
    end
```

## 5. Database Design

```mermaid
---
config:
  layout: elk
---
erDiagram
    users ||--o| user_profiles : "擁有個人檔案"
    users ||--o| preferences : "設定偏好"
    users ||--o{ menus : "建立菜單"
    users ||--o{ shopping_lists : "產生清單"
    users ||--o{ consumption_logs : "記錄飲食"
    users ||--o{ recommendations : "獲得推薦"
    users ||--o{ goals : "設定目標"
    users ||--o{ reports : "查看報告"
    users ||--o{ notifications : "收到通知"
    
    dishes ||--o{ consumption_logs : "記錄為飲食"
    dishes }o--o{ menus : "加入菜單"
    ingredients }o--o{ consumption_logs : "記錄食材"
    menus ||--o{ recommendations : "產生推薦"
    menus ||--o{ shopping_lists : "生成清單"
    
    users {
        int id
        string email
        string password_hash
        string status
        datetime created_at
    }
    
    user_profiles {
        int user_id
        float height
        float weight
        int age
        string gender
        string activity_level
        json allergies
        json medical_history
        float bmi
    }
    
    preferences {
        int user_id
        string diet_type
        json cuisine_prefs
        string budget_range
        json notification_settings
    }
    
    ingredients {
        int id
        string name
        string barcode
        json allergens
        json nutrition
        json tags
    }
    
    dishes {
        int id
        string name
        string category
        json recipe
        json nutrition
        float calories
        json tags
    }
    
    menus {
        int id
        int user_id
        date date
        json items
        float total_calories
        json macro_ratio
    }
    
    shopping_lists {
        int id
        int user_id
        int menu_id
        string week
        json items
        string status
    }
    
    consumption_logs {
        int id
        int user_id
        datetime time
        string meal_type
        string source
        int dish_id
        json ingredient_ids
        float portion
        json nutrition
    }
    
    recommendations {
        int id
        int user_id
        date date
        json context
        int menu_id
        float score
        string type
    }
    
    goals {
        int id
        int user_id
        string type
        float target_value
        date start_date
        date end_date
        string status
        float current_value
    }
    
    reports {
        int id
        int user_id
        string period
        json kpis
        text summary
        datetime generated_at
    }
    
    notifications {
        int id
        int user_id
        string type
        json payload
        datetime scheduled_at
        datetime sent_at
        string status
    }
```

## 6. Non-Functional Requirements

- **性能（Performance）**
  - 主要 API 延遲 ≤ 300ms；掃描辨識端到端 ≤ 2s
  - 每日活躍 10k 用戶下維持 ≥ 99.9% 可用性
  - 資料庫查詢優化與索引完整；批次作業離峰執行

- **安全（Security）**
  - OAuth2/OpenID Connect；JWT；雙因素驗證（選擇性）
  - 靜態/傳輸加密（AES-256、TLS 1.2+）；秘密管理（Vault/KMS）
  - RBAC 權限；審核日誌；GDPR/CCPA 合規；隱私遮罩

- **可用性（Usability**）
  - 無障礙 AA 級（文字放大、色彩對比、語音輔助）
  - 專注於低認知負擔之流程與介面設計
  - 離線模式支援基本功能（資料緩存、同步）
  - 國際化與在地化（繁中優先）

- **維護性（Maintainability）**
  - 程式碼規範（ESLint/Black）；單元測試覆蓋率 ≥ 80%
  - 版本化管理
  - 自動化 CI/CD 流程
  - 集中化日誌與監控（ELK、Prometheus/Grafana）

- **可擴展性（Scalability）**
  - 水平擴展（K8s HPA）
  - 快取與排隊（Redis/Queue）；影像與模型分離部署
  - 分割讀寫與資料庫分片

```mermaid
mindmap
  root(("非功能性需求"))
    性能優化
      快取
      索引
      背景作業
    安全
      OAuth2
      加密
      RBAC
    可用性
      i18n
      無障礙
      離線
    維護性
      日誌
      測試覆蓋
      CI/CD
    擴展性
      微服務
      事件總線
      水平擴展
```

## 7. Testing Strategy

- 測試層級
  - 單元測試：服務邏輯、模型推論、資料庫操作
  - 整合測試：API 合約、跨服務流程（推薦→清單→通知）
  - 端到端測試（E2E）：使用者場景（建立目標→掃描→生成菜單→追蹤→報表）
  - 性能測試：負載、壓力、延遲監控
  - 安全測試：權限驗證、資料洩漏檢查

- 測試案例
  1. 用戶資料建立與驗證  
     - 前置：新用戶  
     - 步驟：填寫資料→提交→查詢 profile  
     - 期望：必填驗證、BMI正確、資料持久化成功
  2. 菜單推薦避開過敏食材  
     - 前置：設定花生過敏  
     - 步驟：生成週菜單  
     - 期望：菜單中不含花生/其衍生物
  3. 食物掃描條碼解析  
     - 前置：已知產品條碼  
     - 步驟：掃描→比對資料庫  
     - 期望：返回正確產品與營養
  4. 購物清單生成與電商下單  
     - 前置：既有菜單與庫存  
     - 步驟：生成清單→推送到電商  
     - 期望：清單完整且下單 API 成功
  5. 每日飲食追蹤與建議  
     - 前置：連續三天高糖攝取  
     - 步驟：匯總→生成建議→通知  
     - 期望：建議具體、通知送達、可操作連結
  6. 報表與目標達成率  
     - 前置：一週完整記錄  
     - 步驟：生成週報表  
     - 期望：數據一致、載入迅速、圖表正確
  7. 非功能測試（性能/安全）  
     - 負載 1k ≤ 300ms  
     - 權限測試：未授權請求被拒
     - 資料加密驗證

## 8. Stakeholders & Team Structure

- **Stakeholders**

| 角色 | 職責與說明 |
|---|---|
| 系統使用者 | 注重健康飲食、希望透過 App 管理個人飲食習慣與健康的最終用戶。他們是系統服務的主要對象，其需求和反饋是專案成功的關鍵。 |
| 系統管理員 | 負責管理與維護 App 後台內容的人員。主要工作包含維護健康餐食資料庫、審核資訊、管理使用者回饋等，確保內容的正確性與即時性。 |
| 系統維護者 | 負責確保系統伺服器、資料庫和整體應用程式穩定運行的技術人員。他們處理系統部署、監控性能、進行例行維護與故障排除。 |
| 系統開發者 | 負責 App 所有功能開發與實作的團隊成員，包含前後端程式設計、AI 模型整合與資料庫建構等。他們需要將需求轉化為實際的應用程式功能。 |
| 客服人員 | 作為使用者與開發團隊之間的橋樑，負責收集、整理並回覆使用者的問題、建議與錯誤回報，協助提升使用者體驗。 |

```mermaid
---
config:
  layout: dagre
---
flowchart TD
    A[投資者]
    B[系統管理員]
    C[系統開發者]
    D[系統維護者]
    E[客服人員]
    F[系統使用者]
    
    A -->|提供資金與監督| B
    B -->|管理與協調| C
    B -->|管理與協調| D
    B -->|管理與協調| E
    C -->|開發系統| G[健康生活管家 App]
    D -->|維護系統| G
    E -->|支援服務| F
    G -->|提供服務| F
    F -->|使用回饋| E
    F -->|需求反饋| B
```

- **團隊成員與任務分工**
  - B1243017 吳彥廷
  - B1243009 陳彥儒
  - B1243005 曹哲維
  - B1243001 江珩安
  - B1243018 王翃陞

## 9. Risk Analysis

- 模型準確率不足影響推薦品質（緩解：持續擴建資料集與標註與 A/B 測試）
- OCR/NLP 在包裝標示不一致時錯誤率高（緩解：規則校正與增加人為審核介面）
- 電商 API 變更導致整合失效（緩解：API 版本化與回退機制）
- 隱私與法規風險（緩解：資料最小化、加密、合規審查）
- 系統高峰期性能瓶頸（緩解：水平擴展、快取、異步化）

## 10. Deployment & Operations

- 部署：容器化（Docker）、Kubernetes、分環境（Dev/Staging/Prod）
- 配置：集中化設定（ConfigMap/Secrets）、版本控管
- 監控：APM、指標、日誌；異常自動告警
- 災難復原：多區域備援、每日備份、RTO/RPO 目標
- 滾動更新；回滾機制

## 11. Appendices

- API 範例
  - 建立使用者偏好
    - POST /api/v1/users/{id}/preferences  
      請求：

      ```json
      {
        "diet_type": "low_carb",
        "cuisine_prefs": ["japanese", "mediterranean"],
        "notification_settings": {"meal_reminder": true}
      }
      ```

      回應：

      ```json
      {"status":"ok","updated":true}
      ```

  - 生成週菜單
    - POST /api/v1/menus/generate  
      請求：

      ```json
      {"user_id":123,"week":"2025-W01","constraints":{"allergies":["peanut"],"budget":50}}
      ```

      回應：

      ```json
      {"menu_id":456,"total_calories":14000,"macro_ratio":{"carb":0.45,"protein":0.25,"fat":0.30}}
      ```
