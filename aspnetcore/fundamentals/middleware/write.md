---
title: Özel ASP.NET Core ara yazılımı yaz
author: rick-anderson
description: Özel ASP.NET Core ara yazılımı yazmayı öğrenin.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/22/2019
uid: fundamentals/middleware/write
ms.openlocfilehash: e74bba9e1bd826d4f493b0ee642a198f984daada
ms.sourcegitcommit: f65d8765e4b7c894481db9b37aa6969abc625a48
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 09/06/2019
ms.locfileid: "70773725"
---
# <a name="write-custom-aspnet-core-middleware"></a>Özel ASP.NET Core ara yazılımı yaz

By [Rick Anderson](https://twitter.com/RickAndMSFT) ve [Steve Smith](https://ardalis.com/)

Ara yazılım, istekleri ve yanıtları işlemek için bir uygulama ardışık düzenine çevrilmiş yazılımdır. ASP.NET Core, zengin bir yerleşik ara yazılım bileşenleri kümesi sağlar, ancak bazı senaryolarda özel bir ara yazılım yazmak isteyebilirsiniz.

## <a name="middleware-class"></a>Ara yazılım sınıfı

Ara yazılım genellikle bir sınıfta kapsüllenir ve bir genişletme yöntemiyle birlikte sunulur. Bir sorgu dizesinden geçerli istek için kültürü ayarlayan aşağıdaki ara yazılımı göz önünde bulundurun:

[!code-csharp[](write/snapshot/StartupCulture.cs)]

Yukarıdaki örnek kod, bir ara yazılım bileşeni oluşturmayı göstermek için kullanılır. ASP.NET Core yerleşik yerelleştirme desteği için bkz <xref:fundamentals/localization>.

Kültürü geçirerek, ara yazılımı test edin. Örneğin, istek `https://localhost:5001/?culture=no`.

Aşağıdaki kod, ara yazılım temsilcisini bir sınıfa taşıtabilirler:

[!code-csharp[](write/snapshot/RequestCultureMiddleware.cs)]

Ara yazılım sınıfı şunları içermelidir:

* Türünde <xref:Microsoft.AspNetCore.Http.RequestDelegate>bir parametreye sahip ortak Oluşturucu.
* `Invoke` Veya`InvokeAsync`adlı bir genel yöntem. Bu yöntem şunları içermelidir:
  * Bir `Task`döndürür.
  * Türünde <xref:Microsoft.AspNetCore.Http.HttpContext>bir ilk parametreyi kabul edin.
  
Oluşturucuya yönelik ek `Invoke` parametreler ve / `InvokeAsync` [bağımlılık ekleme (dı)](xref:fundamentals/dependency-injection)tarafından doldurulur.

## <a name="middleware-dependencies"></a>Ara yazılım bağımlılıkları

Ara yazılım, bağımlılıklarındaki bağımlılıklarını açığa çıkararak [Açık bağımlılıklar ilkesini](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies) izlemelidir. Ara yazılım, *uygulama ömrü*başına bir kez oluşturulur. Bir istek içindeki ara yazılım ile hizmetleri paylaşmanız gerekiyorsa, [Istek başına ara yazılım bağımlılıkları](#per-request-middleware-dependencies) bölümüne bakın.

Ara yazılım bileşenleri, [bağımlılık ekleme (dı)](xref:fundamentals/dependency-injection) öğesinden Oluşturucu parametreleri aracılığıyla bağımlılıklarını çözümleyebilir. [Useara&lt;yazılım&gt; T](/dotnet/api/microsoft.aspnetcore.builder.usemiddlewareextensions.usemiddleware#Microsoft_AspNetCore_Builder_UseMiddlewareExtensions_UseMiddleware_Microsoft_AspNetCore_Builder_IApplicationBuilder_System_Type_System_Object___) ayrıca ek parametreleri doğrudan kabul edebilir.

## <a name="per-request-middleware-dependencies"></a>İstek başına ara yazılım bağımlılıkları

Ara yazılım uygulama başlangıcında oluşturulduğundan, isteğe göre değil, ara yazılım oluşturucuları tarafından kullanılan *kapsamlı* ömür Hizmetleri, her istek sırasında bağımlılığı eklenen diğer türlerle paylaşılmaz. Bir *kapsamlı* hizmeti, ara yazılım ve diğer türler arasında paylaşmanız gerekiyorsa, bu Hizmetleri `Invoke` metodun imzasına ekleyin. Yöntemi `Invoke` , dı tarafından doldurulan ek parametreleri kabul edebilir:

```csharp
public class CustomMiddleware
{
    private readonly RequestDelegate _next;

    public CustomMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    // IMyScopedService is injected into Invoke
    public async Task Invoke(HttpContext httpContext, IMyScopedService svc)
    {
        svc.MyProperty = 1000;
        await _next(httpContext);
    }
}
```

## <a name="middleware-extension-method"></a>Ara yazılım genişletme yöntemi

Aşağıdaki genişletme yöntemi, ara yazılımı aracılığıyla <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder>kullanıma sunar:

[!code-csharp[](write/snapshot/RequestCultureMiddlewareExtensions.cs)]

Aşağıdaki kod, içindeki `Startup.Configure`ara yazılımı çağırır:

[!code-csharp[](write/snapshot/Startup.cs?highlight=5)]

## <a name="additional-resources"></a>Ek kaynaklar

* <xref:fundamentals/middleware/index>
* <xref:migration/http-modules>
* <xref:fundamentals/startup>
* <xref:fundamentals/request-features>
* <xref:fundamentals/middleware/extensibility>
* <xref:fundamentals/middleware/extensibility-third-party-container>
