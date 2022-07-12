---
title:  "[Java] Date - 날짜 더하기 빼기"
excerpt: "날짜 더하기 뺴기와 현재시간 구하기"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[Date]"
 
date: 2022-07-12
last_modified_at: 2022-07-12
---


### ``샘플코드``

```java
public static void main(String[] args) {
	String currentTime = getCurrentDateString("yyyy-MM-dd'T'HH:mm:ssXXX");				
	try {
		String expireTime = AddDate(currentTime, 0, 0, 0, 2);
		Date currentDate = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ssXXX").parse(currentTime);
		Date expireDate = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ssXXX").parse(expireTime);
		
		long differenceSecond = (expireDate.getTime() - currentDate.getTime()) / 1000;
		System.out.println("currentDate : "+currentDate);
		System.out.println("expireDate : "+expireDate);
		System.out.println("differenceSecond : "+differenceSecond);
	} catch (Exception e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}

}

// 현재 시간을 주어진 포멧의 문자열로 반환 
public static String getCurrentDateString(String format){
	SimpleDateFormat formatSdf = new SimpleDateFormat(format);
	TimeZone time = TimeZone.getTimeZone("Asia/Seoul");
	formatSdf.setTimeZone(time);

	return formatSdf.format(new Date());
}

// 날짜 더하기 빼기
private static String AddDate(String strDate, int year, int month, int day, int minute) throws Exception {
	SimpleDateFormat dtFormat = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ssXXX");
	Calendar cal = Calendar.getInstance();
	Date dt = dtFormat.parse(strDate);
	cal.setTime(dt);
	cal.add(Calendar.YEAR, year);
	cal.add(Calendar.MONTH, month);
	cal.add(Calendar.DATE, day);
	cal.add(Calendar.MINUTE, minute);
	return dtFormat.format(cal.getTime());
}
```

