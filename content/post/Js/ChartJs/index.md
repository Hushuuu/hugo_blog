---
title: "圖表套件使用-Chart js"
description: "利用Chart js產出長條圖折線圖等"
date: 2021-05-03T13:59:47+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Javascript
tags: [
    "Chart"
]
---

## 前言

在使用套件時，根據環境跟使用，引入的`script`版本非常重要  
版本不同有些使用的方式也會改變或是增減造成效果不一。  
`CDN`
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/moment.js/2.22.2/moment.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/2.7.3/Chart.min.js"></script>
```

## 主要內容

資料固定且簡單的範例只需引入後直接使用產生  

```html
<canvas id="myChart"></canvas>
<script>
 var ctx = document.getElementById("myChart");
    var chart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels:{"A","B","C"},
            datasets: [{
                label: ["銷售量"],
                data: { "A": "100", "B": "200", "C": "140" }
            }],
       }
    })
</script>
```

### 非固定資料

可用`Ajax`撈回或使用`PartialView`的概念

```html
<script>
$(document).on('click', '#show_board', function () {
         $.ajax({
            type: "POST",
            url: '@Url.Action("ShowBoard", "Product")'
                })
             .done(function (msg) {
                 //後端傳回為json物件包含圖表所需的資料跟設定
                 var obj = msg;
                 chart.data.datasets = obj.chartDatasets;
                 chart.data.labels = obj.xaxisLabels;
                 //重繪
                 chart.update();
        });
    });
</script>
```
`設計圖表所需的Model`
```C#
    /// <summary>
    /// 線資料只含值的model
    /// </summary>
    /// <typeparam name="T"></typeparam>
    public class ChartLineModel<T>
    //where T : struct
    {
        public ChartLineModel()
        {
            this.fill = false;
        }
        /// <summary>
        /// 資料TAG
        /// </summary>
        public string label { get; set; }
        /// <summary>
        /// 是否填滿
        /// </summary>
        public bool fill { get; set; }
        /// <summary>
        /// 折線顏色
        /// </summary>
        public string borderColor { get; set; }
        /// <summary>
        /// 點內顏色
        /// </summary>
        public string pointBackgroundColor { get; set; }
        /// <summary>
        /// 資料值清單
        /// </summary>
        public List<T> data { get; set; }
    }
    /// <summary>
    /// 線資料包含X與Y的值的model
    /// </summary>
    /// <typeparam name="X"></typeparam>
    /// <typeparam name="Y"></typeparam>
    /// <summary>
    /// XY軸的值
    /// </summary>
    public class ChartPointData<X, Y>
    //where X : struct
    //where Y : struct
    {
        /// <summary>
        /// x value
        /// </summary>
        public X x { get; set; }
        /// <summary>
        /// y value
        /// </summary>
        public Y y { get; set; }
    }

    /// <summary>
    /// 圖表設定
    /// </summary>
    public class ChartSetting<X, Y>
    //where X :struct
    {
        public ChartSetting()
        {
            this.type = "line";
        }
        public string type { get; set; }
        public List<string> XaxisLabels { get; set; }
        public List<ChartDatasets<X, Y>> chartDatasets { get; set; }
    }
    /// <summary>
    /// 圖表設定=>datasets
    /// </summary>
    public class ChartDatasets<X, Y>
    //where X : struct
    {
        public ChartDatasets()
        {
            this.backgroundColor = RandomColor();
            this.borderColor = backgroundColor;
            this.showLine = true;
            this.fill = false;
            this.pointBackgroundColor = backgroundColor;
            this.cubicInterpolationMode = "monotone";
        }
        /// <summary>
        /// 資料
        /// </summary>
        public List<ChartPointData<X, Y>> data { get; set; }
        private string RandomColor()
        {
            string color = string.Empty;
            StringBuilder sb = new StringBuilder();
            int temp;
            for (int i = 0; i < 6; i++)
            {
                Random r = new Random(Guid.NewGuid().GetHashCode());
                temp = r.Next(0, 16);
                sb.Append(Convert.ToString(temp, 16));
            }
            color = $"#{sb.ToString()}";
            return color;
        }
        /// <summary>
        /// 資料TAG
        /// </summary>
        public string label { get; set; }
        /// <summary>
        /// 是否填滿
        /// </summary>
        public bool fill { get; set; }
        ///// <summary>
        ///// 填滿顏色
        ///// </summary>
        public string backgroundColor { get; set; }
        /// <summary>
        /// 是否顯示
        /// </summary>
        public bool showLine { get; set; }
        /// <summary>
        /// 折線顏色
        /// </summary>
        public string borderColor { get; set; }
        /// <summary>
        /// 點內顏色
        /// </summary>
        public string pointBackgroundColor { get; set; }
        /// <summary>
        /// 線的弧度
        /// </summary>
        public string cubicInterpolationMode { get; set; }
    }
```

`Action 以日期區間的銷售圖`
```C#
public async Task<IActionResult> ShowBoard()
{
    ChartSetting<DateTime, decimal?> chartSetting = new ChartSetting<DateTime, decimal?>();
    //圖表樣式
    chartSetting.type = "line";
    //x座標日期區間
    List<string> xAxisLabels = new List<string>();
    DateTime backhunday = DateTime.Now.AddDays(-100);
    var maxdate = _context.ORDER_M.Where(x => x.DOC_DATE > backhunday).Max(x => x.DOC_DATE).ToString("yyyy-MM-dd");
    var mindate = _context.ORDER_M.Where(x => x.DOC_DATE > backhunday).Min(x => x.DOC_DATE).ToString("yyyy-MM-dd");
    xAxisLabels.Add(mindate);
    xAxisLabels.Add(maxdate);
    //XY資料
    List<ChartDatasets<DateTime, decimal?>> chartDatasets = new List<ChartDatasets<DateTime, decimal?>>();
    //從資料庫撈出訂單日期銷售情況的LIST
    List<LeaderBoardBase> list = await _ProductDB.LeaderBoard(EnumKind.LeaderBoardKind.ByDate, pKind, qDate);

    //將LIST做GROUP BY商品FOREACH=>商品數=圖表線(datasets)的數量
    list.GroupBy(z => z.PRO_ID).Select(x => x.Key).ToList()
        .ForEach(x =>
    {
        //線
        ChartDatasets<DateTime, decimal?> chartds = new ChartDatasets<DateTime, decimal?>();
        //線裡的xy值list
        List<ChartPointData<DateTime, decimal?>> cpd_list = new List<ChartPointData<DateTime, decimal?>>();
        list
        .Where(z=>z.PRO_ID==x)
        .GroupBy(z=> new { PRO_ID=z.PRO_ID,SALE_DATE=z.SALE_DATE})
        .Select(z=> new { SALE_DATE=z.Key.SALE_DATE,SALE_MONEY=z.Sum(c=>c.SALE_MONEY)})
        .OrderBy(z=>z.SALE_DATE)
        .ToList()
        .ForEach(z =>
        {
            ChartPointData<DateTime, decimal?> cpd = new ChartPointData<DateTime, decimal?>
            {
                x = z.SALE_DATE.Value.Date,
                y = z.SALE_MONEY
            };
            cpd_list.Add(cpd);
        });
        chartds.data = cpd_list;
        //線label
        chartds.label = _context.PRODUCT.Where(c => c.PRO_ID.ToString() == x).Select(c => c.PRO_NAME).FirstOrDefault();
        //將線加進圖表datasets裡
        chartDatasets.Add(chartds);
    });
    //圖表設定x軸標籤
    chartSetting.XaxisLabels = xAxisLabels;
    //圖表設定資料
    chartSetting.chartDatasets = chartDatasets;
    return Json(chartSetting);
}
```

設定圖表時間格式
```html
<script>
var chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels : {},
            datasets : {}
        },
        options: {
            responsive: true,
            title: {
                display: true,
            },
            animation: {
                duration: 2000
            },
            scales: {
                xAxes: [{
                    scaleLabel: {
                        display: true,
                        labelString: '日期',
                    },
                    type: 'time',
                    time: {
                        unit: 'day',
                        tooltipFormat: 'YYYY-MM-DD',
                        displayFormats: {
                            'day': 'YY-MM-DD'
                        }
                    }
                }],
                yAxes: [{
                    scaleLabel: {
                        display: true,
                        labelString: '金額',
                    },
                    ticks: {
                        beginAtZero: true,
                    }
                }]
            },
        }
    });
</script>
```

## 小結

`ChartJs`在官網的介紹跟範例中有更多樣式和設定的方法  
要注意設定參數、資料的順序、格式是否傳入正確  
否則可能畫出來會不符預期或是圖表顯示不出來  
需要花點時間研究設定的格式

## 參考連結

>* [url1](https://www.chartjs.org/)