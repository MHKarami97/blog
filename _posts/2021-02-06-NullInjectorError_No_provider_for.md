---
title: "خطا null injector در زمان استفاده از service در انگولار"
date: 2021-02-06T13:00:00-00:00
categories:
  - Angular
tags:
  - angular
  - dependecy
  - nullinjector
---

در یکی از پروژ هایی که در حال انجامشون بودم، در قسمتی نیاز به استفاده از یک سرویس بصورت زیر بود:

```ts
@Component({
    selector: 'app-com',
})
export class AppComponent {

    constructor(private _doctorToken: DoctorTokenLocalService) {
    }

    selectItem (item) {
        let result = item.row.data.id;

        if (result !== null) {
            this._doctorToken.setToken(result);
            this._doctorToken.ChangeIsFirstTime(true);

            this.ref.close();
        }
    }
}
```

```ts
@Injectable()
export class DoctorTokenLocalService {

    public setToken(token: string): void {
        localStorage.setItem('token', token);
    }

    public getToken(): string {
        let result = localStorage.getItem('token');

        if (result !== null) {
            return result;
        } else {
            this.showTokenSelect();
        }
    }
}
```

روش کار راحت بود و در پروژه های قبل برای این کار به ارور خاصی برخورد نکرده بودم. اما این بار تقریبا 1.5 روز وقت خواست تا بدونم مشکل چی هست
<br />
اروری که بهش برخوردم، به این صورت بود:

```shell
NullInjectorError: No provider for DoctorTokenLocalService
```

با کمی جستجو داخل اینترنت به لینک های زیر رسیدم که البته روش های گفته شده رو بلد بودم ولی برای اطمینان کارهای گفته شده رو دوباره انجام دادم:
<br />
[َAngular](https://angular.io/guide/dependency-injection)  
[Stackoverflow](https://stackoverflow.com/questions/47380239/nullinjectorerror-no-provider-for-angularfirestore)  
<br />

اما باز هم مشکل حل نشد، گفتم شاید جایی اشتباه کردم پس داخل یه برنچ جدید شروع به نوشتن دوباره کدها کردم اما باز هم مشکل حل نشد.
<br />
در قسمت UI من از کامپوننت devextreme استفاده کرده بودم و طبق داکیومنت خودش برای افزودن ایونت کلیک به قسمتی میشد بصورت زیر عمل کرد:


```html
<dxi-column type="buttons"
            caption="انتخاب"
            [width]="50">
            <dxi-button name="انتخاب"
            icon="rename"
            [onClick]="selectItem">
</dxi-button>
```

این ارور هم زمانی بوجود میومد که این متد فراخونی میشد
<br />
نکته ای که وجود داشت این بود که سرویسی که inject کرده بودم داخل constructor به درستی کار میکرد ولی داخل متود نه
<br />
با جستجوهایی که کردم تا حدودی به مشکل پی بردم، متد مورد نظر رو من بصورت پارامتر استفاده کرده بودم.
<br />
به این صورت:

```html
<dxi-column>
            <dxi-button
            [onClick]="selectItem">
</dxi-button>
```

در حالت عادی متدها به این صورت فراخونی میشن:

```html
<dxi-column>
            <dxi-button
            (onClick)="selectItem()">
</dxi-button>
```

راه حلی که به ذهنم رسید بصورت زیر بود:
```html
<dxi-column>
            <dxi-button
            (onClick)="selectItem($event)">
</dxi-button>
```

مشکلی که بود این بود که داخل کامپوننت استفاده شده برای اکشن onClick فقط پیاده سازی اول بود.
<br />
با کمی جستجو به کد زیر رسیدم. روش زیر داخل فریم ورک های دیگه زیاد استفاده میشه ولی من تو انگولار خیلی شبیهش رو ندیده بودم:

```ts
@Component({
    selector: 'app-com',
})
export class AppComponent {

    constructor(private _dialogService: DialogService) {
    }

    selectItem = (item) => {
        let result = item.row.data.id;

        if (result !== null) {
            this._doctorToken.setToken(result);
            this._doctorToken.ChangeIsFirstTime(true);

            this.ref.close();
        }
    }
}
```

تقریبا شبیه کد قبلی هست اما روش فراخونی متد تغییر کرده. تو این حالت میشه یه متد رو بصورت پارامتر پاس داد بدون اینکه به ارور گفته شده برخورد کرد
<br />
دلیل این مشکل هم این هست که کلمه کلیدی this وقتی متود رو بصورت پارامتر پاس بدیم دیگه داخل متود اشاره به کامپوننت جاری نداره
<br />
پس از چیزهایی که داخل کامپوننت تعریف شدن مثل this._doctorToken نمیشه استفاده کرد 
