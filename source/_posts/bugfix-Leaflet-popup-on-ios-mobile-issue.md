---
title: [bug fix] 使用Leaflet.GridLayer.GoogleMutant，popup window 在ios mobile不能點
date: 2018-09-12 18:54:40
tags: 
- bug 
- Leaflet 
- GridLayer 
- GoogleMutant
- popup
- ios
- mobile
---

我們公司專案的地圖部分最近都換成用 [Leaflet](https://leafletjs.com/) 這個套件
並且是用vue包裝起來後的 `vue2-leaflet` 和 `vue2-leaflet-markercluster`
圖層部分使用 [Leaflet.GridLayer.GoogleMutant](https://gitlab.com/IvanSanchez/Leaflet.GridLayer.GoogleMutant) 這個套件

## 問題

但最近遇到一個很奇怪的問題
如題，就是地圖上popup window不能點
而且電腦版Chrome跟safari正常，Android手機也可以
就只有ios手機不行

## 初步判斷

一開始發現是z-index問題

![](https://i.imgur.com/G5kYzgt.png)

這個套件所在的 `leaflet-google-mutant` z-index 為800
而popup所在的 `leaflet-pane` 這層為400

如果把 `leaflet-pane` 這層的 z-index 設成800以上就可以點了
不過想不透如果是 z-index問題，為何只有ios mobile不能點

於是找了他們的issue看有沒有人提出類似的問題

## 尋找案例

果然找到很類似的issue：[GoogleMutant not allowing buttons/links on popup on iOS](https://gitlab.com/IvanSanchez/Leaflet.GridLayer.GoogleMutant/issues/42)

花了很多時間在咀嚼他們的討論串（英文不好...）
裡面也有人提到z-index的問題
有個像是contributer的人[回覆](https://gitlab.com/IvanSanchez/Leaflet.GridLayer.GoogleMutant/issues/42#note_36882665)，似乎是說z-index會調整到800是因為要讓google的logo浮出來

![](https://i.imgur.com/ZFvniwW.png)

後面也有人提出是ios不支援 `pointer-events: none` 的 [bug](https://bugs.webkit.org/show_bug.cgi?id=154807) ，他表示改成 `pointer-events: auto` 就可以了（但我試了發現連popup都出不來了!）
而developer針對此解法有開出一條branch請大家幫忙測試，不過好像沒後續

最後有人提出一個[解法](https://gitlab.com/IvanSanchez/Leaflet.GridLayer.GoogleMutant/issues/42#note_85257464) ，試過發現成功了！

```css=
.gm-style iframe {
  width: 0px !important;
  height: 0px !important;
}
```

原本iframe的css width和height都是100%
所以看起來是google map 本身的iframe 遮住了
不過還是覺得很奇怪，為何只有ios mobile有問題... 

但目前還找不太到原因，日後有發現再來補充吧！
如果有人也遇到這個問題的，希望可以幫到你們：）