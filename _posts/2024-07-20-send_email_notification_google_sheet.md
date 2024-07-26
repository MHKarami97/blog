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
function onOpen(e) {
   installTrigger();
};

// Function to install the onEdit trigger
function installTrigger() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet();
  ScriptApp.newTrigger('sendEmailOnEdit')
           .forSpreadsheet(sheet)
           .onEdit()
           .create();
}

// Function to send email on edit
function sendEmailOnEdit(e) {
  // Get the active spreadsheet and sheet
  const sheet = e.source.getActiveSheet();
  const range = e.range;
  const row = range.getRow();
  const column = range.getColumn();
  const oldValue = e.oldValue;
  const newValue = e.value;
  const changeType = getChangeType(e);
  const headerRange = sheet.getRange(1,1,1,sheet.getLastColumn()).getValues()[0];
  const header = headerRange[column-1];

  if(header !== 'توضیح'){
    return;
  }

  // Get all user emails with access to the spreadsheet
  const userEmails = getAllUserEmails();

  // Create the email subject and body
  const subject = 'Life';

    const body = `
    <div style="direction: rtl; text-align: right;font-family: Tahoma;">
      <p>نوع تغییر: ${changeType}</p>
      <p>نام شیت: ${sheet.getName()}</p>
      <p>ردیف: ${row}</p>
      <p>مقدار: ${newValue}</p>
      <p>ویرایش توسط: ${Session.getActiveUser().getEmail()}</p>
    </div>
  `;             

  //Send the email to all users
  userEmails.forEach(email => {
    MailApp.sendEmail({
      to: email,
      subject: subject,
      htmlBody: body
    });
 });
}

// Function to determine the type of change
function getChangeType(e) {
  const range = e.range;
  const sheet = e.source.getActiveSheet();
  const lastRow = sheet.getLastRow();
  const lastColumn = sheet.getLastColumn();

  if (range.getRow() > lastRow - 1 && range.getLastRow() == lastRow) {
    return 'جدید';
  } else if (e.oldValue === undefined) {
    return 'حذف';
  } else {
    return 'ویرایش';
  }
}

// Function to get all user emails with access to the spreadsheet
function getAllUserEmails() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet();
  const editors = sheet.getEditors();
  //const owner = sheet.getOwner();
  
  const emails = new Set();

  editors.forEach(editor => emails.add(editor.getEmail()));
  //emails.add(owner.getEmail());

  return Array.from(emails);
}
```

این کد ایمیل تمام افرادی که به فایل دسترسی دارند را بدست می‌آوردن و در صورت تغییر به آنها ایمیل ارسال می‌کند.  
اکنون بر روی Save کلیک کنید. سپس از همان بالا که لیست متودها را نشان می‌دهد یکبار OpOpen را انتخاب کنید و سپس ر روی Run کلیک کنید.  
با اینکار پنجره‌ای برای گرفتن دسترسی باز می‌شود که باید آن را تایید کنید. همچنین با توجه به اینکه حساب شما تایید شده توسط گوگل نیست نیز ممکن است هشدار داده شود که آن را نیز می‌توانید با unsafe تایید کنید.  