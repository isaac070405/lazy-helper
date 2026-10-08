# Data visualisation 圖表指南

常見圖表 + 冷門實用圖 + 統計常用圖(共 46 個)

所有圖表均使用模擬數據,只作示範形狀同用法,唔代表真實結果。

## 目錄

**A. 常見圖表(6)**

- A1 · Bar chart ± SE + 原始點
- A2 · Boxplot(箱形圖)
- A3 · Histogram + density
- A4 · Scatter plot + 回歸線
- A5 · Line plot(mean ± SE)
- A6 · Heatmap(相關矩陣)

**B. 實用但冷門 — 第一批(12)**

- B1 · Raincloud plot(雨雲圖)
- B2 · Ridgeline plot(山脊圖)
- B3 · Bland–Altman plot
- B4 · Dumbbell plot(啞鈴圖)
- B5 · Ternary plot(三元圖)
- B6 · UpSet plot
- B7 · Alluvial / Sankey diagram
- B8 · Forest plot(森林圖)
- B9 · Funnel plot(漏斗圖)
- B10 · T–S diagram
- B11 · Hovmöller diagram
- B12 · nMDS ordination

**C. 額外推薦(8)**

- C1 · Slopegraph(斜率圖)
- C2 · Coefficient plot(係數圖)
- C3 · ECDF(累積分佈圖)
- C4 · Q–Q plot(殘差常態檢查)
- C5 · Kaplan–Meier 生存曲線
- C6 · Volcano plot(火山圖)
- C7 · Species accumulation curve
- C8 · Rose diagram(玫瑰圖)

**D. 海報遺漏嘅冷門圖(9)**

- D1 · Lollipop chart(棒棒糖圖)
- D2 · Violin plot(小提琴圖)
- D3 · Horizon graph
- D4 · Cycle plot
- D5 · Bullet graph
- D6 · Bump chart
- D7 · Deviation column(偏離柱狀圖)
- D8 · Food web(食物網)
- D9 · Metabolic pathway(代謝途徑圖)

**E. 統計常用但未提及(11)**

- E1 · ROC curve
- E2 · Residuals vs fitted(殘差診斷)
- E3 · Scatterplot matrix(散佈矩陣)
- E4 · PCA biplot
- E5 · Dendrogram(樹狀圖)
- E6 · Interaction plot(交互作用圖)
- E7 · Pareto chart
- E8 · Control chart(管制圖)
- E9 · Mosaic plot
- E10 · Permutation null distribution
- E11 · Hexbin / 2D density

配色:色盲友善(藍 / 橙 / 綠 / 紫);軸標籤、n 同 error 類型記得喺 figure caption 寫清楚。

## A1 · Bar chart ± SE + 原始點

用途:比較幾組平均值(例如三個處理)。加上原始點同 n,先符合 "indicate sample size and error"。

R:ggplot2(geom\_col + geom\_errorbar + geom\_jitter)

![A1 · Bar chart ± SE + 原始點](Data_visualisation_guide_v2_media/A1.png)

## A2 · Boxplot(箱形圖)

用途:睇中位數、四分位距同離群值,適合偏斜或有 outlier 嘅數據。

R:ggplot2(geom\_boxplot)

![A2 · Boxplot(箱形圖)](Data_visualisation_guide_v2_media/A2.png)

## A3 · Histogram + density

用途:睇單一變量嘅分佈形狀(偏斜、雙峰),同埋比較兩組分佈。

R:ggplot2(geom\_histogram / geom\_density)

![A3 · Histogram + density](Data_visualisation_guide_v2_media/A3.png)

## A4 · Scatter plot + 回歸線

用途:兩個連續變量嘅關係;灰帶係 95% CI。呈現 R² 或斜率。

R:ggplot2(geom\_smooth(method = "lm"))

![A4 · Scatter plot + 回歸線](Data_visualisation_guide_v2_media/A4.png)

## A5 · Line plot(mean ± SE)

用途:睇趨勢隨時間變化,適合重複量度;用唔同線型同符號,黑白打印都分到。

R:ggplot2(stat\_summary / geom\_errorbar)

![A5 · Line plot(mean ± SE)](Data_visualisation_guide_v2_media/A5.png)

## A6 · Heatmap(相關矩陣)

用途:一眼睇晒多個變量兩兩相關;亦可用喺 site × species 矩陣。

R:corrplot / ggplot2(geom\_tile)

![A6 · Heatmap(相關矩陣)](Data_visualisation_guide_v2_media/A6.png)

## B1 · Raincloud plot(雨雲圖)

用途:同時顯示密度、boxplot 同原始點,細樣本(n \< 30)特別好用,取代 bar ± SE。

R:ggdist、ggrain

![B1 · Raincloud plot(雨雲圖)](Data_visualisation_guide_v2_media/B1.png)

## B2 · Ridgeline plot(山脊圖)

用途:多組分佈疊埋比較,例如唔同月份或地點嘅體長分佈。

R:ggridges

![B2 · Ridgeline plot(山脊圖)](Data_visualisation_guide_v2_media/B2.png)

## B3 · Bland–Altman plot

用途:評估兩種量度方法係咪一致(例如 FFQ vs 食物記錄),虛線係 bias 同 95% limits of agreement。

R:blandr / ggplot2

![B3 · Bland–Altman plot](Data_visualisation_guide_v2_media/B3.png)

## B4 · Dumbbell plot(啞鈴圖)

用途:before–after 或兩個時間點嘅變化,比雙 bar 清楚。

R:ggalt(geom\_dumbbell)

![B4 · Dumbbell plot(啞鈴圖)](Data_visualisation_guide_v2_media/B4.png)

## B5 · Ternary plot(三元圖)

用途:三成分比例(例如碳水化合物 / 蛋白質 / 脂肪供能比,或沉積物組成)。

R:ggtern

![B5 · Ternary plot(三元圖)](Data_visualisation_guide_v2_media/B5.png)

## B6 · UpSet plot

用途:多集合交集;Venn diagram 超過 3 組就睇唔清,UpSet 冇問題。

R:UpSetR、ComplexUpset

![B6 · UpSet plot](Data_visualisation_guide_v2_media/B6.png)

## B7 · Alluvial / Sankey diagram

用途:類別之間嘅流動,例如飲食模式轉變、樣本分類轉換。

R:ggalluvial、networkD3

![B7 · Alluvial / Sankey diagram](Data_visualisation_guide_v2_media/B7.png)

## B8 · Forest plot(森林圖)

用途:meta-analysis 標準圖,顯示各研究 effect size 同 95% CI,菱形係合併結果。

R:metafor、forestplot

![B8 · Forest plot(森林圖)](Data_visualisation_guide_v2_media/B8.png)

## B9 · Funnel plot(漏斗圖)

用途:檢查 publication bias;一邊缺咗細研究(不對稱)就要懷疑有 bias。

R:metafor(funnel)

![B9 · Funnel plot(漏斗圖)](Data_visualisation_guide_v2_media/B9.png)

## B10 · T–S diagram

用途:溫度對鹽度,辨識水團;虛線係等密度線(示意)。

R:oce、ggplot2

![B10 · T–S diagram](Data_visualisation_guide_v2_media/B10.png)

## B11 · Hovmöller diagram

用途:時間對空間(例如緯度)嘅 heatmap,睇 SST 或 chlorophyll 嘅長期模式。

R:ggplot2(geom\_raster)、oce

![B11 · Hovmöller diagram](Data_visualisation_guide_v2_media/B11.png)

## B12 · nMDS ordination

用途:群落結構相似度,常配 PERMANOVA;要標 stress 值,\< 0.2 可接受。

R:vegan(metaMDS、adonis2)

![B12 · nMDS ordination](Data_visualisation_guide_v2_media/B12.png)

## C1 · Slopegraph(斜率圖)

用途:同一個體兩個時間點嘅變化;粗線係平均,顏色分辨升跌。

R:ggplot2(geom\_line + geom\_point)

![C1 · Slopegraph(斜率圖)](Data_visualisation_guide_v2_media/C1.png)

## C2 · Coefficient plot(係數圖)

用途:一眼比較回歸模型各預測因子嘅效應大小同 CI;CI 冇跨過 0 = 顯著。

R:broom + ggplot2、sjPlot

![C2 · Coefficient plot(係數圖)](Data_visualisation_guide_v2_media/C2.png)

## C3 · ECDF(累積分佈圖)

用途:比較各組整體分佈,唔使揀 bin 數;啱用喺 CFU 呢類偏斜數據。

R:ggplot2(stat\_ecdf)

![C3 · ECDF(累積分佈圖)](Data_visualisation_guide_v2_media/C3.png)

## C4 · Q–Q plot(殘差常態檢查)

用途:ANOVA 或回歸嘅殘差診斷;點貼住線表示接近常態。

R:qqnorm / car::qqPlot

![C4 · Q–Q plot(殘差常態檢查)](Data_visualisation_guide_v2_media/C4.png)

## C5 · Kaplan–Meier 生存曲線

用途:比較兩組生存時間(例如藻類存活、隊列研究);+ 記號係 censored 個案。

R:survival + survminer

![C5 · Kaplan–Meier 生存曲線](Data_visualisation_guide_v2_media/C5.png)

## C6 · Volcano plot(火山圖)

用途:同時睇效應大小同顯著性,找出「大變化 + 顯著」嘅 taxa 或基因。

R:EnhancedVolcano、ggplot2

![C6 · Volcano plot(火山圖)](Data_visualisation_guide_v2_media/C6.png)

## C7 · Species accumulation curve

用途:睇取樣夠唔夠(曲線是否趨平),比較兩種生境嘅 taxa 數。

R:vegan(specaccum)

![C7 · Species accumulation curve](Data_visualisation_guide_v2_media/C7.png)

## C8 · Rose diagram(玫瑰圖)

用途:方向性數據,例如海流、風向或潮流頻率。

R:ggplot2(coord\_polar)、openair(windRose)

![C8 · Rose diagram(玫瑰圖)](Data_visualisation_guide_v2_media/C8.png)

## D1 · Lollipop chart(棒棒糖圖)

用途:排名或比較大量類別(例如各物種豐度),比 bar chart 版面清爽。

R:ggplot2(geom\_segment + geom\_point)

![D1 · Lollipop chart(棒棒糖圖)](Data_visualisation_guide_v2_media/D1.png)

## D2 · Violin plot(小提琴圖)

用途:同時顯示分佈形狀同中位數 / 四分位距,睇到雙峰(Site B)。

R:ggplot2(geom\_violin)、vioplot

![D2 · Violin plot(小提琴圖)](Data_visualisation_guide_v2_media/D2.png)

## D3 · Horizon graph

用途:將多條長時間序列壓縮成細細一行;紅 = 高於平均,藍 = 低於平均,越深越偏離。適合大量站點嘅 SST 比較。

R:ggHoriPlot

![D3 · Horizon graph](Data_visualisation_guide_v2_media/D3.png)

## D4 · Cycle plot

用途:同時睇季節規律(每個月一段)同長期趨勢(段內斜率);橙線係該月平均。

R:ggplot2(手動 x 位置或 facet)

![D4 · Cycle plot](Data_visualisation_guide_v2_media/D4.png)

## D5 · Bullet graph

用途:實際值對目標,例如營養攝取對 RDI;灰帶係表現範圍,黑線係目標。

R:ggplot2(geom\_col + geom\_errorbar)

![D5 · Bullet graph](Data_visualisation_guide_v2_media/D5.png)

## D6 · Bump chart

用途:排名隨時間變化,例如物種豐度排名。

R:ggbump

![D6 · Bump chart](Data_visualisation_guide_v2_media/D6.png)

## D7 · Deviation column(偏離柱狀圖)

用途:偏離基線嘅異常(例如 SST anomaly),紅 / 藍一眼睇到升跌。

R:ggplot2(geom\_col + fill = value > 0)

![D7 · Deviation column(偏離柱狀圖)](Data_visualisation_guide_v2_media/D7.png)

## D8 · Food web(食物網)

用途:物種之間嘅捕食關係;箭嘴由被食者指向捕食者,縱位代表營養級。

R:igraph、ggraph、ggnetwork

![D8 · Food web(食物網)](Data_visualisation_guide_v2_media/D8.png)

## D9 · Metabolic pathway(代謝途徑圖)

用途:代謝物(節點)同酵素反應(箭嘴)嘅連接;橙色係限速 / 調控步驟。此圖為糖酵解簡化示意。

R / 工具:pathview、KEGG、Cytoscape

![D9 · Metabolic pathway(代謝途徑圖)](Data_visualisation_guide_v2_media/D9.png)

## E1 · ROC curve

用途:評估診斷或分類模型表現;越貼近左上角越好,AUC 越接近 1。

R:pROC

![E1 · ROC curve](Data_visualisation_guide_v2_media/E1.png)

## E2 · Residuals vs fitted(殘差診斷)

用途:回歸診斷;隨機散佈(左)= OK,漏斗或彎曲(右)= 變異不均或非線性。

R:plot(lm)、ggfortify::autoplot

![E2 · Residuals vs fitted(殘差診斷)](Data_visualisation_guide_v2_media/E2.png)

## E3 · Scatterplot matrix(散佈矩陣)

用途:探索多變量兩兩關係;對角線係各變量分佈,右上係 r 值。

R:GGally::ggpairs、pairs()

![E3 · Scatterplot matrix(散佈矩陣)](Data_visualisation_guide_v2_media/E3.png)

## E4 · PCA biplot

用途:降維;點係樣本,箭嘴係變量負荷,睇到組別分離同變量貢獻。

R:prcomp + factoextra::fviz\_pca\_biplot

![E4 · PCA biplot](Data_visualisation_guide_v2_media/E4.png)

## E5 · Dendrogram(樹狀圖)

用途:階層式聚類,睇樣本或物種點樣分組;高度 = 距離。

R:hclust、ggdendro、dendextend

![E5 · Dendrogram(樹狀圖)](Data_visualisation_guide_v2_media/E5.png)

## E6 · Interaction plot(交互作用圖)

用途:兩因子 ANOVA;線唔平行表示有 interaction。

R:ggplot2(stat\_summary)、emmeans::emmip

![E6 · Interaction plot(交互作用圖)](Data_visualisation_guide_v2_media/E6.png)

## E7 · Pareto chart

用途:找出「少數主因」(80/20);柱係頻數,線係累積百分比(呢類圖天生用雙 y 軸)。

R:qcc(pareto.chart)、ggQC

![E7 · Pareto chart](Data_visualisation_guide_v2_media/E7.png)

## E8 · Control chart(管制圖)

用途:監察量度過程是否穩定;超出 ±3σ 或連續漂移即係失控,適合儀器校正。

R:qcc、ggQC

![E8 · Control chart(管制圖)](Data_visualisation_guide_v2_media/E8.png)

## E9 · Mosaic plot

用途:兩個分類變量嘅關係,面積代表比例(例如處理 × 健康狀態);配合卡方檢定。

R:ggmosaic、vcd::mosaic

![E9 · Mosaic plot](Data_visualisation_guide_v2_media/E9.png)

## E10 · Permutation null distribution

用途:排列檢定;灰色係隨機分配下嘅差異分佈,橙色尾部係比觀察值更極端嘅部份,即 p-value。

R:infer、coin

![E10 · Permutation null distribution](Data_visualisation_guide_v2_media/E10.png)

## E11 · Hexbin / 2D density

用途:數據點太多(overplotting)時,用顏色顯示密度。

R:ggplot2(geom\_hex / geom\_density\_2d)

![E11 · Hexbin / 2D density](Data_visualisation_guide_v2_media/E11.png)
