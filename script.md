
# Wildfire CO₂｜PPT 備忘錄完整講稿

> 由原始 PowerPoint 的「備忘稿 / Presenter Notes」逐頁擷取。**講稿文字保留原文，沒有改寫**；來源與補充說明也一併保留。
>
> 14 頁｜適合 GitHub 手機版快速跳頁。

## 快速跳頁

- [第 01 頁｜Paper Survey](#slide-01)
- [第 02 頁｜Outline](#slide-02)
- [第 03 頁｜Why Estimate Wildfire CO₂ Emissions?](#slide-03)
- [第 04 頁｜Three Phases: Height → Biomass → CO₂](#slide-04)
- [第 05 頁｜Data: SAR, Optical, GEDI and FIA](#slide-05)
- [第 06 頁｜Phase 1: From Samples to Height Maps](#slide-06)
- [第 07 頁｜Phase 1: How LightGBM Learns](#slide-07)
- [第 08 頁｜Phase 2: Estimate Forest Biomass](#slide-08)
- [第 09 頁｜Phase 3: Calculate CO₂ Emissions](#slide-09)
- [第 10 頁｜Uncertainty: Error Propagation](#slide-10)
- [第 11 頁｜Uncertainty: Height and Biomass](#slide-11)
- [第 12 頁｜Uncertainty: AGB Loss and CO₂](#slide-12)
- [第 13 頁｜Final Estimate and Main Limitations](#slide-13)
- [第 14 頁｜Future Work](#slide-14)

---

<a id="slide-01"></a>
## 第 01 頁｜Paper Survey: Estimating Wildfire CO₂ from GEDI, SAR and Optical Data

### 講稿

今天介紹的研究，是利用衛星資料估算森林火災的二氧化碳排放。我會把重點放在資料怎麼配對、模型怎麼計算，以及最後的估算有哪些限制。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 1–2.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-02"></a>
## 第 02 頁｜Outline

### 講稿

先介紹資料，再說明三個計算階段，最後看不確定性和結果。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 2.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-03"></a>
## 第 03 頁｜Why Estimate Wildfire CO₂ Emissions?

### 講稿

這篇想解決的問題是：從影像看到森林燒毀後，要怎麼估計它排放了多少二氧化碳？

研究案例是美國奧勒岡州二〇二〇年的 Beachie Creek 大火。作者利用災前和災後的森林變化，建立一套排放量的估算方法。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 2–3.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-04"></a>
## 第 04 頁｜Three Phases: Height → Biomass → CO₂

### 講稿

作者把計算分成三個階段。

第一階段，用衛星和地形資料預測樹高，產生災前、災後兩張樹高地圖。

第二階段，利用地面調查建立樹高和生物量的關係，把樹高換成每公頃有多少地上植物乾重。

第三階段，將災前生物量減掉災後生物量，用這個差值估計火災消耗量，再換算成二氧化碳排放。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 2–3、5–6.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-05"></a>
## 第 05 頁｜Data: SAR, Optical, GEDI and FIA

### 講稿

這幾種資料各有不同的用途。

Sentinel-1 提供 VV、VH 雷達數值；Sentinel-2 提供光譜，以及 NDVI 等植生指標。加上地形和土地覆蓋，這些都是預測樹高的輸入。

GEDI 是裝在國際太空站上的光達，利用雷射回波觀測森林的垂直結構。它提供的 RH 是相對高度；RH95 就是從地面往上累積回波能量，達到百分之九十五時所對應的高度。這篇用它當作樹冠高度的參考。

FIA 是美國的森林調查計畫，利用地面量到的樹種、胸徑和樹高估算生物量。這篇用 FIA 資料建立和驗證第二階段的公式。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 3–4.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-06"></a>
## 第 06 頁｜Phase 1: From Samples to Height Maps

### 講稿

這一階段的目標，是估計災前和災後，整片森林各個位置的樹高。

但 GEDI 只在部分位置有量測，所以作者要利用這些有參考高度的位置，訓練模型去預測其他地方。

首先，把同一個位置的資料配在一起。輸入包括 Sentinel-1 的 VV、VH 雷達數值、Sentinel-2 的光譜，以及 NDVI 等植生指標，再加上高程、坡度、坡向和土地覆蓋類型。模型要預測的答案，則是這個位置的 GEDI RH95，也就是這篇採用的參考高度。

例如，某個位置的 RH95 是三十公尺，作者就把這個位置的衛星和地形數值，配上三十公尺，整理成一筆樣本。其他有 GEDI 量測的位置也照樣處理，模型就有很多筆資料可以學習。

接著，作者把災前和災後的資料分開，各自隨機取八成來訓練，留下兩成測試。

這裡使用的演算法叫 LightGBM。它會根據輸入數值，用決策樹把樣本分組，產生高度預測。接著比較預測和 GEDI 參考高度的差距，再加入新的決策樹，學習怎麼修正目前的誤差。也就是透過一棵接一棵的樹，逐步調整預測結果。

為了確認哪些資料有幫助，作者比較三種組合。第一組使用光學、植生指標和土地覆蓋；第二組再加入雷達；第三組再加入地形資料。這樣就能檢查，多加入一種資料，是否真的能讓預測更準。模型設定則用 Optuna 搜尋，例如要用多少棵樹，以及每次修正的幅度。

訓練完成後，再拿留下的兩成資料做測試，把預測高度和 GEDI 參考高度比較。

作者用三個指標評估。R 平方看模型能解釋多少高度差異，越接近一越好；RMSE 用公尺表示整體誤差大小，越小越好；rRMSE 則把 RMSE 除以平均參考高度，換成百分比，方便比較。

最後，把整個研究區各位置的輸入資料交給訓練好的模型，就能預測沒有 GEDI 量測的位置。災前和災後各用自己的模型，產生兩張完整的樹高地圖。

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-07"></a>
## 第 07 頁｜Phase 1: How LightGBM Learns

### 講稿

這一頁主要介紹 LightGBM 如何利用光學與 SAR 特徵預測森林高度，以及如何透過多棵樹，逐步修正預測誤差。

【指左圖】

首先看左邊，這裡用一棵簡化的回歸樹，說明預測的過程。

模型的輸入包含光學與 SAR 特徵，例如光學的植生指數 NDVI，以及 SAR 的 VH 極化後向散射資訊。

模型會根據這些特徵，一層一層進行判斷。例如，先判斷 NDVI 是否大於 0.6，如果是，就繼續判斷 VH 是否超過某個門檻；不同的判斷結果，會走到不同的葉節點，輸出對應的森林高度。

圖中的 8、18 和 24 公尺都是示意值，用來說明一棵回歸樹如何把輸入特徵轉換成高度預測。實際的切分門檻與輸出數值，都是透過訓練資料學習得到的。

【指中間】

了解單棵樹的運作方式後，再看中間：LightGBM 如何讓多棵樹一起改善預測。

它會先產生一個初始預測，接著依序建立新的樹，學習目前模型留下的誤差，並提供修正量。

例如，圖中從 20 公尺開始，加入一棵樹提供的正 3 公尺修正，再加入下一棵樹提供的負 1 公尺修正，最後得到 22 公尺。

也就是說，後面的樹會根據目前還有哪些地方預測不準，進一步調整結果。

【指右邊】

那麼，每一輪的修正量是怎麼來的？右邊用一個簡化的平方誤差範例來說明。

假設有兩筆訓練樣本，參考高度分別是 30 公尺和 28 公尺，但模型目前都預測成 20 公尺，因此分別低估了 10 公尺和 8 公尺。這個「參考值減去預測值」的差距，就稱為殘差。

如果新的一棵樹把這兩筆樣本分到同一個葉節點，在這個簡化例子中，就可以取兩筆殘差的平均，得到 9 公尺，作為這個葉節點的修正量。

接著，再透過學習率控制實際修正的幅度。這裡的學習率是 0.1，所以這一輪實際加上的數值，就是 9 乘以 0.1，等於 0.9 公尺。

因此，兩筆樣本的預測都會從 20 公尺更新成 20.9 公尺。更新後仍然存在的誤差，就交給下一棵樹繼續學習。

所以，整個流程就是：先預測、計算誤差、加入修正，再根據剩餘誤差繼續學習。LightGBM 就是透過這樣逐輪累加修正的方式，改善森林高度的預測結果。

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-08"></a>
## 第 08 頁｜Phase 2: Estimate Forest Biomass

### 講稿

這張圖是在建立預測樹高和生物量之間的關係。

每個黑點代表一個 FIA 樣區。橫軸是第一階段預測的樹高，縱軸則是 FIA 地面調查資料估算的生物量，也就是每公頃有多少公噸的地上植物乾重。

作者把這些資料配在一起，調整公式裡的 a 和 b，讓算出的生物量盡量接近這些黑點，最後得到圖中的虛線。這個過程就叫擬合。

例如，預測樹高是二十公尺，代入這篇建立的公式，就會得到每公頃大約一百五十二公噸的生物量。

因此，只要把整張樹高地圖逐格代入公式，就能得到生物量地圖。作者用災前資料建立這條關係，再用災後 FIA 資料驗證，最後將同一條公式套用到災前和災後。

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-09"></a>
## 第 09 頁｜Phase 3: Calculate CO₂ Emissions

### 講稿

這一頁就是把生物量減少的重量，換成二氧化碳的重量。

例如，災前每公頃兩百公噸，災後剩一百公噸，作者就把差的一百公噸當作消耗量。

先乘零點五，表示生物量約有一半是碳，得到五十公噸碳。再乘零點九，表示其中以二氧化碳形式排放的碳所占比例，得到四十五公噸碳。

最後乘四十四除以十二，把碳的重量換成二氧化碳的重量，得到每公頃一百六十五公噸二氧化碳。

這三個係數合起來是一點六五。每個網格算完後，再乘上以公頃表示的網格面積，最後加總成整場火災的排放量。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 6，式 7.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-10"></a>
## 第 10 頁｜Uncertainty: Error Propagation

### 講稿

不確定性分析要回答的是：前面的估算誤差，會讓最後的排放量有多不確定？

作者先利用災前災後都有 GEDI 觀測的位置，檢查模型估出的變化是否一致；再用誤差傳播公式，把樹高、生物量和相減加總時的誤差逐步合併。圖中的每一格，就是在追蹤這些誤差如何往後影響。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 6–8，2.3.6.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-11"></a>
## 第 11 頁｜Uncertainty: Height and Biomass

### 講稿

這頁先算單一網格的樹高和生物量，有多少不確定性。

公式八，用預測高度乘 rRMSE。例如二十公尺乘百分之二十，就是四公尺。公式九再合併作者採用的 GEDI 誤差零點五公尺；假設兩者獨立，平方相加再開根號，約為四點零三公尺。

接著，生物量除了受樹高影響，也受 a、b 的估計影響。公式十用偏微分表示各項改變會影響多少生物量，再合併各項不確定性。公式十四就是把偏微分代入後的結果。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 7，式 8–14.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-12"></a>
## 第 12 頁｜Uncertainty: AGB Loss and CO₂

### 講稿

這頁處理兩件事：災前災後相減，以及整個區域加總。

先看公式十五。如果災前災後都多估十，相減時這個誤差可能抵消；如果一邊高估、一邊低估，差值的誤差就可能更大。所以要考慮兩個時期的誤差關聯。

再看公式十六。如果附近網格常常一起高估，加總時就不能假設誤差會互相抵消。作者利用 semivariogram 估計不同距離的關聯，再用公式十八計算 covariance，納入區域的不確定性。

最後，公式十九乘上相同的排放換算係數。作者在這一步忽略換算係數本身的變動，將它們視為固定值。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 7–8，式 15–19.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-13"></a>
## 第 13 頁｜Final Estimate and Main Limitations

### 講稿

最後，作者估計這場火災消耗八點八一 Tg 生物量，換算為十四點五四 Tg 二氧化碳。Tg 就是百萬公噸；排放量的不確定性為正負四點七八 Tg，大約是估算值的三分之一。

解讀時有兩個重點。第一，樹高預測仍有誤差，尤其容易低估較高的樹。第二，作者把生物量減少當作火災消耗量；但若死亡木材仍留在現場，就不一定代表當下全部燒掉。

因此，這個數字是依照模型和換算假設得到的排放估計。以上是我的報告，謝謝大家。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Source: Cui et al. (2025), Ecological Indicators 178, 113889. DOI: 10.1016/j.ecolind.2025.113889. PDF pp. 6、8、14–17. The retained-dead-wood caveat is an interpretation of the consumption assumption in section 2.3.5, not a measured result. Reported uncertainty is not labeled a 95% confidence interval.

</details>

[↑ 回到目錄](#快速跳頁)

---

<a id="slide-14"></a>
## 第 14 頁｜Future Work

### 講稿

未來想做兩件事。第一，參考 BioMassters 的多模態模型，用 SAR 和光學影像直接估計災前、災後的生物量。第二，比較這種方法和目前三階段方法的生物量損失及二氧化碳估算結果，檢查在火災區域的準確度。以上是我的報告，謝謝大家。

<details>
<summary>原始備忘錄的來源／補充資訊（報告時可略過）</summary>

Proposed future work, not an established result. BioMassters is a benchmark with multiple baseline models; the architecture remains to be selected and validated on pre/post-fire data. Source: https://nascetti-a.github.io/BioMasster/ ; Cui et al. (2025), DOI:10.1016/j.ecolind.2025.113889.

</details>

[↑ 回到目錄](#快速跳頁)

---
