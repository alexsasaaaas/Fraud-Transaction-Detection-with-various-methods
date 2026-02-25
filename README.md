# Fraud Transaction Detection with various methods
**從資料洞察到可落地的風險分層系統**

這不是只把模型 train 好就結束的 Kaggle 專案。  
而是從 EDA 看到詐騙行為模式 → 快速試各種不平衡處理與 ensemble 模型 → 最後把機率轉成 **真的能用的風險分數與決策規則**。


### 專案快速概覽

- **目標**：在極不平衡的交易資料中，穩定抓到詐騙，同時讓風控團隊看得懂、好調整門檻。
- **資料**：Final Transactions.csv（約 175 萬筆，時間、客戶、金額等等全都有）。
- **核心方法**：根據Chen, T., Sun, R., Ma, T., & Sergeev, S. (2026) 的文獻內容，比較Ensemble 方法以及 Ensemble + Hybird 方法，實驗各種方法對於預估詐騙機率，並完整比較。
- **最終產出**：每筆交易一個 0~1 的 Risk Score，搭配 Low/Medium/High 分級 + 對應決策建議。

### 專案重點

1. **EDA.ipynb**  
   分析詐騙什麼時候、怎麼發生：  
   - 深夜交易是不是特別可疑？  
   - 同一客戶 5 分鐘內連刷好幾筆怎麼辦？  
   - 金額突然爆高或偏離客戶平均值有沒有用？

2. **final_ensemble+hybird.ipynb**  
   同一個 Stacking 架構（RF + XGBoost + LightGBM），比了四種不平衡資料處理方式：  
   - Baseline  
   - Class Weight  
   - SMOTE  
   - ADASYN  
   重點不是只看 F1，而是看「最佳門檻」跟「訓練時間」誰最實務。

3. **final_gbdt_ensemble.ipynb**  
   三個 GBDT 模型（XGBoost + LightGBM + CatBoost）做 soft voting。  
   結果：ROC-AUC 0.984、F1 0.978、最佳門檻 ≈0.81、訓練只要 10 分鐘。  

4. **conclusion.ipynb**  
   直接給你比較表：  
   - 哪個方法 F1 最高？  
   - 哪個門檻最穩定（不會調到 0.99 才好看）？  
   - 訓練時間差多少？  
   結論：**GBDT Ensemble 贏在實務性**。

5. **application.ipynb**  
   將預測模型套用在實務應用上。
   把模型機率 + 統計異常 + 行為異常 → 加權變成 Hybrid Risk Score（0~1）。  
   再切成 Low / Medium / High 三級：  
   - Low Risk：Fraud Rate ≈0.6% → 直接放行  
   - Medium Risk：Fraud Rate ≈99% → 強制 OTP / 人工審  
   - High Risk：Fraud Rate 100% → 自動鎖卡  

### 專案 insight

- 不只是單純秀指標，還考量了「最佳門檻穩定性」跟「訓練成本」的 trade-off  
- 把模型輸出實際應用在風險分層 + 決策建議  
- 使用真實感強的資料（時間、客戶、金額全都有），不是純 PCA 特徵  
- 在選擇模型時有考量到「上線後怎麼調、怎麼 retrain」