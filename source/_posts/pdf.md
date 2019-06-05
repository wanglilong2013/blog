---
title: PHP HTML转PDF
date: 2019-06-05 14:45:50
tags: [php, pdf]
---

最近在开发中,有一个页面的行程部分需要下载成pdf文件，由于有composer包的加持，很快这个功能就实现了。但是中途还是出现了一些小插曲，在这里简单记录一下。

--------------


开发语言： php 

工具包： knplabs/knp-snappy


## 代码实现

```
    /**
     * Download itinerary pdf （行程下载）
     *
     * @return document
     */
    public function actionItinerary()
    {
        $snappy = new  \Knp\Snappy\Pdf('/usr/local/bin/wkhtmltopdf');
        $productId = Yii::app()->request->getParam('product_id', null);

        if (empty($productId)) {
            return $this->responseError(null, '行程id不能为空');
        }
        $productInfo = $this->factoryService->getProductInfo(['product_id' => $productId]);
        if ($productInfo == false) {
            return $this->responseError(null, '获取产品信息失败');
        }
        $itineraryData = isset($productInfo['itinerary']) ? $productInfo['itinerary'] : [];
        if (empty($itineraryData)) {
            return $this->responseError(null, '获取产品行程失败');
        }
        $params = [
            'itinerary_data' => $itineraryData,
            'return' => true,
        ];

        $itineraryHtml = $this->widget('application.widgets.CustomizeItineraryWidget', $params)->content;
        header('Content-Type: application/pdf');
        header('Content-Disposition: attachment; filename="itinerary.pdf"');
        echo $snappy->getOutputFromHtml($itineraryHtml);
    }

```

## 遇到的问题

下载下来的pdf全是黑色方块字体，英文能正常显示。起初以为是header头的问题，但是加上header过后依然乱码，最后确认问题在于系统缺少中文字体文件。


## 解決方案

上传或下载中文字体(wqy-zenhei.ttc)到服务器/usr/share/fonts 目录,可根据实际情况选择合适的中文字体





