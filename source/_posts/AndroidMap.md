---
title: 开源地图方案:天地图+osmdroid
date: 2023-08-22 18:19:20
tags: 
categories: Android
---

> 随着地图服务提供商(腾讯、百度、高德)开始收费，商业授权都是5万一年，贫穷使我们需要找一个免费的替代方案。
本文介绍了一个基于 天地图+osmdroid 的开源地图方案，源码见 github 项目 [AndroidMap](https://github.com/yadiq/AndroidMap)。

## 1.天地图

### 1.1 官网配置
天地图API是一个地图服务提供商，之前还是提供Android的地图SDK的，现在就只提供了API服务了。
官网地址：http://lbs.tianditu.gov.cn/
需要注册账户，创建应用，获取应用Key

### 1.2 API服务
1. 地图API，可以获取地图瓦片，这里我们使用以下两种图层。
网址：http://lbs.tianditu.gov.cn/server/MapService.html。
+ 影像底图-球面墨卡托投影
http://t0.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=您的密钥

+ 影像注记-球面墨卡托投影
http://t0.tianditu.gov.cn/cia_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=cia&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=您的密钥

2. WEB服务API，可以获取地理信息数据，例如：地名搜索、逆地理编码查询
网址：http://lbs.tianditu.gov.cn/server/geocoding.html

## 2.osmdroid
### 2.1 官方说明
osmdroid 是免费的Android的MapView类的替代品，可以加载多个图层，可以绘制点线面。
网址：https://github.com/osmdroid/osmdroid
使用说明在wiki: https://github.com/osmdroid/osmdroid/wiki

### 2.2 加载地图资源
```java
//影像地图 _W是墨卡托投影  _c是国家2000的坐标系
private OnlineTileSourceBase tileSourceImg = new OnlineTileSourceBase("Tian Di Tu Img", Constants.zoomMinLevel, Constants.zoomMaxLevel, Constants.tileSizePixels, "",
	new String[]{Constants.tileSourceImgUrl}) {
		@Override
		public String getTileURLString(final long pMapTileIndex) {
			return getBaseUrl() + "&TILECOL=" + MapTileIndex.getX(pMapTileIndex) + "&TILEROW=" + MapTileIndex.getY(pMapTileIndex) + "&TILEMATRIX=" + MapTileIndex.getZoom(pMapTileIndex);
		}
};
MapView mapView;
mapView.setTileSource(tileSourceImg);
```

### 2.3 绘制线

```java
Polyline line = new Polyline(mapView);
Paint paint = line.getOutlinePaint();
paint.setStrokeWidth(CommonUtil.dp2px(mapView.getContext(), 2));
paint.setColor(Color.YELLOW);
List<GeoPoint> points = new ArrayList<>();
line.setPoints(points);
line.setGeodesic(false);
line.setInfoWindow(null);
mapView.getOverlayManager().add(line);
```

### 2.4 添加标记

```java
Marker myLocation2;
myLocation2 = new Marker(mapView);
startMarker.setPosition(startPoint);
myLocation2.setAnchor(Marker.ANCHOR_CENTER, Marker.ANCHOR_CENTER);
Drawable drawable = CommonUtil.getContext().getResources().getDrawable(R.drawable.location_vehicle);
myLocation2.setIcon(drawable);
myLocation2.setInfoWindow(null);
mapView.getOverlays().add(myLocation2);
```

## 3.源码
源码位置：
https://github.com/yadiq/AndroidMap/blob/main/app/src/main/java/com/hqumath/map/ui/main/MapWidgetPresenter.java

功能说明：
+ 添加两个图层，影像底图和影像注记。
+ 添加缩放按钮，支持缩放手势。
+ 支持点击事件。
+ 绘制航线、航点、标记。

## 4.效果图

![AndroidMap](https://github.com/yadiq/AndroidMap/raw/main/img/AndroidMap1.jpg)