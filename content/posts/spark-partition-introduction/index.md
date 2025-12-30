---
sys:
  pageId: "2d81f336-f350-8000-bd6c-ecd18230ae20"
  createdTime: "2025-12-29T06:45:00.000Z"
  lastEditedTime: "2025-12-30T08:08:00.000Z"
  propFilepath: "posts/spark-partition-introduction/index.md"
title: "Partition — Spark 執行效能的關鍵"
date: "2025-12-30T00:00:00Z"
description: "Partition 是 Spark 運算的基礎單元，對於整體的執行效能有重要的影響。透過了解分區數量如何被創造，再針對實際的資料處理流程進行不同的優化改善。"
tags:
  - "🖇Data Engineering"
  - "🌟Spark"
categories:
  - "📱 技術"
author: "Harry Yang"
draft: false
url: "posts/spark-partition-introduction"
section: "posts"
---

# 分區(Partition)


在上一篇文章介紹了 Spark 如何有計畫性的執行程式碼，接下來要來探討如何在運算時操作 partition 數量來改善 Spark 的運算表現。


現在知道一個 job 開始運作後，會產生很多 tasks，每一個都會被指派給一個 executor，且一次只處理一個 partition 的資料。


因此當某個任務被分配到一個比較大的分區，就會佔用 executor 較長的時間。或者如果分區數量過多，整個系統就會被過多的開銷淹沒，如果分區的分配不平均，少數幾個執行緩慢的任務，會成為整個系統的瓶頸。


想像要搬運 100 個箱子到一輛卡車上，如果把它們分成 2 個貨棧，各有 50 個小箱子，只能讓 2 個工人搬動，速度很慢。如果把它們分成 100 個貨棧，雖然能讓 100 個工人處理，但是協調這些工作需要額外的時間。最佳平衡點應該介於兩者之間。這就是優化分區的意義。


{{< alert "lightbulb" >}}
Spark 分區與 Hive 分區完全不同， Hive 分區指的是檔案在磁碟上的分配方式，而 Spark 分割區指的是資料在計算期間在記憶體中如何分配。雖然是同一個詞，但意義卻截然不同。
{{< /alert >}}


## 初始分區(Initial Partition)


當資料第一次被載入 Spark 時，引擎會悄悄地分割成多個分區，分區的數量將取決於資料來源、檔案格式和配置設定的組合。


在處理 Parquet、CSV 或 single-line JSON p等可分割檔案時，分區數主要受這些參數控制：

- `spark.sql.files.maxPartitionBytes`：決定單個分區的資料上限(預設 128MB)，用於確保單個 Task 不會過大，導致資料容量超過 CPU 記憶體限制。
- `spark.sql.files.openCostInBytes`：定義開啟檔案的虛擬成本，限制一個分區塞入過多小型檔案，減少檔案讀取和寫入的壓力。
- `spark.default.parallelism`：只會影響 RDD 的 shuffle 運算，Spark SQL 或 DataFrame會忽略此設定，建議設為集群總核心數的 2 到 4 倍。
- `spark.sql.files.minPartitionNum`: 設定在讀取檔案類型的資料時，應該使用的最小分區數量。如果沒有額外定義則會讀取 `spark.default.parallelism` 設定的數量。

實際在運算時，這三個參數會被綜合考慮，依照以下算式決定分區的實際大小:

{{< katex >}}
$$
\begin{aligned}
\text{splitSize} = \min(&\text{maxPartitionBytes}, \newline
&\max(\text{openCostInBytes}, \text{bytesPerCore}))
\end{aligned}
$$


其中，`bytesPerCore` 的計算方式如下：

{{< katex >}}
$$
\text{bytesPerCore} = \frac{\sum \text{fileSizes} + (\text{fileCount} \times \text{openCostInBytes})}{\text{default.parallelism}}
$$


至於 streaming 資料來源，Spark 的分區邏輯會轉為對應 Kafka 的每個主題分區，建立一個分區，而 Cassandra 則會基於詞元範圍進行分割。此外，如果處理的檔案是不可分割的，如壓縮檔或 multi-line JSON，則無論檔案大小與 `maxPartitionBytes` ，只會產生一個分區。


## 洗牌 (Shuffle Partition)


當資料進入 Spark 引擎後，遇到 wide-transformation 就會發生 shuffle 的狀況，像是`groupBy()`, `join()`, `distinct()`, `orderBy()` 等。Spark 將 RDD 儲存資料，且資料分散在不同的執行區域，當使用如 `groupBy`，由於相同 key 的資料可能散布各處，無法在單獨區域上進行運算。所以在計算時會觸發 shuffle 機制，重新分配資料所處的區域後，才開始計算。


這些運算基本上，需要移動存在不同執行器中的數據，讓相同鍵的鍵位於同一節點上，同時可能會建立新的分區，是需要耗費大量資源的運算(網路傳輸、磁碟讀取、序列化和反序列化)。


而在配置的預設設定中，這些運算會使用 200 個分區來運算，不論運算的資料量大小。當檔案很小，每個分區可能處理幾十行資料，任務會變得極為簡單，浪費叢集時間，只是假裝在進行分散式運算。當然也有可能發生分區資料量過大，導致記憶體的運算問題。所以不論怎樣的資料量體，都要考量在 wide-transformation 實際要處理的資料量，來配置適當的分區數量。


## 產出 (Output Partition)


當運算完成，要將資料輸出成實體檔案，如果直接進行寫入檔案，Spark 會將運算結果的每個分區轉為一個檔案。如果目前有 200 個分區，就會有 200 個文件。即使其中一半檔案只有三行資料，Spark 照樣會存成一個檔案。最後沒有妥當處理分區，就會寫出成千上萬個散落在雲端儲存各處的小檔案。


# 優化分區配置


想要解決在不同階段可能遇到的效能問題，就要運用不同的手段來處理。因為 Spark 引擎不會了解資料特性，它不知道資料中某個屬性的分布嚴重傾斜，也不知道處理過程會捨棄多少資料，或者資料的稀疏姓，當按照某個屬性分區時，資料就會變得異常糟糕。


## Shuffle Partitions


透過設定 shuffle 行為的分區配置規則，可以改善 Spark SQL 或 DataFrame 執行 shuffle的效率。如果沒有依據資料處理的特性進行調整，引擎會依照預設的參數運算，導致效能降低。但如何選擇合適的分區數，這並沒有萬能公式，只能依照經驗和對數據的理解來調整。


通常會考慮上游的要 shuffle 的資料大小，並設定合理的 partition size 避免記憶體溢出，然後計算可能的分區數量。但初步計算後，也要同時考量工作的核心數目是否被妥善利用，有無閒置的工作節點。或是任務的數量非核心的整數倍，可能會導致最後一次計算，出現過多閒置的核心。


通常每個任務運算 100~200MB 左右的資料能達到不錯效果，需要再次強調，只有工程師了解數據，Spark 並不了解。


```javascript
spark.conf.set("spark.sql.shuffle.partitions", "600")
```


## Repartition


Repartition 的操作會迫使 spark 進行資料 shuffle，依照指定數量或特定的 key value 將資料分區重新分區。雖然會耗費龐大的運算資源，但有時也是必要的。例如資料經過過濾後存留的資料僅存在於部分分區，或是準備發送資料到模型訓練器或多節點寫入，需要平衡記憶體負載。


Repartition 有以下的特點

- 可以用來增加或減少分區。
- 會觸發 shuffle
- 能確保資料均衡分配，有助於更好的分散運算表現
- 最適用於：為實現分散運算而擴展分區、修復不平衡的分區。

```javascript
cleaned = df.filter(...).repartition(100)
```


## Coalesce


coalesce 是在不使用 shuffle 的情況下減少分區數量，合併現有分區，只能用於減少分區數量。但也因為這樣，不會重新平衡數據，而是直接將分區堆疊在一起，可能會導致分區的大小不平衡。


當在處理巨量的資料集時，如果將分區簡化為一個分區，會造成整個資料集由單一執行核心處理。對於大型資料集，應要維護足夠的分區，以保持分區的記憶分配合理。


```javascript
df.coalesce(100).write.parquet("s3://bucket/output/")
```


## PartitionBy


這個操作指的是磁碟上實體的資料分佈，決定資料如何保存到資料夾中，和實際的 Spark 分區是不一樣的。


使用情境通常是，希望輸出的檔案能依照資料欄位的值進行分類。例如要依照時間分類儲存，可以這樣寫，先使用 `repartition` 將相同日期的分配在同一區，再使用 `partitionBy` 歸類，這樣就能依照日期產生一個檔案並歸類在對應的檔案夾內。


```javascript
df.repartition("date").write.partitionBy("date").parquet(...)
```


# Adaptive Query Execution


自 Spark 3.0 起，Spark 引進了名為 [Adaptive Query Execution (AQE)](https://spark.apache.org/docs/latest/sql-performance-tuning.html#adaptive-query-execution) 的優化方法。AQE 比一般的 SQL 查詢規劃器更聰明，會根據執行效能統計(runtime statistics)，變更實際執行計劃(physical plan)。在執行階段，AQE 能幫助 Spark 引擎

- 自動調整分區: 解決 small files problem，AQE 會自動合併過小的分區，避免產生過多低數據的小型任務，減少排程負擔。
- 優化 join 策略: Spark 在編譯會決定資料 Join 方式，如 Sort-Merge Join 或 Broadcast Hash Join。如資料過濾後大幅縮減，AQE 會重新評估表的大小，如果表的大小低於閾值(Broadcast Threshold)，會將效能較差的 Sort-Merge Join 轉換為更快的 Broadcast Hash Join。
- 改善偏向的資料 join 效能(skew join): 當資料中特定 key 的數據量遠大於其他 key 時，在執行上會造成完成工作的 executor 閒置。AQE 會將傾斜的分區拆分成多個子分區，並將另一側要 join 的數據進行複製，縮短處理時間。

但 AQE 僅會發生在非串流的資料處理以及包含 shuffle 的執行運算，所以當資料第一次被載入還是要妥善規劃讀取後的分區配置，因為讀取資料不會涉及 shuffle。在處理串流資料而想要應用 AQE，可以使用 `foreachBatch` 將資料轉為 micro-batch 的技巧處理，但是要注意 AQE 會重新評估執行計畫，所以要求低延遲的系統可能就不適用。


{{<bookmark
name="databricks Optimization & performance — Adaptive query execution"
link="https://docs.databricks.com/aws/en/optimizations/aqe#query-plans"
description="Discusses how you can examine query plans in different ways.">}}


# Key Takeaways

- 初步載入資料，應該要深入了解資料和工作節點的數量配置，不要依賴 Spark 的自動分區，避免造成後續運算的效能瓶頸。
- Shuffle 運算產生的分區數在生產環境中，需要根據實際資料量調整，預設的數值不符合每個情境。
- 寫入輸出檔案前，務必做好規劃和評估，檔案數量等於分區數，善用 `coalesce` 或 `repartition` 進行控制，避免產生數千個小檔案。
- `repartition` vs. `coalesce`
	- `repartition` 在數據分佈不均或資料過濾時進行調整，雖然消耗龐大的運算成本但能確保資料分佈均勻，提升實際運算的效率。
	- `coalesce` 僅能用於減少分區數量，將分區合併但不會觸發 shuffle 行為。
- `partitionBy`： 決定磁碟儲存結構而非記憶體運算邏輯，切勿與 `repartition(num,'key')` 混淆。
- 自適應查詢執行 (AQE)： AQE 能幫忙改善運算效率，但仍需掌握資料特性與設定參數值，而非完全依賴自動修正。

[How to Optimize Your Apache Spark Application with Partitions](https://engineering.salesforce.com/how-to-optimize-your-apache-spark-application-with-partitions-257f2c1bb414/)


[luminousmen | Spark Partitions](https://luminousmen.com/post/spark-partitions/)


[Apache Spark | SQL Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html#tuning-partitions)


[Source Code of Partitioning from Apache Spark](https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/execution/datasources/FilePartition.scala)


[The Data Forge | Spark Partitions Explained](https://thedataforge.medium.com/spark-partitions-explained-why-partitioning-matters-for-performance-857ac2799c47)


[Vincent DANIEL | Optimizing Spark Performance with AQE: Mastering Shuffle Partition Coalescing](https://medium.com/@vincent_daniel/optimizing-spark-performance-with-aqe-mastering-shuffle-partition-coalescing-e9d3eef8579f)


[databricks article | Adaptive Query Execution: Speeding Up Spark SQL at Runtime](https://www.databricks.com/blog/2020/05/29/adaptive-query-execution-speeding-up-spark-sql-at-runtime.html)


[Daniel Aronovich | Mastering Apache Spark Partitioning: Coalesce vs. Repartition Optimizing Performance Through Strategic Data Distribution](https://bigdataperformance.substack.com/p/mastering-apache-spark-partitioning?utm_source=publication-search)


[Spark的RDD分區數如何決定](http://www.fanyilun.me/2021/10/23/Spark%E7%9A%84RDD%E5%88%86%E5%8C%BA%E6%95%B0%E7%94%B1%E4%BB%80%E4%B9%88%E5%86%B3%E5%AE%9A/)


[Guide to Partitions Calculation](https://dzone.com/articles/guide-to-partitions-calculation-for-processing-dat#:~:text=The%20majority%20of%20Spark%20applications,be%20tweaked%20by%20the%20user)
