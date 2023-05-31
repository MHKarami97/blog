---
title: "الگوی Template Method"
categories:
  - DesignPattern
tags:
  - design_pattern
  - template_method
---

این الگو در زمان هایی کاربرد دارد که شما یک کار را میخواهید انجام بدهید که بیشتر قسمت های آن یکسان است اما بعضی از قسمت های آن دارای تفاوت است.  

بطور مثال شما میخواهید پیام ها را از یک منبع که میتواند صف یا فایل باشد بخوانید و آنها را در دیتابیس ذخیره و سپس به محل دیگری ارسال کنید.  
در این حالت قسمت ذخیره و ارسال دوباره یکسان است و فقط قسمت خواندن متفاوت است.  
در این حالت میتوان از این پترن استفاده کرد.  

روش انجام به این صورت میباشد که یک کلاس `abstract` تعریف کرده و متودهای یکسان را در آن تعریف میکنیم و متدهای متفاوت را بصورت `abstract` تعریف میکنیم.  


```c#
public abstract class Processor
{
    protected abstract void Process(string messageId, string message);

    public abstract bool Start(string msgId);

    protected void SendToQueue(string messageId, string message, string msgType)
    {
        _sender.SendToQueue(label, body, msgType);
    }

    protected void SaveToDb(string messageId, string msgType, string message, DateTime eventDateTime)
    {
        DbPersistor.Instance.Add(new RawMessage
        {
            MessageId = messageId,
            MessageType = msgType,
            MessageText = message,
            EventDateTime = eventDateTime,
            CreationDateTime = DateTime.Now
        });
    }
}
```

سپس با توجه به نوع دریافتی که میخواهیم کلاس های مختلفی ایجاد میکنیم که از کلاس بالا ارث بری کرده اند.  


```c#
public abstract class StreamProcessor : Processor
{
    protected override void Process()
    {
        try
        {
           var messageResult = _connection.InitAsyncConnection(msgId);

            SaveToDb(messageResult.messageId, messageResult.msgType, messageResult.message,
                messageResult.eventDateTime);

            SendToQueue(messageResult.messageId, messageResult.message, messageResult.msgType);
        }
        catch (Exception ex)
        {
            Log4Repo.Critical("Exception in process message string", ex);
        }
    }
}
```

```c#
public abstract class FileProcessor : Processor
{
    protected override void Process()
    {
        try
        {
            using (var fileStream = new FileStream(filePath, FileMode.Open, FileAccess.Read))
            {
                using (var streamReader = new StreamReader(fileStream, Encoding.Default))
                {
                    string messageResult;

                    while (!_shouldStop && (messageResult = streamReader.ReadLine()) != null)
                    {
                        try
                        {
                            SaveToDb(messageResult.messageId, messageResult.msgType,
                                messageResult.message, messageResult.eventDateTime);

                            SendToQueue(messageResult.messageId, messageResult.message,
                                messageResult.msgType);
                        }
                        catch (Exception e)
                        {
                            Log4Repo.Critical(e);
                        }
                    }
                }
            }    
        }
        catch (Exception ex)
        {
            Log4Repo.Critical("Exception in process message string", ex);
        }
    }
}
```

در دو قسمت بالا متود proccess که دارای پیاده سازی متفاوتی میباشد، بازنویسی شده است و بقیه متودها که یکسان میباشند به همان صورت استفاده شده اند.  

برای انتخاب نوع کلاسی که میخواهید نیز میتوانید از `Factory Pattern` استفاده کنید.  

در لینک زیر هم میتونید درباره این الگو بیشتر مطالعه کنید:  

[refactoring](https://refactoring.guru/design-patterns/template-method)  

