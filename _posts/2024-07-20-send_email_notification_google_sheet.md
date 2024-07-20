---
title: "ارسال ایمیل در صورت تغییر در Google Sheet"
categories:
  - Trick
tags:
  - google_sheet
  - notification
  - email
  - script
---

در صورتی که در Google Sheet نیاز داشتید در صورت Add/Update/Delete متوجه تغییرات بشوید کافی است از اسکریپت زیر استفاده کنید.  
بدین منظور از سربرگ Extensions گزینه Apps Script را انتخاب کنید. سپس توسط + یک فایل جدید ایجاد کنید.  
در فایل ایجاد شده کد زیر را قرار دهید.   

```csharp
function installTrigger() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet();
  ScriptApp.newTrigger('sendEmailOnEdit')
           .forSpreadsheet(sheet)
           .onEdit()
           .create();
}

function sendEmailOnEdit(e) {
  const sheet = e.source.getActiveSheet();
  const range = e.range;
  const row = range.getRow();
  const column = range.getColumn();
  const oldValue = e.oldValue;
  const newValue = e.value;
  const changeType = getChangeType(e);
  const userEmails = getAllUserEmails();

  const subject = 'Google Sheet Change Notification';
  const body = `A change was made to your Google Sheet:\n\n` +
               `Change Type: ${changeType}\n` +
               `Sheet Name: ${sheet.getName()}\n` +
               `Cell: ${range.getA1Notation()}\n` +
               `Old Value: ${oldValue}\n` +
               `New Value: ${newValue}\n` +
               `Edited By: ${Session.getActiveUser().getEmail()}\n`;

  userEmails.forEach(email => {
    MailApp.sendEmail(email, subject, body);
  });
}

function getChangeType(e) {
  const range = e.range;
  const sheet = e.source.getActiveSheet();
  const lastRow = sheet.getLastRow();
  const lastColumn = sheet.getLastColumn();

  if (range.getRow() > lastRow - 1 && range.getLastRow() == lastRow) {
    return 'Addition';
  } else if (e.oldValue === undefined) {
    return 'Deletion';
  } else {
    return 'Update';
  }
}

function getAllUserEmails() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet();
  const editors = sheet.getEditors();
  const owners = sheet.getOwners();
  
  const emails = new Set();

  editors.forEach(editor => emails.add(editor.getEmail()));
  owners.forEach(owner => emails.add(owner.getEmail()));

  return Array.from(emails);
}

function onOpen() {
  installTrigger();
}
```

این کد ایمیل تمام افرادی که به فایل دسترسی دارند را بدست می‌آوردن و در صورت تغییر به آنها ایمیل ارسال می‌کند.  
اکنون بر روی Save کلیک کنید. سپس از همان بالا که لیست متودها را نشان می‌دهد یکبار OpOpen را انتخاب کنید و سپس ر روی Run کلیک کنید.  
با اینکار پنجره‌ای برای گرفتن دسترسی باز می‌شود که باید آن را تایید کنید. همچنین با توجه به اینکه حساب شما تایید شده توسط گوگل نیست نیز ممکن است هشدار داده شود که آن را نیز می‌توانید با unsafe تایید کنید.  