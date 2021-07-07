---
title: "مانیتور کردن لاگ های IIS توسط ElasticSearch و Kibana"
date: 2021-07-04T15:52:00-00:00
categories:
  - IIS
tags:
  - elasticSearch
  - kibana
  - log
  - iis
  - filebeat
---

با بزرگ شدن پروژه و گسترش اون، به تبع اطلاعات بدست اومده بصورت نمایی زیاد میشه و دیگه با ابزارهای قبلی نمیشه این حجم از اطلاعات رو پردازش کرد.  
بطور مثال اگه بخوایم لاگ های جمع شده از یه برنامه رو بررسی کنیم، ابزارهای قدیمی جوابگو نیستن.  
یکی از ابزارهای خیلی مفید و خوب برای این کار Elastic Search هستش.  
این ابزار پنل مدیریت گرافیکی بصورت پیش فرض نداره، پس ابزاری به اسم kibana تولید شده که امکانات گرافیکی رو در اختیار شما میزاره.  

در این مطلب میخوایم لاگ های تولید شده توسط IIS رو با این ابزارها مدیریت کنیم.  
برای این کار ابتدا از نصب بودن Java روی سیستم خودتون مطمئن بشید.  
توسط دستور زیر میشه از ورژن جاوا نصب شده مطمئن شد:  

```shell
java --version
```

برای اینکه مطمئن بشید قابلیت لاگ برای iis فعال هست هم میتونید از قسمت زیر اقدام کنید:  

<p align="center" >
  <img src="/assets/img/iis_log.png" alt="mhkarami97" width="600" />
</p>

[اطلاعات بیشتر درباره لاگ](https://docs.microsoft.com/en-us/iis/manage/provisioning-and-managing-iis/managing-iis-log-file-storage)  

حالا نوبت به نصب elasticsearch میرسه.  
برای این کار کافیه فایل msi رو از لینک زیر دانلود کنید و طبق داکیومنت نصب کنید:  

[elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/windows.html) 

اگه همه چیز درست پیش رفته باشه باید آدرس زیر رو ببینید

```shell
http://localhost:9200
```

برای نصب kibana میتویند از لینک زیر فایل zip برای windows رو دانلود کنید.  

[kibana](https://www.elastic.co/downloads/kibana) 

بعد از دانلود اون بطور مثال در درایو C از حالت فشرده خارج کنید و اسم پوشه رو به kibana تغییر بدید.  
حالا cmd رو در پوشه `C:\kibana\bin` باز کنید

با اجرا دستور زیر برنامه شروع به کار میکنه:  

```shell
kibana.bat
```

اگر کانفیگ پیش فرض elasticsearch رو تغییر داده بودید نیاز به تغییر فایل `config/kibana.yml` در kibana هستش.  

اگه همه چیز درست پیش رفته باشه آدرس زیر رو باید مشاهده کنید.

```shell
http://localhost:5601
```

حالا نوبت به نصب ابزار filebeat میرسه.  
برای این کار فایل windows zip رو از آدرس زیر دانلود کنید و اون رو در درایو c از حالت فشرده خارج کنید و اسمش رو به filebeat تغییر بدید.  

[filebeat](https://www.elastic.co/downloads/beats/filebeat) 

حالا powershell رو بصورت ادمین اجرا کنید و به آدرس `C:\filebeat` برید و دستور زیر رو اجرا کنید:  

```shell
.\install-service-filebeat.ps1
```

اگه کانفیگ های پیش فرض elasticsearch رو تغییر داده باشید نیاز به تغییر فایل `filebeat.yml` هستش.  

حالا دستور زیر رو داخل cmd و همون مسیر `C:\filebeat` وارد کنید.  

```shell
filebeat.exe modules enable iis
```

و بعدش دستور زیر رو وارد کنید:  

```shell
.\filebeat.exe setup
```

و بعد دستور زیر رو وارد کنید تا سرویس استارت بشه:  

```shell
Start filebeat
```

اگه همه چی درست پیش رفته باشه باید با کلیک روی کلید Check Data در لینک زیر متن سبز رنگ رو مشاهده کنید.  

[iisLogs](http://localhost:5601/app/home#/tutorial/iisLogs) 

اگه متن پیام سبز رنگ رو مشاهده کردید، با کلیک روی iis log dashboard لینک بالا میتونید پنل مانیتور لاگ ها رو مشاهده کنید.  

همچنین توسط این ابزار میتونید مانیتور ها مدیریت دیتا خیلی از ابزارهای دیگه رو انجام بدید که در لینک زیر راهنما اونها در دسترس هست:  

[tools](http://localhost:5601/app/home#/tutorial_directory) 

<p align="center" >
  <img src="/assets/img/kibana.png" alt="mhkarami97" width="600" />
</p>

یکی از ابزارهای خوب دیگه برای مدیریت لاگ، ابزاری با اسم logstash هستش.  
توسط این ابزار میتونید لاگ های خودتون رو مدیریت کنید و کوئری های دلخواه رو روی اون اجرا کنید.  

برای نصب این ابزار، از لینک زیر فایل zip رو دانلود کنید و در درایو c از حالت فشرده خارج کنید و اسم اون رو به logstash تغییر بدید.  

[logstash](https://www.elastic.co/downloads/logstash) 

برای اجرا این ابزار ابتدا باید فایل کانفیگ بسازید و مشخصات برنامه ای که میخواید لاگ های اون پردازش بشه رو وارد کنید.  
برای این کار داخل پوشه config فایلی بطور مثال با اسم `iis.conf` بسازید.  
برای مدیریت لاگ های iis میتونید کانفیگ زیر رو قرار بدید.  

```
input {
	file {
		type => "IISLog"
		path => "C:/inetpub/logs/LogFiles/W3SVC*/*.log"
		start_position => "beginning"
	}
}

filter {

	# ignore log comments
	if [message] =~ "^#" {
		drop {}
	}
 
 	# check that fields match your IIS log settings
	grok {
        match => ["message", "%{TIMESTAMP_ISO8601:log_timestamp} %{IPORHOST:site} %{WORD:method} %{URIPATH:page} %{NOTSPACE:querystring} %{NUMBER:port} %{NOTSPACE:username} %{IPORHOST:clienthost} %{NOTSPACE:useragent} (%{URI:referer})? %{NUMBER:response} %{NUMBER:subresponse} %{NUMBER:scstatus} %{NUMBER:time_taken}"]
	}
  
	# set the event timestamp from the log
	# https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html
	date {
		match => [ "log_timestamp", "YYYY-MM-dd HH:mm:ss" ]
		timezone => "Etc/UCT"
	}
	
	# matches the big, long nasty useragent string to the actual browser name, version, etc
	# https://www.elastic.co/guide/en/logstash/current/plugins-filters-useragent.html
	useragent {
		source=> "useragent"
		prefix=> "browser_"
	}
	
	mutate {
		remove_field => [ "log_timestamp"]
	}
}

# output logs to console and to elasticsearch
output {
    stdout { codec => rubydebug }
	elasticsearch { hosts => ["localhost:9200"] }
}
```

برای مطالعه بیشتر درباره کانفیگ میتونید لینک زیر رو مطالعه کنید.  

[configuration](https://www.elastic.co/guide/en/logstash/current/configuration.html) 

حالا به پوشه bin برید و دستور زیر رو وارد کنید تا ابزار شروع به کار کنه:  

```
logstash.bat -f C:\logstash\config\iis.conf
```

بعد از انجام کار بالا نیاز به تعریف index در آدرس زیر هستش که نام کانفیگ اضافه شده در قسمت pattern ها نشون داده میشه

```
http://localhost:5601/app/management/kibana/indexPatterns
```

با انتخاب اون و انجام مراحل میتونید متغیرهایی که نیاز به کوئری زدن و بررسی دارید رو مشخص کنید.  
البته ابزار filebeat برای مدیریت لاگ iis راه اندازی راحت تری داره.
