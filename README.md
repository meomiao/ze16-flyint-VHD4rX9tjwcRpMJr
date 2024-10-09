
Microsoft 于 [2024 年 8 月 30 日](https://github.com)\[1]宣布推出 ASP.NET Core OData 9 包。 这个新包将ASP.NET Core与.NET 8 OData库保持一致，改变了OData格式中数据编码的内部细节，使其更符合[OData 规范](https://github.com)\[2]。

在2024年8月早些时候，Microsoft [将 OData .NET 库更新到版本 8\.0\.0](https://github.com)\[1]。其中最重要的更改是放弃了对旧版 .NET Framework 的支持。从此版本开始，将仅支持 .NET 8 及更高版本。使用旧版 .NET Framework 的开发人员仍然可以使用 OData 库的 7\.x 版，这些库[在 2025 年 3 月之前仍受到积极支持](https://github.com)\[3]，届时他们将处于维护模式。

OData 8库使用了新的JSON写入器[System.Text.Utf8JsonWriter](https://github.com)\[4]来序列化和反序列化JSON负载，这个新写入器比旧的`JsonWriter`更快且需要更少的内存。并且需要的内存更少，由 [Microsoft.OData.Json.DefaultJsonWriterFactory](https://github.com)\[5] 创建，因为它不是基于`JsonWriter` 而是基于TextWriter .虽然新编写器自 OData 版本 7\.12\.2 以来一直可用，但现在它是 OData 8 中的默认实现。

如果需要，开发人员仍然可以使用旧编写器，方法是在服务生成器中调用 AddOData 方法并提供一个实例，该实例对应于旧编写器，为清楚起见，已重命名。

builder.Services.AddControllers().AddOData(options \=\> options.EnableQueryFeatures().AddRouteComponents(routePrefix: string.Empty, model: modelBuilder.GetEdmModel(), configureServices: (services) \=\>
{
 services.AddScoped \< Microsoft.OData.Json.IJsonWriterFactory \> (sp \=\> new Microsoft.OData.Json.ODataJsonWriterFactory());
}));

新编写器的序列化方式与旧编写器不同。它不会像较旧的编写器那样对所有高 ASCII Unicode 字符进行编码。例如，它不会将非拉丁符号（如希腊字母）编码为 Unicode 数字序列。相反，它将输出 Unicode 字符本身。旧编写器会将几乎所有非 ASCII 字符编码为数字，从而使有效负载的大小更大，编码过程更慢。新的 JSON 编写器输出大写 Unicode 字符，而不是以前版本使用的小写。

ASP.NET Core OData 9的另一个重大变化是依赖注入的工作方式，更新后的库使用与.NET相同的抽象，即`IServiceProvider`。

builder.Services.AddControllers().AddOData(options \=\> options.EnableQueryFeatures().AddRouteComponents(routePrefix: string.Empty, model: modelBuilder.GetEdmModel(), configureServices: (services) \=\>
{
 services.AddDefaultODataServices(odataVersion: Microsoft.OData.ODataVersion.V4, configureReaderAction: (messageReaderSettings) \=\>
{
// Relevant changes to the ODataMessageReaderSettings instance here
}, configureWriterAction: (messageWriterSettings) \=\>
{
// Relevant changes to the ODataMessageWriterSettings instance here
}, configureUriParserAction: (uriParserSettings) \=\>
{
// // Relevant changes to the ODataUriParserSettings instance here
});
}));

此外，新库还移除了旧的实现和标准，如JSONP格式。新的 ASP.NET Core OData 9 库作为 [NuGet 包\[6]](https://github.com)分发。OData 通过各种 NuGet 包提供，包括：

* [Microsoft.OData.Core 8\.0\.1](https://github.com)
* [Microsoft.OData.Edm 8\.0\.1](https://github.com)
* [Microsoft.OData.Client 8\.0\.1](https://github.com)
* [Microsoft.Spatial 8\.0\.1](https://github.com):[蓝猫机场](https://fenfang.org)

新版本在过去9 周内 已被下载了 250\.000 次。ASP.NET Core OData 的源代码在 [GitHub 上提供](https://github.com)\[7]，存储库目前有 458 个未解决的问题,有关完整列表，开发人员可以查看 [OData 8 .NET 库的发行说明](https://github.com)\[8]。

**相关链接**

* \[1]Announcing ASP.NET Core OData 9 Official Release:[https://devblogs.microsoft.com/odata/announcing\-asp\-net\-core\-odata\-9\-official\-release/](https://devblogs.microsoft.com/odata/announcing-asp-net-core-odata-9-official-release/ "https://devblogs.microsoft.com/odata/announcing-asp-net-core-odata-9-official-release/")
* \[2]OData 规范:[https://groups.oasis\-open.org/communities/tc\-community\-home2?CommunityKey\=e7cac2a9\-2d18\-4640\-b94d\-018dc7d3f0e2](https://groups.oasis-open.org/communities/tc-community-home2?CommunityKey=e7cac2a9-2d18-4640-b94d-018dc7d3f0e2 "https://groups.oasis-open.org/communities/tc-community-home2?CommunityKey=e7cac2a9-2d18-4640-b94d-018dc7d3f0e2")
* \[3]在 2025 年 3 月之前仍受到积极支持: [https://learn.microsoft.com/en\-us/odata/support/support\-policy](https://learn.microsoft.com/en-us/odata/support/support-policy "https://learn.microsoft.com/en-us/odata/support/support-policy")
* \[4]System.Text.Utf8JsonWriter: [https://learn.microsoft.com/en\-us/dotnet/api/system.text.json.utf8jsonwriter?view\=net\-8\.0](https://learn.microsoft.com/en-us/dotnet/api/system.text.json.utf8jsonwriter?view=net-8.0 "https://learn.microsoft.com/en-us/dotnet/api/system.text.json.utf8jsonwriter?view=net-8.0")
* \[5]Microsoft.OData.Json.DefaultJsonWriterFactory:[https://learn.microsoft.com/en\-us/dotnet/api/microsoft.odata.json.defaultjsonwriterfactory?view\=odata\-core\-7\.0](https://learn.microsoft.com/en-us/dotnet/api/microsoft.odata.json.defaultjsonwriterfactory?view=odata-core-7.0 "https://learn.microsoft.com/en-us/dotnet/api/microsoft.odata.json.defaultjsonwriterfactory?view=odata-core-7.0")
* \[6]ASP.NET Core OData 9 nuget: [https://www.nuget.org/packages/Microsoft.AspNetCore.OData/9\.0\.0](https://www.nuget.org/packages/Microsoft.AspNetCore.OData/9.0.0 "https://www.nuget.org/packages/Microsoft.AspNetCore.OData/9.0.0")
* \[7]GitHub 上提供:[https://github.com/OData/AspNetCoreOData](https://github.com/OData/AspNetCoreOData "https://github.com/OData/AspNetCoreOData")
* \[8]Announcing OData .NET 8 Official Release:[https://devblogs.microsoft.com/odata/announcing\-odata\-net\-8\-official\-release/](https://devblogs.microsoft.com/odata/announcing-odata-net-8-official-release/ "https://devblogs.microsoft.com/odata/announcing-odata-net-8-official-release/")


