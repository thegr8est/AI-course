---
layout: post
title: Capstone Project
author: [HsuanYu Yeh]
category: [Project]
tags: [jekyll, ai]
---

## 期末專題 -- 人體3D建模結合OpenPose的脊椎側彎檢測

### 實驗步驟圖
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/%E5%9C%96%E7%89%871.jpg?raw=true)<br>

---
### 一、建模
* **1.設定建模軟體參數**<br>
將體積大小或譯為定界框的參數設定為2m，以及深度參數設定為150cm，在定界框外以及距離攝影鏡頭150cm外的環境資訊將被建模軟體忽略，這算是一種初步的濾除背景。<br>
* **2.掃描人體**<br>
當建模軟體參數設定完成後，便可以開始拿著攝影機掃描人體，一開始先將攝影機對準人體正面，並確認定界框完整將人包住，確認無誤後，便可以拿著攝影機開始從正面由上往下掃，並繞著模特兒旋轉一圈，進行三百六十度的全身掃描，在掃描的過程，可以透過RecFusion建模軟體提供的即時建模畫面，觀看掃描的結果。雖然RecFusion提供定界框進行初步的濾除背景，但是在建模的過程中會產生些微背景雜訊。<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/%E5%9C%96%E7%89%872.jpg?raw=true)<br>
* **建模結果示意圖**<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/%E5%9C%96%E7%89%873.jpg?raw=true)<br>
* **3.濾除背景**<br>
利用Open3D模組，我們可以輕易地將建模軟體輸出的ply檔轉成pcd形式，轉成pcd後，將會使我們對模型的處理更加簡單，有利於我們後續的處理。
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/%E5%9C%96%E7%89%874.jpg?raw=true)<br>
* **4.將點雲分群**<br>
利用DBSCAN演算法，以密度為基礎對每個點雲進行分群，每個點雲都會被賦予一數值代表其屬於該群，此時離散在人體周圍的雜訊將被歸類為不同於人體的群。<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/%E5%9C%96%E7%89%875.jpg?raw=true)<br>
* **5.尋找目標群**<br>
從上圖中得出目標是綠色的群，且其是點雲數最多的一群，利用賦予給每個點雲的數值，我們透過scipy模組中的stats參數尋找眾數，便可以找出身為目標的綠色群。
* **6.移除雜訊**<br>
利用迴圈的方式，依序對每個點雲進行比對，如果該點雲被賦予的數值不等於眾數，則將其移除，以上便完成移除雜訊。
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/%E5%9C%96%E7%89%876.jpg?raw=true)<br>

---
### 二、正規化
方法是將地板法向量旋轉成與Z軸平行，便能將人體轉正，使人的身高方向與Z軸平行，正規化目的是方便我們之後對胸腰臀切片的處理，我們只需要分析XY平面即可。<br>
* **1.算出地板平面之方程式**<br>
利用RANSAC演算法，其將隨機抽取點雲作為內群來計算模型，模型完成後，將其他未被抽取到的點雲帶入模型中檢驗是否屬於內群，並記下內群數量，重複數次上述動作後，將含有最多內群數量的該模型作為我們的結果，最後將得到地板平面的方程式，因為該平面含有最多的點雲，以下圖是計算出地板平面的示意圖，以紅色來標記通過該平面的點雲。




<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*


