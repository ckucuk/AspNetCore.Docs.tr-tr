---
title: ASP.NET Core 'de uygulama modeliyle çalışma
author: ardalis
description: MVC öğelerinin ASP.NET Core nasıl davranacağını değiştirmek için uygulama modelini okumayı ve işlemeyi öğrenin.
ms.author: riande
ms.date: 12/05/2019
uid: mvc/controllers/application-model
ms.openlocfilehash: 4b6c978e5752eb320412a1c204df8e3d288fe4a1
ms.sourcegitcommit: c0b72b344dadea835b0e7943c52463f13ab98dd1
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 12/06/2019
ms.locfileid: "74881086"
---
# <a name="work-with-the-application-model-in-aspnet-core"></a>ASP.NET Core 'de uygulama modeliyle çalışma

[Steve Smith](https://ardalis.com/) tarafından

ASP.NET Core MVC, MVC uygulamasının bileşenlerini temsil eden bir *uygulama modeli* tanımlar. MVC öğelerinin nasıl davrandığını değiştirmek için bu modeli okuyabilir ve düzenleyebilirsiniz. Varsayılan olarak, MVC, hangi sınıfların denetleyici olduğunu belirlemek için bazı kuralları izler, bu sınıfların hangi yöntemlerin eylem olduğunu ve parametrelerin ve yönlendirmenin nasıl davranacağını sağlar. Kendi kurallarınızı oluşturarak ve bunları küresel olarak veya öznitelik olarak uygulayarak, uygulamanızın gereksinimlerine uyacak şekilde bu davranışı özelleştirebilirsiniz.

## <a name="models-and-providers"></a>Modeller ve sağlayıcılar

ASP.NET Core MVC uygulama modeli, hem soyut arabirimleri hem de bir MVC uygulamasını tanımlayan somut uygulama sınıflarını içerir. Bu model, uygulamanın denetleyicilerini, eylemlerini, eylem parametrelerini, yolları ve filtreleri varsayılan kurallara göre bulma hakkında MVC 'nin sonucudur. Uygulama modeliyle çalışarak, uygulamanızı varsayılan MVC davranışından farklı kurallara göre izlemek için değiştirebilirsiniz. Parametrelerin, adların, yolların ve filtrelerin hepsi, Eylemler ve denetleyiciler için yapılandırma verileri olarak kullanılır.

ASP.NET Core MVC uygulama modeli aşağıdaki yapıya sahiptir:

* ApplicationModel
  * Denetleyiciler (ControllerModel)
    * Eylemler (ActionModel)
      * Parametreler (ParameterModel)

Modelin her düzeyinin ortak bir `Properties` koleksiyonuna erişimi vardır ve alt düzeyler hiyerarşide daha yüksek düzeyler tarafından ayarlanan özellik değerlerine erişebilir ve üzerine yazabilir. Eylemler oluşturulduğunda Özellikler `ActionDescriptor.Properties` kalıcı hale getirilir. Bir istek işlenirken, bir kurala eklenen veya değiştirilen tüm özelliklere `ActionContext.ActionDescriptor.Properties`aracılığıyla erişilebilir. Özellikleri kullanmak, filtrelerinizi, model ciltlerinizi, vb. her eylem temelinde yapılandırmanın harika bir yoludur.

> [!NOTE]
> Uygulama başlatma işlemi tamamlandıktan sonra `ActionDescriptor.Properties` koleksiyonu iş parçacığı güvenli değildir (yazma işlemleri için). Kurallar, bu koleksiyona güvenle veri eklemenin en iyi yoludur.

### <a name="iapplicationmodelprovider"></a>Iapplicationmodelprovider

ASP.NET Core MVC, [ıapplicationmodelprovider](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelprovider) arabirimi tarafından tanımlanan bir sağlayıcı modeli kullanarak uygulama modelini yükler. Bu bölümde, bu sağlayıcının nasıl çalıştığı hakkında bazı iç uygulama ayrıntıları ele alınmaktadır. Bu gelişmiş bir konudur. uygulama modelinden yararlanan birçok uygulama, kurallara göre çalışır.

Her uygulamayla, `Order` özelliğine göre artan sırada `OnProvidersExecuting` arayan `IApplicationModelProvider` arabiriminin "Wrap" uygulamaları diğeri. `OnProvidersExecuted` yöntemi daha sonra ters sırada çağrılır. Framework çeşitli sağlayıcıları tanımlar:

İlk (`Order=-1000`):

* [`DefaultApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.defaultapplicationmodelprovider)

Sonra (`Order=-990`):

* [`AuthorizationApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.authorizationapplicationmodelprovider)
* [`CorsApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.cors.internal.corsapplicationmodelprovider)

> [!NOTE]
> `Order` için aynı değere sahip iki sağlayıcının çağrılmasıyla ilgili sıralama tanımsız ve bu nedenle güvenmemelidir.

> [!NOTE]
> `IApplicationModelProvider`, çerçeve yazarlarının genişlemesine yönelik gelişmiş bir kavramdır. Genel olarak, uygulamalar, sağlayıcıları kullanması gereken kuralları ve çerçeveleri kullanmalıdır. Anahtar ayrımı, sağlayıcıların kuralların önüne her zaman çalıştırılacağı bir değer.

`DefaultApplicationModelProvider`, ASP.NET Core MVC tarafından kullanılan varsayılan davranışların çoğunu belirler. Sorumlulukları şunları içerir:

* Bağlama genel filtreler ekleme
* Bağlama denetleyicilere ekleniyor
* Genel denetleyici yöntemlerini eylem olarak ekleme
* Bağlama eylem yöntemi parametreleri ekleme
* Yol ve diğer öznitelikler uygulanıyor

Bazı yerleşik davranışlar `DefaultApplicationModelProvider`uygulanır. Bu sağlayıcı, [`ActionModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.actionmodel), [`PropertyModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.propertymodel)ve [`ParameterModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.parametermodel) örneklerine başvuran [`ControllerModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.controllermodel)oluşturmaktan sorumludur. `DefaultApplicationModelProvider` sınıfı, gelecekte değiştirecek bir iç çerçeve uygulama ayrıntısıyla sonuçlanır. 

`AuthorizationApplicationModelProvider`, `AuthorizeFilter` ve `AllowAnonymousFilter` öznitelikleriyle ilişkili davranışı uygulamaktan sorumludur. [Bu öznitelikler hakkında daha fazla bilgi edinin](xref:security/authorization/simple).

`CorsApplicationModelProvider`, `IEnableCorsAttribute` ve `IDisableCorsAttribute`ve `DisableCorsAuthorizationFilter`ilişkili davranışı uygular. [CORS hakkında daha fazla bilgi edinin](xref:security/cors).

## <a name="conventions"></a>Kurallar

Uygulama modeli, modellerin tüm model veya sağlayıcıyı geçersiz kılmasından daha basit bir yol sağlayan kural soyutlamalarını tanımlar. Bu soyutlamalar, uygulamanızın davranışını değiştirmek için önerilen yoldur. Kurallar, özelleştirmeleri dinamik olarak uygulayacak kodu yazmanız için bir yol sağlar. [Filtreler](xref:mvc/controllers/filters) çerçeve davranışının değiştirilmesini sağlayan bir yol sağlarken, özelleştirmeler uygulamanın tamamının birlikte nasıl çalıştığını denetlemenizi sağlar.

Aşağıdaki kurallar kullanılabilir:

* [`IApplicationModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelconvention)
* [`IControllerModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.icontrollermodelconvention)
* [`IActionModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iactionmodelconvention)
* [`IParameterModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iparametermodelconvention)

Kurallar, MVC seçeneklerine eklenerek veya `Attribute`s uygulayarak ve bunları denetleyicilere, eylemlere veya eylem parametrelerine uygulayarak ( [`Filters`](xref:mvc/controllers/filters)benzer şekilde) uygulanır. Filtrelerin aksine, kurallar yalnızca uygulama başlatıldığında yürütülür, her isteğin bir parçası olarak değildir.

### <a name="sample-modifying-the-applicationmodel"></a>Örnek: ApplicationModel değiştirme

Aşağıdaki kural uygulama modeline bir özellik eklemek için kullanılır. 

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ApplicationDescription.cs)]

`Startup``ConfigureServices` MVC eklendiğinde, uygulama modeli kuralları seçenek olarak uygulanır.

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=5)]

Özellikler, denetleyici eylemleri içindeki `ActionDescriptor` özellikleri koleksiyonundan erişilebilir:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/AppModelController.cs?name=AppModelController)]

### <a name="sample-modifying-the-controllermodel-description"></a>Örnek: ControllerModel açıklamasını değiştirme

Önceki örnekte olduğu gibi, denetleyici modeli de özel özellikler içerecek şekilde değiştirilebilir. Bunlar, varolan özellikleri uygulama modelinde belirtilen adla geçersiz kılar. Aşağıdaki kural özniteliği, denetleyici düzeyine bir açıklama ekler:

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ControllerDescriptionAttribute.cs)]

Bu kural, bir denetleyiciye bir öznitelik olarak uygulanır.

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=ControllerDescription&highlight=1)]

"Description" özelliğine, önceki örneklerde olduğu gibi erişilir.

### <a name="sample-modifying-the-actionmodel-description"></a>Örnek: ActionModel açıklamasını değiştirme

Ayrı bir öznitelik kuralı tek tek eylemlere uygulanabilir, uygulama veya denetleyici düzeyinde zaten uygulanmış davranışı geçersiz kılar.

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ActionDescriptionAttribute.cs)]

Bu, önceki örnekte denetleyicinin içindeki bir eyleme uygulandığında, denetleyici düzeyi kuralı nasıl geçersiz kılacağını gösterir:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=DescriptionAttributesController&highlight=9)]

### <a name="sample-modifying-the-parametermodel"></a>Örnek: ParameterModel değiştirme

Aşağıdaki kural, `BindingInfo`değiştirmek için eylem parametrelerine uygulanabilir. Aşağıdaki kural parametrenin bir yol parametresi olmasını gerektirir; diğer olası bağlama kaynakları (sorgu dizesi değerleri gibi) yok sayılır.

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/MustBeInRouteParameterModelConvention.cs)]

Öznitelik herhangi bir eylem parametresine uygulanabilir:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/ParameterModelController.cs?name=ParameterModelController&highlight=5)]

### <a name="sample-modifying-the-actionmodel-name"></a>Örnek: ActionModel adını değiştirme

Aşağıdaki kural, uygulandığı eylemin *adını* güncelleştirmek için `ActionModel` değiştirir. Yeni ad, özniteliğe bir parametre olarak sağlanır. Bu yeni ad yönlendirme tarafından kullanılır, bu nedenle bu eylem yöntemine ulaşmak için kullanılan yolu etkileyecektir.

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/CustomActionNameAttribute.cs)]

Bu öznitelik `HomeController`bir eylem yöntemine uygulanır:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/HomeController.cs?name=ActionModelConvention&highlight=2)]

Yöntem adı `SomeName`olsa da öznitelik, yöntem adını kullanmanın MVC kuralını geçersiz kılar ve eylem adını `MyCoolAction`olarak değiştirir. Bu nedenle, bu eyleme ulaşmak için kullanılan yol `/Home/MyCoolAction`.

> [!NOTE]
> Bu örnek temelde, yerleşik [ActionName](/dotnet/api/microsoft.aspnetcore.mvc.actionnameattribute) özniteliği kullanılarak aynıdır.

### <a name="sample-custom-routing-convention"></a>Örnek: özel yönlendirme kuralı

Yönlendirmenin nasıl çalıştığını özelleştirmek için bir `IApplicationModelConvention` kullanabilirsiniz. Örneğin, aşağıdaki kural, denetleyiciler ' ad alanlarını rotalarıyla birleştirir ve ad alanındaki `.`, rotadaki `/` olacak şekilde değiştirir:

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/NamespaceRoutingConvention.cs)]

Kural başlangıçta bir seçenek olarak eklenir.

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=6)]

> [!TIP]
> `services.Configure<MvcOptions>(c => c.Conventions.Add(YOURCONVENTION));` kullanarak `MvcOptions` erişerek, [Ara yazılıma](xref:fundamentals/middleware/index) kurallar ekleyebilirsiniz

Bu örnek, denetleyicinin adında "namespace" özelliği bulunan öznitelik yönlendirme kullanmayan yollara bu kuralı uygular. Aşağıdaki denetleyicide bu kural gösterilmektedir:

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/NamespaceRoutingController.cs?highlight=7-8)]

## <a name="application-model-usage-in-webapicompatshim"></a>WebApiCompatShim 'de uygulama modeli kullanımı

ASP.NET Core MVC, ASP.NET Web API 2 ' den farklı bir kural kümesi kullanır. Özel kuralları kullanarak, bir ASP.NET Core MVC uygulamasının davranışını bir Web API uygulaması ile tutarlı olacak şekilde değiştirebilirsiniz. Microsoft bu amaçla [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim/) 'i özel olarak sevk eder.

> [!NOTE]
> [ASP.NET Web API 'sinden geçiş](xref:migration/webapi)hakkında daha fazla bilgi edinin.

Web API 'SI uyumluluk dolgusu 'nı kullanmak için, paketi projenize eklemeniz ve sonra `Startup``AddWebApiConventions` çağırarak kuralları MVC 'ye eklemeniz gerekir:

```csharp
services.AddMvc().AddWebApiConventions();
```

Dolgu tarafından belirtilen kurallar yalnızca belirli özniteliklerin uygulanmış olduğu uygulamanın bölümlerine uygulanır. Aşağıdaki dört öznitelik, hangi denetleyicilerin kendi kurallarını Shim 'in kuralları tarafından değiştirildiğini denetlemek için kullanılır:

* [UseWebApiActionConventionsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiactionconventionsattribute)
* [UseWebApiOverloadingAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapioverloadingattribute)
* [UseWebApiParameterConventionsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiparameterconventionsattribute)
* [UseWebApiRoutesAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiroutesattribute)

### <a name="action-conventions"></a>Eylem kuralları

`UseWebApiActionConventionsAttribute`, HTTP yöntemini adına göre eylemlerle eşlemek için kullanılır (örneğin, `Get` `HttpGet`eşlenir). Yalnızca öznitelik yönlendirme kullanmayan eylemler için geçerlidir.

### <a name="overloading"></a>Aşırı Yükleme

`UseWebApiOverloadingAttribute`, `WebApiOverloadingApplicationModelConvention` kuralını uygulamak için kullanılır. Bu kural, aday eylemlerini isteğin tüm isteğe bağlı olmayan parametreleri karşılayan olanlarla sınırlayan eylem seçimi işlemine bir `OverloadActionConstraint` ekler.

### <a name="parameter-conventions"></a>Parametre kuralları

`UseWebApiParameterConventionsAttribute`, `WebApiParameterConventionsApplicationModelConvention` eylem kuralını uygulamak için kullanılır. Bu kural, eylem parametreleri olarak kullanılan basit türlerin varsayılan olarak URI 'den bağlandığı, karmaşık türlerin istek gövdesinden bağlandığı bir şekilde belirtir.

### <a name="routes"></a>Rotalar

`UseWebApiRoutesAttribute`, `WebApiApplicationModelConvention` denetleyicisi kuralının uygulanıp uygulanmadığını denetler. Bu kural etkinleştirildiğinde, rotadaki [alanlara](xref:mvc/controllers/areas) yönelik destek eklemek için kullanılır.

Bir kural kümesine ek olarak, uyumluluk paketi, Web API 'SI tarafından sağlanarak yerine geçen bir `System.Web.Http.ApiController` taban sınıfı içerir. Bu, denetleyicilerinizin Web API 'SI için yazılmasını ve `ApiController` ASP.NET Core MVC üzerinde çalıştırılırken tasarlandıkları şekilde çalışmasını sağlar. Daha önce listelenen tüm `UseWebApi*` öznitelikleri, temel denetleyici sınıfına uygulanır. `ApiController`, Web API 'sinde bulunan özelliklerle uyumlu özellikleri, yöntemleri ve sonuç türlerini kullanıma sunar.

## <a name="using-apiexplorer-to-document-your-app"></a>Uygulamanızı belgelemek için ApiExplorer kullanma

Uygulama modeli, uygulamanın yapısına çapraz geçiş için kullanılabilecek her düzeyde bir [`ApiExplorer`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.apiexplorermodel) özelliği sunar. Bu, [Swagger gibi araçları kullanarak Web API 'leriniz için yardım sayfaları oluşturmak](xref:tutorials/web-api-help-pages-using-swagger)üzere kullanılabilir. `ApiExplorer` özelliği, uygulamanızın modelinin hangi bölümlerinin sunulduğunu belirtmek üzere ayarlanabilir bir `IsVisible` özelliğini kullanıma sunar. Bu ayarı, bir kuralı kullanarak yapılandırabilirsiniz:

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/EnableApiExplorerApplicationConvention.cs)]

Bu yaklaşımı (ve gerekirse ek kuralları) kullanarak uygulamanızın içindeki herhangi bir düzeyde API görünürlüğünü etkinleştirebilir veya devre dışı bırakabilirsiniz. 
