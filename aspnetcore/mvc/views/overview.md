---
title: ASP.NET Core MVC 'deki görünümler
author: ardalis
description: Görünümlerin ASP.NET Core MVC 'de uygulamanın veri sunumunu ve kullanıcı etkileşimini nasıl işleyeceğinizi öğrenin.
ms.author: riande
ms.date: 12/05/2019
uid: mvc/views/overview
ms.openlocfilehash: f636908ee36d0af6e92875876240cb8712dd2ccc
ms.sourcegitcommit: c0b72b344dadea835b0e7943c52463f13ab98dd1
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 12/06/2019
ms.locfileid: "74881031"
---
# <a name="views-in-aspnet-core-mvc"></a>ASP.NET Core MVC 'deki görünümler

[Steve Smith](https://ardalis.com/) ve [Luke Latham](https://github.com/guardrex) tarafından

Bu belgede ASP.NET Core MVC uygulamalarında kullanılan görünümler açıklanmaktadır. Razor Pages hakkında bilgi için bkz. [Razor Pages giriş](xref:razor-pages/index).

Model-View-Controller (MVC) modelinde, *Görünüm* uygulamanın veri sunumunu ve kullanıcı etkileşimini işler. Görünüm, gömülü [Razor işaretlemesi](xref:mvc/views/razor)IÇEREN bir HTML şablonudur. Razor işaretleme, istemciye gönderilen bir Web sayfası oluşturmak için HTML işaretlemesi ile etkileşen koddur.

ASP.NET Core MVC 'de, görünümler Razor biçimlendirmesinde [ C# programlama dilini](/dotnet/csharp/) kullanan *. cshtml* dosyalarıdır. Genellikle, görünüm dosyaları uygulama [denetleyicilerinin](xref:mvc/controllers/actions)her biri için adlandırılmış klasörler halinde gruplandırılır. Klasörler, uygulamanın kökündeki bir *Görünümler* klasöründe depolanır:

![Visual Studio Çözüm Gezgini Görünüm klasörü,. cshtml, Contact. cshtml ve Index. cshtml dosyalarını göstermek için açık giriş klasörüyle açıktır](overview/_static/views_solution_explorer.png)

*Ana* denetleyici, *Görünümler* klasörünün içindeki bir *giriş* klasörüyle temsil edilir. *Giriş* klasörü, *hakkında*, *iletişim*ve *Dizin* (giriş sayfası) Web sayfalarının görünümlerini içerir. Bir Kullanıcı bu üç Web sayfasından birini istediğinde, *ana* denetleyicideki denetleyici eylemleri, Kullanıcı için bir Web sayfası oluşturmak ve döndürmek için kullanılan üç görünümden hangisinin kullanıldığını tespit ediyor.

Tutarlı Web sayfası bölümleri sağlamak ve kod tekrarlamayı azaltmak için [düzenleri](xref:mvc/views/layout) kullanın. Düzenler genellikle üstbilgiyi, gezinti ve menü öğelerini ve alt bilgisini içerir. Üst bilgi ve altbilgi genellikle birçok meta veri öğesi için ortak biçimlendirme ve betik ve stil varlıklarına bağlantılar içerir. Düzenler, görünümlerinizde bu ortak biçimlendirmeyi önlemenize yardımcı olur.

[Kısmi görünümler](xref:mvc/views/partial) , görünümlerin yeniden kullanılabilir parçalarını yöneterek kod yinelemeyi azaltır. Örneğin, kısmi bir görünüm çeşitli görünümlerde görüntülenen bir blog web sitesinde yazar biyografı için yararlıdır. Yazar biyografları sıradan görünüm içeriğine sahiptir ve Web sayfasının içeriğini üretmek için kodun yürütülmesi gerekmez. Yazar biyografları, tek başına model bağlama tarafından kullanılabilir, bu nedenle bu içerik türü için kısmi bir görünüm kullanmak idealdir.

[Görünüm bileşenleri](xref:mvc/views/view-components) , yinelenen kodu azaltmanıza izin veren kısmi görünümlere benzerdir, ancak Web sayfasını işlemek için kodun sunucuda çalıştırılmasını gerektiren içeriği görüntüleme için uygundur. Görüntüleme bileşenleri, işlenen içerik bir Web sitesi alışveriş sepeti gibi veritabanı etkileşimi gerektirdiğinde yararlıdır. Web sayfası çıkışı oluşturmak için, görünüm bileşenlerini model bağlama ile sınırlı değildir.

## <a name="benefits-of-using-views"></a>Görünümleri kullanmanın avantajları

Görünümler, Kullanıcı arabirimi işaretlemesini uygulamanın diğer bölümlerinden ayırarak bir MVC uygulamasında [kaygıların ayrılmasını](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns) sağlamaya yardımcı olur. Aşağıdaki SoC tasarımı, uygulamanızın modüler olmasını sağlayarak çeşitli avantajlar sağlar:

* Daha iyi bir şekilde düzenlendiğinden, uygulamanın bakımını daha kolay hale getirir. Görünümler genellikle uygulama özelliğine göre gruplandırılır. Bu, bir özellik üzerinde çalışırken ilgili görünümleri bulmayı kolaylaştırır.
* Uygulamanın parçaları gevşek olarak bağlanmış. Uygulamanın görünümlerini iş mantığı ve veri erişim bileşenlerinden ayrı olarak oluşturup güncelleştirebilirsiniz. Uygulamanın diğer bölümlerini güncelleştirmek zorunda kalmadan uygulamanın görünümlerini değiştirebilirsiniz.
* Görünümler ayrı birimler olduğundan uygulamanın kullanıcı arabirimi parçalarını test etmek daha kolay.
* Daha iyi bir kuruluş nedeniyle, Kullanıcı arabiriminin bölümlerini yanlışlıkla tekrarlamanız daha az olabilir.

## <a name="creating-a-view"></a>Görünüm oluşturma

Denetleyiciye özgü görünümler, *Görünümler/[ControllerName]* klasöründe oluşturulur. Denetleyiciler arasında paylaşılan görünümler *Görünümler/paylaşılan* klasörüne yerleştirilir. Bir görünüm oluşturmak için yeni bir dosya ekleyin ve *. cshtml* dosya uzantısıyla ilişkili denetleyici eylemiyle aynı adı verin. *Giriş* denetleyicisindeki *hakkında* eylemine karşılık gelen bir görünüm oluşturmak Için, *Görünümler/giriş* klasöründe bir *About. cshtml* dosyası oluşturun:

[!code-cshtml[](../../common/samples/WebApplication1/Views/Home/About.cshtml)]

*Razor* biçimlendirmesi `@` simgesiyle başlar. İfadeleri C# , C# [Razor kod blokları](xref:mvc/views/razor#razor-code-blocks) içine kod yerleştirerek küme ayraçları (`{ ... }`) tarafından kapalı olarak ayarlayarak çalıştırın. Örneğin, yukarıda gösterilen `ViewData["Title"]` için "hakkında" atamasını inceleyin. Yalnızca `@` simgesiyle bir değere başvurarak, HTML içindeki değerleri görüntüleyebilirsiniz. Yukarıdaki `<h2>` ve `<h3>` öğelerinin içeriğine bakın.

Yukarıda gösterilen görünüm içeriği yalnızca kullanıcıya işlenmiş olan tüm Web sayfasının bir parçasıdır. Sayfanın düzeninin geri kalanı ve görünümün diğer yaygın yönleri diğer görünüm dosyalarında belirtilir. Daha fazla bilgi için [Düzen konusuna](xref:mvc/views/layout)bakın.

## <a name="how-controllers-specify-views"></a>Denetleyiciler görünümleri nasıl belirler

Görünümler genellikle eylemlerden bir [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult)türü olan [ViewResult](/dotnet/api/microsoft.aspnetcore.mvc.viewresult)olarak döndürülür. Eylem yönteminiz doğrudan bir `ViewResult` oluşturup döndürebilir, ancak yaygın olarak yapılmaz. Çoğu [Denetleyici denetleyicisinden](/dotnet/api/microsoft.aspnetcore.mvc.controller)devraldığı için, `ViewResult`döndürmek üzere `View` yardımcı yöntemini kullanmanız yeterlidir:

*HomeController.cs*

[!code-csharp[](../../common/samples/WebApplication1/Controllers/HomeController.cs?highlight=5&range=16-21)]

Bu eylem döndüğünde, son bölümde gösterilen *hakkında. cshtml* görünümü aşağıdaki Web sayfası olarak işlenir:

![Edge tarayıcısında işlenen sayfa hakkında](overview/_static/about-page.png)

`View` Yardımcısı yönteminde birkaç aşırı yükleme vardır. İsteğe bağlı olarak şunları belirtebilirsiniz:

* Döndürülecek açık bir görünüm:

  ```csharp
  return View("Orders");
  ```

* Görünüme geçirilecek bir [model](xref:mvc/models/model-binding) :

  ```csharp
  return View(Orders);
  ```

* Hem görünüm hem de model:

  ```csharp
  return View("Orders", Orders);
  ```

### <a name="view-discovery"></a>Bulmayı görüntüle

Bir eylem bir görünüm döndürdüğünde, *görünüm bulma* adlı bir işlem gerçekleşir. Bu işlem, görünüm adına göre hangi görünüm dosyasının kullanıldığını belirler. 

`View` yönteminin varsayılan davranışı (`return View();`), çağrılan eylem yöntemiyle aynı ada sahip bir görünüm döndürmemelidir. Örneğin, denetleyicinin yaklaşık *`ActionResult` Yöntem* adı *. cshtml*adlı bir görünüm dosyasını aramak için kullanılır. İlk olarak, çalışma zamanı görünümün *Görünümler/[ControllerName]* klasörüne bakar. Burada eşleşen bir görünüm bulamazsa, görünümün *paylaşılan* klasörünü arar.

`ViewResult` örtülü olarak `return View();` döndürmeniz ya da görünüm adını `return View("<ViewName>");`ile `View` yöntemine açıkça geçirmeniz önemlidir. Her iki durumda da, eşleşen bir görünüm dosyası için bulma aramalarını şu sırayla görüntüleyin:

   1. *Görünümler/\[ControllerName]/\[ViewName]. cshtml*
   1. *Görünümler/paylaşılan/\[ViewName]. cshtml*

Görünüm adı yerine bir görünüm dosyası yolu sağlanıyor. Uygulama kökünde başlayan mutlak bir yol kullanılıyorsa (isteğe bağlı olarak "/" veya "~/" ile başlayan), *. cshtml* uzantısı belirtilmelidir:

```csharp
return View("Views/Home/About.cshtml");
```

Ayrıca, *. cshtml* uzantısı olmadan farklı dizinlerdeki görünümleri belirtmek için göreli bir yol da kullanabilirsiniz. `HomeController`içinde, *Yönetim* görünümlerinizin *Dizin* görünümünü göreli bir yol ile döndürebilirsiniz:

```csharp
return View("../Manage/Index");
```

Benzer şekilde, "./" önekiyle geçerli denetleyiciye özgü dizini belirtebilirsiniz:

```csharp
return View("./About");
```

[Kısmi görünümler](xref:mvc/views/partial) ve [Görünüm bileşenleri](xref:mvc/views/view-components) benzer (ancak aynı) bulma mekanizmalarını kullanır.

Özel bir [ıviewlocationgenişleticisi](/dotnet/api/microsoft.aspnetcore.mvc.razor.iviewlocationexpander)kullanarak, görünümler uygulama içinde nasıl konumlandırılabilir varsayılan kuralı özelleştirebilirsiniz.

Görünüm bulma, dosya adına göre görünüm dosyalarını bulmayı kullanır. Temeldeki dosya sistemi büyük/küçük harfe duyarlı ise, görünüm adları büyük olasılıkla büyük küçük harfe duyarlıdır. İşletim sistemleri arasında uyumluluk için, denetleyici ve eylem adları ile ilişkili görünüm klasörleri ve dosya adları arasındaki büyük/küçük harfe eşleştirin. Büyük/küçük harfe duyarlı dosya sistemiyle çalışırken bir görünüm dosyasının bulunamadığını belirten bir hatayla karşılaşırsanız, istenen görünüm dosyası ile gerçek görünüm dosyası adı arasında büyük/küçük harf eşleştiğini doğrulayın.

Görünümlerinizin dosya yapısını, bakım ve açıklık için denetleyiciler, Eylemler ve görünümler arasındaki ilişkileri yansıtacak şekilde düzenleme konusunda en iyi yöntemi izleyin.

## <a name="passing-data-to-views"></a>Verileri görünümlere geçirme

Çeşitli yaklaşımlar kullanarak verileri görünümlere geçirin:

* Kesin türü belirtilmiş veri: ViewModel
* Zayıf yazılmış veriler
  * `ViewData` (`ViewDataAttribute`)
  * `ViewBag`

### <a name="strongly-typed-data-viewmodel"></a>Kesin tür belirtilmiş veriler (ViewModel)

En güçlü yaklaşım, görünümde bir [model](xref:mvc/models/model-binding) türü belirtmektir. Bu model genellikle *ViewModel*olarak adlandırılır. ViewModel türünün bir örneğini eylemden görünüme geçirirsiniz.

Görünümü bir görünüme aktarmak için ViewModel kullanmak, görünümün *tanımlayıcı* tür denetlemesinin avantajlarından yararlanmasını sağlar. *Kesin yazma* (veya *türü kesin belirlenmiş*), her değişken ve sabitin açıkça tanımlanmış bir tür (örneğin, `string`, `int`veya `DateTime`) olduğu anlamına gelir. Bir görünümde kullanılan türlerin geçerliliği derleme zamanında denetlenir.

[Visual Studio](https://visualstudio.microsoft.com) ve [Visual Studio Code](https://code.visualstudio.com/) listesi [IntelliSense](/visualstudio/ide/using-intellisense)adlı bir özellik kullanılarak türü kesin belirlenmiş sınıf üyeleridir. ViewModel özelliklerini görmek istediğinizde, ViewModel için değişken adını ve ardından bir nokta (`.`) yazın. Bu, daha az hata vererek kodu daha hızlı yazmanıza yardımcı olur.

`@model` yönergesini kullanarak bir model belirtin. Modeli `@Model`ile birlikte kullanın:

```cshtml
@model WebApplication1.ViewModels.Address

<h2>Contact</h2>
<address>
    @Model.Street<br>
    @Model.City, @Model.State @Model.PostalCode<br>
    <abbr title="Phone">P:</abbr> 425.555.0100
</address>
```

Modeli görünüme sağlamak için, denetleyici bu parametreyi bir parametre olarak geçirir:

```csharp
public IActionResult Contact()
{
    ViewData["Message"] = "Your contact page.";

    var viewModel = new Address()
    {
        Name = "Microsoft",
        Street = "One Microsoft Way",
        City = "Redmond",
        State = "WA",
        PostalCode = "98052-6399"
    };

    return View(viewModel);
}
```

Bir görünüme sağlayabilmeniz için model türlerinde hiçbir kısıtlama yoktur. Basit eski CLR nesnesi (POCO) viewmodeller için çok az veya hiç davranış (Yöntem) tanımlanmış olarak kullanılması önerilir. Genellikle ViewModel sınıfları, uygulamanın kökündeki *modeller* klasöründe veya ayrı bir *viewmodeller* klasöründe depolanır. Yukarıdaki örnekte kullanılan *Adres* viewmodel, *Address.cs*adlı bir dosyada depolanan bir poco ViewModel modelidir:

```csharp
namespace WebApplication1.ViewModels
{
    public class Address
    {
        public string Name { get; set; }
        public string Street { get; set; }
        public string City { get; set; }
        public string State { get; set; }
        public string PostalCode { get; set; }
    }
}
```

Hiçbir şey, hem ViewModel türleriniz hem de iş modeli türleriniz için aynı sınıfları kullanmanızı önler. Ancak, ayrı modeller kullanmak görünümlerinizin, uygulamanızın iş mantığı ve veri erişimi bölümlerinden bağımsız olarak değişiklik yapmasına izin verir. Modeller ve ViewModel ayrımı, modeller Kullanıcı tarafından uygulamaya gönderilen veriler için [model bağlama](xref:mvc/models/model-binding) ve [doğrulama](xref:mvc/models/validation) kullanırken de güvenlik avantajları sağlar.

<a name="VD_VB"></a>

### <a name="weakly-typed-data-viewdata-viewdata-attribute-and-viewbag"></a>Zayıf yazılmış veriler (ViewData, ViewData özniteliği ve ViewBag)

`ViewBag` *Razor Pages kullanılamaz.*

Türü kesin belirlenmiş görünümlere ek olarak, görünümler, verilerin bir *zayıf türü* olan ( *gevşek olarak yazılmış*) bir veri koleksiyonu erişimi vardır. Güçlü türlerin aksine, *zayıf türler* (veya *gevşek türler*), kullanmakta olduğunuz veri türünü açıkça bildirmeyeceğiniz anlamına gelir. Denetleyicilerde ve görünümlerde küçük miktarlarda veri iletmek için zayıf yazılmış verilerin toplanmasını kullanabilirsiniz.

| Verileri bir... arasında geçirme                        | Örnek                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| Denetleyici ve görünüm                             | Açılan listeyi verilerle doldurma.                                          |
| Görünüm ve [Düzen görünümü](xref:mvc/views/layout)   | **\<başlığını** bir görünüm dosyasındaki düzen görünümünde > öğe içeriğini ayarlama.  |
| [Kısmi görünüm](xref:mvc/views/partial) ve görünüm | Kullanıcı tarafından istenen web sayfasını temel alan verileri görüntüleyen pencere öğesi.      |

Bu koleksiyona, denetleyiciler ve görünümlerde `ViewData` ya da `ViewBag` özellikleri aracılığıyla başvurulabilir. `ViewData` özelliği, Zayıf yazılmış nesnelerin bir sözlüğüdür. `ViewBag` özelliği, temel `ViewData` koleksiyonu için dinamik özellikler sağlayan `ViewData` çevresindeki bir sarmalayıcıdır. Note: anahtar aramaları hem `ViewData` hem de `ViewBag`için büyük/küçük harfe duyarsızdır.

`ViewData` ve `ViewBag`, çalışma zamanında dinamik olarak çözümlenir. Derleme zamanı tür denetimi sunmadığı için, her ikisi de bir ViewModel kullanmaktan daha fazla hataya açıktır. Bu nedenle, bazı geliştiriciler `ViewData` ve `ViewBag`en düşük düzeyde bir süre önce kullanmaz veya hiç kullanmayın.

<a name="VD"></a>

**ViewData**

`ViewData`, `string` anahtarlar aracılığıyla erişilen bir [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) nesnesidir. Dize verileri doğrudan bir dönüştürme gerektirmeden depolanabilir ve kullanılabilir, ancak diğer `ViewData` nesne değerlerini ayıkladığınızda belirli türlere atamalısınız. `ViewData` kullanarak, [kısmen görünümler](xref:mvc/views/partial) ve [düzenler](xref:mvc/views/layout)dahil olmak üzere denetleyicilerden görünümlere ve görünümlere veri geçirebilirsiniz.

Aşağıda, bir bir eylem içinde `ViewData` kullanarak bir selamlama ve adres değerlerini ayarlayan bir örnek verilmiştir:

```csharp
public IActionResult SomeAction()
{
    ViewData["Greeting"] = "Hello";
    ViewData["Address"]  = new Address()
    {
        Name = "Steve",
        Street = "123 Main St",
        City = "Hudson",
        State = "OH",
        PostalCode = "44236"
    };

    return View();
}
```

Bir görünümdeki verilerle çalışın:

```cshtml
@{
    // Since Address isn't a string, it requires a cast.
    var address = ViewData["Address"] as Address;
}

@ViewData["Greeting"] World!

<address>
    @address.Name<br>
    @address.Street<br>
    @address.City, @address.State @address.PostalCode
</address>
```

::: moniker range=">= aspnetcore-2.1"

**ViewData özniteliği**

[ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) ' i kullanan başka bir yaklaşım da [viewdataattribute](/dotnet/api/microsoft.aspnetcore.mvc.viewdataattribute). `[ViewData]` özniteliğiyle işaretlenmiş denetleyiciler veya Razor sayfa modelleriyle ilgili özellikler, değerlerinin depolandığı ve sözlükten yüklendiği değerlerdir.

Aşağıdaki örnekte, giriş denetleyicisi `[ViewData]`işaretli bir `Title` özelliği içerir. `About` yöntemi, hakkında bilgi görünümü için başlığı ayarlar:

```csharp
public class HomeController : Controller
{
    [ViewData]
    public string Title { get; set; }

    public IActionResult About()
    {
        Title = "About Us";
        ViewData["Message"] = "Your application description page.";

        return View();
    }
}
```

Hakkında görünümünde, `Title` özelliğine model özelliği olarak erişin:

```cshtml
<h1>@Model.Title</h1>
```

Mizanpajda, başlık ViewData sözlüğünden okundu:

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"] - WebApplication</title>
    ...
```

::: moniker-end

**ViewBag**

`ViewBag` *Razor Pages kullanılamaz.*

`ViewBag`, `ViewData`depolanan nesnelere dinamik erişim sağlayan bir [Dynamicviewdata](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.internal.dynamicviewdata) nesnesidir. `ViewBag`, atama gerektirmediğinden, ile çalışmak daha uygun olabilir. Aşağıdaki örnek, yukarıdaki `ViewData` kullanmayla aynı sonuçla `ViewBag` nasıl kullanacağınızı gösterir:

```csharp
public IActionResult SomeAction()
{
    ViewBag.Greeting = "Hello";
    ViewBag.Address  = new Address()
    {
        Name = "Steve",
        Street = "123 Main St",
        City = "Hudson",
        State = "OH",
        PostalCode = "44236"
    };

    return View();
}
```

```cshtml
@ViewBag.Greeting World!

<address>
    @ViewBag.Address.Name<br>
    @ViewBag.Address.Street<br>
    @ViewBag.Address.City, @ViewBag.Address.State @ViewBag.Address.PostalCode
</address>
```

**ViewData ve ViewBag 'i aynı anda kullanma**

`ViewBag` *Razor Pages kullanılamaz.*

`ViewData` ve `ViewBag` aynı temel `ViewData` koleksiyonuna başvurduğundan, değerleri okurken ve yazarken aralarında hem `ViewData` hem de `ViewBag` ve karıştırma ve eşleştirme kullanabilirsiniz.

Bir *About. cshtml* görünümünün en üstündeki `ViewData` kullanarak `ViewBag` ve açıklamayı kullanarak başlığı ayarlayın:

```cshtml
@{
    Layout = "/Views/Shared/_Layout.cshtml";
    ViewBag.Title = "About Contoso";
    ViewData["Description"] = "Let us tell you about Contoso's philosophy and mission.";
}
```

Özellikleri okuyun, ancak `ViewData` ve `ViewBag`kullanımını ters çevirin. *_Layout. cshtml* dosyasında, `ViewData` kullanarak başlığı alın ve `ViewBag`kullanarak açıklamayı alın:

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"]</title>
    <meta name="description" content="@ViewBag.Description">
    ...
```

Dizelerin `ViewData`için bir dönüştürme gerektirmemenizi unutmayın. `@ViewData["Title"]`, atama olmadan kullanabilirsiniz.

Hem `ViewData` hem de `ViewBag` aynı anda kullanmak, özellikleri karıştırarak ve eşleştirirken de geçerlidir. Aşağıdaki biçimlendirme işlenir:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>About Contoso</title>
    <meta name="description" content="Let us tell you about Contoso's philosophy and mission.">
    ...
```

**ViewData ve ViewBag arasındaki farkların Özeti**

 `ViewBag` Razor Pages yok.

* `ViewData`
  * [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary)öğesinden türetilir, bu nedenle `ContainsKey`, `Add`, `Remove`ve `Clear`gibi yararlı olabilecek Sözlük özellikleri vardır.
  * Sözlükteki anahtarlar dizelerdir, bu nedenle boşluğa izin verilir. Örnek: `ViewData["Some Key With Whitespace"]`
  * `string` dışında herhangi bir tür `ViewData`kullanmak için görünümde tür dönüştürme yapılmalıdır.
* `ViewBag`
  * [Dynamicviewdata](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.internal.dynamicviewdata)öğesinden türetilir, bu nedenle, nokta gösterimini (`@ViewBag.SomeKey = <value or object>`) kullanarak dinamik özellikler oluşturulmasına izin verir ve hiçbir atama gerekmez. `ViewBag` söz dizimi, denetleyicilere ve görünümlere daha hızlı eklemenizi sağlar.
  * Null değerleri denetlemek için daha basittir. Örnek: `@ViewBag.Person?.Name`

**ViewData veya ViewBag ne zaman kullanılır?**

Hem `ViewData` hem de `ViewBag`, denetleyiciler ve görünümler arasında küçük miktarlarda veri iletmek için eşit ölçüde geçerli yaklaşımlar. Hangisinin kullanılacağı seçimi tercihi temel alır. `ViewData` ve `ViewBag` nesneleri karıştırıp eşleştirebilirsiniz, ancak kod, sürekli olarak kullanılan tek bir yaklaşımla daha kolay okunabilir ve devam edebilir. Her iki yaklaşım da çalışma zamanında dinamik olarak çözümlenir ve bu nedenle çalışma zamanı hatalarına neden olur. Bazı geliştirme ekipleri bunlardan kaçınır.

### <a name="dynamic-views"></a>Dinamik Görünümler

`@model` kullanarak bir model türü bildirmeyen ancak bunlara bir model örneği geçirmeyen görünümler (örneğin, `return View(Address);`), örnek özelliklerine dinamik olarak başvurabilir:

```cshtml
<address>
    @Model.Street<br>
    @Model.City, @Model.State @Model.PostalCode<br>
    <abbr title="Phone">P:</abbr> 425.555.0100
</address>
```

Bu özellik esneklik sunar, ancak derleme koruması veya IntelliSense sunmaz. Özellik yoksa, Web sayfası oluşturma çalışma zamanında başarısız olur.

## <a name="more-view-features"></a>Daha fazla görünüm özelliği

[Etiket Yardımcıları](xref:mvc/views/tag-helpers/intro) , var olan HTML etiketlerine sunucu tarafı davranışı eklemenizi kolaylaştırır. Etiket yardımcılarının kullanılması, görünümleriniz içinde özel kod veya yardımcılar yazma ihtiyacını önler. Etiket Yardımcıları HTML öğelerine öznitelikler olarak uygulanır ve bunları işleyeamayan düzenleyiciler tarafından yok sayılır. Bu, çeşitli araçlarındaki görünüm işaretlemesini düzenlemenizi ve işlemeyi sağlar.

Özel HTML işaretlemesi oluşturmak birçok yerleşik HTML Yardımcıda sağlanabilir. Daha karmaşık kullanıcı arabirimi mantığı, [Görünüm bileşenleri](xref:mvc/views/view-components)tarafından işlenebilir. Görüntüleme bileşenleri, denetleyiciler ve görünümler tarafından sunulan aynı SoC öğesine sahiptir. Ortak kullanıcı arabirimi öğeleri tarafından kullanılan verilerle ilgili eylemler ve görünümler gereksinimini ortadan kaldırabilir.

ASP.NET Core diğer birçok yönü gibi, görünümler [bağımlılık ekleme](xref:fundamentals/dependency-injection)işlemini destekler ve hizmetlerin [görünümlere](xref:mvc/views/dependency-injection)eklenmesine izin verir.
