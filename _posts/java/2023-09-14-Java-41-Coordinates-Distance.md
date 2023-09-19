---
title:  "[Java] 두 좌표 사이의 거리 계산"
excerpt: "두 좌표(위경도) 사이의 거리 계산"

tags:
  - [Java, Coordinate, distance]

toc: true
toc_sticky: true
toc_label: "[Java]"
 
date: 2023-09-14
last_modified_at: 2023-09-14
---

### 각 좌표사이의 거리 구하기

```java
// 두 좌표 사이의 거리를 구하는 함수
// dsitance(첫번쨰 좌표의 위도, 첫번째 좌표의 경도, 두번째 좌표의 위도, 두번째 좌표의 경도)
public  Double distance(double lat1, double lon1, double lat2, double lon2){
	Double theta = lon1 - lon2;
	Double dist = Math.sin(deg2rad(lat1))* Math.sin(deg2rad(lat2)) + Math.cos(deg2rad(lat1))*Math.cos(deg2rad(lat2))*Math.cos(deg2rad(theta));
    dist = Math.acos(dist);
    dist = rad2deg(dist);
    dist = dist * 60*1.1515*1609.344;

    return dist; //단위 meter
}

// 10진수를 radian(라디안)으로 변환
private Double deg2rad(Double deg){
    return (deg * Math.PI/180.0);
}
//radian(라디안)을 10진수로 변환
private Double rad2deg(Double rad){
    return (rad * 180 / Math.PI);
}





public double calculateDistance(double lat1, double lon1, double lat2, double lon2) {
    double latDiff = lat2 - lat1;
    double lonDiff = lon2 - lon1;
    return Math.sqrt(latDiff * latDiff + lonDiff * lonDiff);
}
```
