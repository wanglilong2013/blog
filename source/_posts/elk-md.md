---
title: 用elasticsearch + Logstash(过滤) + Filbeat(采集) + kibana收集项目日志
date: 2019-07-17 13:51:52
tags: [ELK]
---

# 用elasticsearch + Logstash(过滤) + Filbeat(采集) + kibana收集项目日志


## elasticsearch

基于lucense，采用Java开发的一款全文搜索引擎软件

## logstash
ES家族中的日志搜集软件，有丰富的input-pluging, filter-pluging, output-plugin, 日志收集部分采用Java开发，插件部分采用ruby开发，对系统消耗较高。

## filebeat
由于Logstash比较耗费系统资源，后面推出了beats,主要用于日志采集，filebeat就是beats中的一员，采用Go语言开发。

## kibana
ES家族中前端呈现, 对ES里面的数据做更好的呈现,可以绘制多种丰富的图表。以及对ES数据的一些相关操作。




## 安装
请参考官方文档

elasticsearch: 

kibana:

logstash:

filebeat:

## 配置



参考文档：
