---
title: "بهبود کدهای یک پروژه .net"
date: 2022-04-21T12:00:00-00:00
categories:
  - Net
tags:
  - net
  - bin
  - obj
  - bat
---

توسط کامپایلر C# که با عنوان Roslyn بازنویسی شده است. قابلیت افزونه‌پذیری به آن اضافه شده است که به شما قابلیت بررسی بیشتر کدها در زمان کامپایل و صادر کردن خطا در مواقع خاص را می‌دهد.  
بطور مثال در این مطلب کدهای قرار داده شده، پروژه را از قسمت‌های مختلف مانند Async بررسی می‌کند و خطا می‌دهد.  

کد زیر را می‌توانید در فایل `Directory.Build.props` در Root پروژه خودتان ایجاد کنید و در آن قرار دهید تا در همه زیرپروژه‌ها اعمال شود.  
وجود این PackageReference ها، به معنای بالا رفتن حجم نهایی قابل ارائه‌ی پروژه نیست؛ چون به صورت PrivateAssets و analyzers تعریف شده‌اند و فقط در حین پروسه‌ی کامپایل، جهت ارائه‌ی راهنمایی‌های بیشتر، تاثیرگذار خواهند بود.  

در کد زیر افزونه‌های کاربردی برای بررسی کد به تمام زیرپروژه‌ها اضافه می‌شود.  

```c#
<Project>
  <PropertyGroup>
    <AnalysisLevel>latest</AnalysisLevel>
    <AnalysisMode>AllEnabledByDefault</AnalysisMode>
    <CodeAnalysisTreatWarningsAsErrors>true</CodeAnalysisTreatWarningsAsErrors>
    <EnableNETAnalyzers>true</EnableNETAnalyzers>
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <Nullable>enable</Nullable>
    <!--<TreatWarningsAsErrors>true</TreatWarningsAsErrors>-->
    <RunAnalyzersDuringBuild>true</RunAnalyzersDuringBuild>
    <RunAnalyzersDuringLiveAnalysis>true</RunAnalyzersDuringLiveAnalysis>
    <!--
      CA2007: Consider calling ConfigureAwait on the awaited task
      MA0004: Use Task.ConfigureAwait(false) as the current SynchronizationContext is not needed
      CA1056: Change the type of property 'Url' from 'string' to 'System.Uri'
      CA1054: Change the type of parameter of the method to allow a Uri to be passed as a 'System.Uri' object
      CA1055: Change the return type of method from 'string' to 'System.Uri'
    -->
    <NoWarn>$(NoWarn);CA2007;CA1056;CA1054;CA1055;MA0004</NoWarn>
    <NoError>$(NoError);CA2007;CA1056;CA1054;CA1055;MA0004</NoError>
    <Deterministic>true</Deterministic>
    <Features>strict</Features>
    <ReportAnalyzer>true</ReportAnalyzer>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.NetAnalyzers" Version="6.0.0">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>

    <PackageReference Include="Meziantou.Analyzer" Version="1.0.698">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>

    <PackageReference Include="Microsoft.VisualStudio.Threading.Analyzers" Version="17.1.46">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>

    <PackageReference Include="Microsoft.CodeAnalysis.BannedApiAnalyzers" Version="3.3.3">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>

    <PackageReference Include="SecurityCodeScan" Version="3.5.4">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>

    <PackageReference Include="AsyncFixer" Version="1.5.1">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>

    <PackageReference Include="Asyncify" Version="0.9.7">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>

    <!--<PackageReference Include="ClrHeapAllocationAnalyzer" Version="3.0.0">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>-->

    <PackageReference Include="SonarAnalyzer.CSharp" Version="8.37.0.45539">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>

  <ItemGroup>
    <AdditionalFiles Include="$(MSBuildThisFileDirectory)BannedSymbols.txt" Link="Properties/BannedSymbols.txt" />
  </ItemGroup>
</Project>
```

با اضافه کردن کد بالا تعداد خطاهای برنامه بسیار زیاد می‌شود.  
در این صورت می‌توانید کد زیر را با نام `.editorconfig` در پروژه قرار دهید و موارد مورد نیاز خود را در آن اعمال کنید.  
بطور مثال dotnet_analyzer_diagnostic.category-Style.severity = warning نوع خطا رعایت نکردن استایل در کد را بصورت Warning درمی‌آورد که می‌توانید آن را به موارد دیگر مانند Suggestion و یا Error تغییر دهید.  

برای خطاهای دیگر مانند CA1308 می‌توانید آن را به کد زیر اضافه کنید و نوع آن را خودتان تعیین کنید.  


```c#
[*.cs]

# Default severity for analyzer diagnostics with category 'Style' (escalated to build warnings)
dotnet_analyzer_diagnostic.category-Style.severity = warning

# MA0026 : Complete the task
dotnet_diagnostic.MA0026.severity = suggestion

# CA1308: In method 'urlToLower', replace the call to 'ToLowerInvariant' with 'ToUpperInvariant' (CA1308)
dotnet_diagnostic.CA1308.severity = suggestion

# CA1040: Avoid empty interfaces
dotnet_diagnostic.CA1040.severity = suggestion

# CA1829 Use the "Count" property instead of Enumerable.Count()
dotnet_diagnostic.CA1829.severity = suggestion

# Use 'Count' property here instead.
dotnet_diagnostic.S2971.severity = suggestion

# S1135 : Complete the task
dotnet_diagnostic.S1135.severity = suggestion

# S2479: Replace the control character at position 7 by its escape sequence
dotnet_diagnostic.S2479.severity = suggestion

# CA2007: Consider calling ConfigureAwait on the awaited task
dotnet_diagnostic.CA2007.severity = none

# MA0004: Use Task.ConfigureAwait(false) as the current SynchronizationContext is not needed
dotnet_diagnostic.MA0004.severity = none

# CA1056: Change the type of property 'Url' from 'string' to 'System.Uri'
dotnet_diagnostic.CA1056.severity = suggestion

# CA1054: Change the type of parameter of the method to allow a Uri to be passed as a 'System.Uri' object
dotnet_diagnostic.CA1054.severity = suggestion

# CA1055: Change the return type of method from 'string' to 'System.Uri'
dotnet_diagnostic.CA1055.severity = suggestion

# S4457: Split this method into two, one handling parameters check and the other handling the asynchronous code.
dotnet_diagnostic.S4457.severity = none

# AsyncFixer01: Unnecessary async/await usage
dotnet_diagnostic.AsyncFixer01.severity = suggestion

# AsyncFixer02: Long-running or blocking operations inside an async method
dotnet_diagnostic.AsyncFixer02.severity = suggestion

# VSTHRD103: Call async methods when in an async method
dotnet_diagnostic.VSTHRD103.severity = suggestion

# AsyncFixer03: Fire & forget async void methods
dotnet_diagnostic.AsyncFixer03.severity = suggestion

# VSTHRD100: Avoid async void methods
dotnet_diagnostic.VSTHRD100.severity = suggestion

# VSTHRD101: Avoid unsupported async delegates
dotnet_diagnostic.VSTHRD101.severity = suggestion

# VSTHRD107: Await Task within using expression
dotnet_diagnostic.VSTHRD107.severity = suggestion

# AsyncFixer04: Fire & forget async call inside a using block
dotnet_diagnostic.AsyncFixer04.severity = suggestion

# VSTHRD110: Observe result of async calls
dotnet_diagnostic.VSTHRD110.severity = suggestion

# VSTHRD002: Avoid problematic synchronous waits
dotnet_diagnostic.VSTHRD002.severity = suggestion

# MA0045: Do not use blocking call (make method async)
dotnet_diagnostic.MA0045.severity = suggestion

# AsyncifyInvocation: Use Task Async
dotnet_diagnostic.AsyncifyInvocation.severity = suggestion

# AsyncifyVariable: Use Task Async
dotnet_diagnostic.AsyncifyVariable.severity = suggestion

# VSTHRD111: Use ConfigureAwait(bool)
dotnet_diagnostic.VSTHRD111.severity = none

# MA0022: Return Task.FromResult instead of returning null
dotnet_diagnostic.MA0022.severity = suggestion

# VSTHRD114: Avoid returning a null Task
dotnet_diagnostic.VSTHRD114.severity = suggestion

# VSTHRD200: Use "Async" suffix for async methods
dotnet_diagnostic.VSTHRD200.severity = suggestion

# MA0040: Specify a cancellation token
dotnet_diagnostic.MA0032.severity = suggestion

# MA0040: Flow the cancellation token when available
dotnet_diagnostic.MA0040.severity = suggestion

# MA0079: Use a cancellation token using .WithCancellation()
dotnet_diagnostic.MA0079.severity = suggestion

# MA0080: Use a cancellation token using .WithCancellation()
dotnet_diagnostic.MA0080.severity = suggestion

#AsyncFixer05: Downcasting from a nested task to an outer task.
dotnet_diagnostic.AsyncFixer05.severity = suggestion

# ClrHeapAllocationAnalyzer
# HAA0301: Closure Allocation Source
dotnet_diagnostic.HAA0301.severity = suggestion

# HAA0601: Value type to reference type conversion causing boxing allocation
dotnet_diagnostic.HAA0601.severity = suggestion

# HAA0302: Display class allocation to capture closure
dotnet_diagnostic.HAA0302.severity = suggestion

# HAA0101: Array allocation for params parameter
dotnet_diagnostic.HAA0101.severity = suggestion

# HAA0603: Delegate allocation from a method group
dotnet_diagnostic.HAA0603.severity = suggestion

# HAA0602: Delegate on struct instance caused a boxing allocation
dotnet_diagnostic.HAA0602.severity = suggestion

# HAA0401: Possible allocation of reference type enumerator
dotnet_diagnostic.HAA0401.severity = silent

# HAA0303: Lambda or anonymous method in a generic method allocates a delegate instance
dotnet_diagnostic.HAA0303.severity = silent

# HAA0102: Non-overridden virtual method call on value type
dotnet_diagnostic.HAA0102.severity = silent

# HAA0502: Explicit new reference type allocation
dotnet_diagnostic.HAA0502.severity = none

# HAA0505: Initializer reference type allocation
dotnet_diagnostic.HAA0505.severity = silent

# MA0009: Regular expressions
dotnet_diagnostic.MA0009.severity = suggestion

# S1118: Protected constructor
dotnet_diagnostic.S1118.severity = suggestion

# CA1052: Static holder
dotnet_diagnostic.CA1052.severity = suggestion

# CA1008: Add a member to SubscriptionStartType that has a value of zero with a suggested name of 'None'
dotnet_diagnostic.CA1008.severity = suggestion

# S4035: Seal class 'PasswordComplexitySetting' or implement 'IEqualityComparer<T>' instead
dotnet_diagnostic.S4035.severity = suggestion

# S3897: Implement 'IEquatable<PasswordComplexitySetting>'
dotnet_diagnostic.S3897.severity = suggestion

# MA0077: A class that provides Equals(T) should implement IEquatable<T>
dotnet_diagnostic.MA0077.severity = suggestion

# CA1034: Do not nest type Host. Alternatively, change its accessibility so that it is not externally visible.
dotnet_diagnostic.CA1034.severity = suggestion

# MA0048: File name must match type name
dotnet_diagnostic.MA0048.severity = suggestion

# CA1014: Mark assemblies with CLSCompliant
dotnet_diagnostic.CA1014.severity = suggestion

# CS0162: Unrichebele code
dotnet_diagnostic.CS0162.severity = suggestion

# MA0049: Type name should not match containing namespace
dotnet_diagnostic.MA0049.severity = suggestion
```

فایل زیر را می‌توانید با نام `BannedSymbols.txt` ذخیره کنید تا موارد گفته شده در آن مانند استفاده از UtcNow بجای Now اجباری شود.  
این فایل نیز در آخر تکه کد اول معرفی شده است.  

```c#
P:System.DateTime.Now;Use System.DateTime.UtcNow instead
P:System.DateTimeOffset.Now;Use System.DateTimeOffset.UtcNow instead
P:System.DateTimeOffset.DateTime;Use System.DateTimeOffset.UtcDateTime instead
```

لینک‌های مفید:  

[editorconfig](https://editorconfig.org/)  
[](https://docs.microsoft.com/en-us/visualstudio/ide/create-portable-custom-editor-options?view=vs-2022)  
[use-roslyn-analyzers](https://docs.microsoft.com/en-us/visualstudio/code-quality/use-roslyn-analyzers?view=vs-2022)  
[configuration-options](https://docs.microsoft.com/en-us/dotnet/fundamentals/code-analysis/configuration-options)  
[]()  
[]()  