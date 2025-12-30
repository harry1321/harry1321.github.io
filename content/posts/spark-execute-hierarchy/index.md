---
sys:
  pageId: "2d61f336-f350-809e-b566-ec0dd80d3f2f"
  createdTime: "2025-12-27T01:03:00.000Z"
  lastEditedTime: "2025-12-30T08:08:00.000Z"
  propFilepath: "posts/spark-execute-hierarchy/index.md"
title: "Spark 的運算的階層結構: 從應用程式到分區"
date: "2025-12-29T00:00:00Z"
description: "了解 Spark 的運作流程處理，實際知道資料處理是如何從高階應用程式逐漸流向工作節點，有助於了解如何優化運算效率。"
tags:
  - "🖇Data Engineering"
  - "🌟Spark"
categories:
  - "📱 技術"
author: "Harry Yang"
draft: false
url: "posts/spark-execute-hierarchy"
section: "posts"
---

了解 Spark 的運作流程處理對於優化平行運算的表現至關重要，解析這些工作流程，能實際知道資料處理是如何從高階應用程式逐級流向工作節點。


Spark 的執行模型是有清晰的層次結構依序是應用 → 作業 → 階段 → 任務，將大型分散式運算分解成越來越小的單元，從而達到平行運算處理。


在 Spark 的層級結構中，Driver 位於這些層級的頂層，管理整個流程，協調工作如何從流向執行器，換句話說 Spark 運算是依照 Driver 創建的計劃來執行。


本質上 Driver 是一個 JVM process，在 client mode 運作下，位於個人電腦或提交伺服器上，遠端協調所有運算工作。在叢集模式下，它會部署到叢集內部，運行在專用節點上。無論哪種情況，驅動程式的存在都至關重要——如果終止驅動程序，應用程式也將隨之消失。沒有驅動程序，就沒有 Spark。


# 應用(Application)：Spark 運算的入口


在最頂層是 Spark 應用程式，就是正在執行的 Python 腳本、Scala 程式或筆記本。在生產環境中，也可能是呼叫 PySpark 腳本的 Airflow 排程任務。一個應用程式可以產生多個 Spark 作業，取決於程式碼中呼叫的操作數量。


應用程式中的所有操作會由 Driver 發出，它是將高階邏輯轉換為可執行現實的唯一入口點，默默地控制著分散式系統的混亂運作。


# 工作(Jobs)：Action 觸發執行計劃


在 Spark 中實際的 action 行為會建立一個單獨的工作，例如 `count()`, `collect()`, `write.parquet()` 等等。


有了 action 行為，Spark Driver 才會將邏輯 DAG 轉換為實體執行計劃，並開始在 executor 上執行計算。


Spark jobs 是 Spark 開始規劃如何將運算任務分解為叢集中平行運算任務的層次。


# 階段(Stages)：執行計畫中的邊界


有了執行計畫，Spark Driver 會進一步細分為多個 Stages。各個階段的切分邊界將由 shuffle 運算決定，也可以稱作為 wide transformation，如`groupByKey()`、`reduceByKey()`、`join()`、`repartition()` 等，也就是那些需要將資料重新分佈到不同 executor 的運算行為。


Wide transformation 會發生資料 shuffle，資料將在在 executor 之間移動，需要耗費很高的運算成本。並且在執行流程中創建邊界，分成不同的 stage 運算。


# 任務(Tasks)：Spark 運算的基礎單元


在結構的最底層是 tasks，每個 stage 被分解成多個 task，每個資料分區對應一個任務，也就是 partition 數量等同 task 數量。


在相同階段內的所有任務，都執行相同的操作，只是操作不同分區的資料。


這些任務就是最終會被指派 executor 的 CPU 核心進行運算，一個任務交由一個可用插槽(slots)處理，這些插槽數量通常對應於 CPU 的核心數。


當任務數量超過可以使用的核心數量，將會需要等候，這也是優化平行運算的要點之一。


當分配資料的 partition 數量時，會希望以 CPU 核心數量的倍數進行分配，來避免有閒置資源，但同時需要注意過多的 partition會造成 overhead 效能問題。


# 參考來源


[Big Data Performance Understanding-apache-sparks-execution](https://bigdataperformance.substack.com/p/understanding-apache-sparks-execution)
