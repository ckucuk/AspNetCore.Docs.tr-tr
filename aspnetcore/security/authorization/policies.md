---
title: ASP.NET Core ilke tabanlı yetkilendirme
author: rick-anderson
description: ASP.NET Core uygulamasında yetkilendirme gereksinimlerini zorlama için yetkilendirme ilkesi işleyicileri oluşturma ve kullanma hakkında bilgi edinin.
ms.author: riande
ms.custom: mvc
ms.date: 10/05/2019
uid: security/authorization/policies
ms.openlocfilehash: eeb5ddd63ef8177325b35e5a666aa5e9ab047057
ms.sourcegitcommit: 7dfe6cc8408ac6a4549c29ca57b0c67ec4baa8de
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 01/09/2020
ms.locfileid: "75828964"
---
# <a name="policy-based-authorization-in-aspnet-core"></a>ASP.NET Core ilke tabanlı yetkilendirme

::: moniker range=">= aspnetcore-3.0"

Kapakların altında [rol tabanlı yetkilendirme](xref:security/authorization/roles) ve [talep tabanlı yetkilendirme](xref:security/authorization/claims) , bir gereksinim, gereksinim işleyicisi ve önceden yapılandırılmış bir ilke kullanır. Bu yapı taşları, koddaki yetkilendirme değerlendirmeleri ifadesini destekler. Sonuç daha zengin, yeniden kullanılabilir ve test edilebilir bir yetkilendirme yapısıdır.

Yetkilendirme ilkesi bir veya daha fazla gereksinimden oluşur. `Startup.ConfigureServices` yönteminde yetkilendirme hizmeti yapılandırmasının bir parçası olarak kaydedilir:

[!code-csharp[](policies/samples/3.0PoliciesAuthApp1/Startup.cs?range=31-32,39-40,42-45, 53, 58)]

Yukarıdaki örnekte, "AtLeast21" ilkesi oluşturulur. Gereksinime bir parametre olarak sağlanan minimum yaş&mdash;tek bir gereksinime sahiptir.

## <a name="iauthorizationservice"></a>IAuthorizationService 

Yetkilendirmenin başarılı olup olmadığını belirleyen birincil hizmet <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService>:

[!code-csharp[](policies/samples/stubs/copy_of_IAuthorizationService.cs?highlight=24-25,48-49&name=snippet)]

Yukarıdaki kod, [IAuthorizationService](https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationService.cs)'in iki yöntemini vurgular.

<xref:Microsoft.AspNetCore.Authorization.IAuthorizationRequirement>, yöntemi olmayan bir işaret hizmetidir ve yetkilendirme işleminin başarılı olup olmadığını izlemeye yönelik mekanizmaya yöneliktir.

Her bir <xref:Microsoft.AspNetCore.Authorization.IAuthorizationHandler> gereksinimlerin karşılanıp karşılanmadığını denetmaktan sorumludur:
<!--The following code is a copy/paste from 
https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationHandler.cs -->

```csharp
/// <summary>
/// Classes implementing this interface are able to make a decision if authorization
/// is allowed.
/// </summary>
public interface IAuthorizationHandler
{
    /// <summary>
    /// Makes a decision if authorization is allowed.
    /// </summary>
    /// <param name="context">The authorization information.</param>
    Task HandleAsync(AuthorizationHandlerContext context);
}
```

<xref:Microsoft.AspNetCore.Authorization.AuthorizationHandlerContext> sınıfı, işleyicinin gereksinimlerin karşılanıp karşılanmadığını işaretlemek için kullandığı şeydir:

```csharp
 context.Succeed(requirement)
```

Aşağıdaki kod, yetkilendirme hizmetinin Basitleştirilmiş (ve açıklamalar ile açıklamalı) varsayılan uygulamasını gösterir:

```csharp
public async Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user, 
             object resource, IEnumerable<IAuthorizationRequirement> requirements)
{
    // Create a tracking context from the authorization inputs.
    var authContext = _contextFactory.CreateContext(requirements, user, resource);

    // By default this returns an IEnumerable<IAuthorizationHandlers> from DI.
    var handlers = await _handlers.GetHandlersAsync(authContext);

    // Invoke all handlers.
    foreach (var handler in handlers)
    {
        await handler.HandleAsync(authContext);
    }

    // Check the context, by default success is when all requirements have been met.
    return _evaluator.Evaluate(authContext);
}
```

Aşağıdaki kod tipik bir `ConfigureServices`gösterir:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add all of your handlers to DI.
    services.AddSingleton<IAuthorizationHandler, MyHandler1>();
    // MyHandler2, ...

    services.AddSingleton<IAuthorizationHandler, MyHandlerN>();

    // Configure your policies
    services.AddAuthorization(options =>
          options.AddPolicy("Something",
          policy => policy.RequireClaim("Permission", "CanViewPage", "CanViewAnything")));


    services.AddControllersWithViews();
    services.AddRazorPages();
}
```

Yetkilendirme için <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService> veya `[Authorize(Policy = "Something")]` kullanın.

## <a name="applying-policies-to-mvc-controllers"></a>MVC denetleyicilerine ilke uygulama

Razor Pages kullanıyorsanız, bkz. bu belgedeki [Razor Pages Ilkeleri uygulama](#applying-policies-to-razor-pages) .

İlkeler, ilke adı ile `[Authorize]` özniteliği kullanılarak denetleyicilere uygulanır. Örneğin:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Controllers/AlcoholPurchaseController.cs?name=snippet_AlcoholPurchaseControllerClass&highlight=4)]

## <a name="applying-policies-to-razor-pages"></a>Razor Pages ilke uygulanıyor

İlkeler, ilke adı ile `[Authorize]` özniteliği kullanılarak Razor Pages uygulanır. Örneğin:

[!code-csharp[](policies/samples/PoliciesAuthApp2/Pages/AlcoholPurchase.cshtml.cs?name=snippet_AlcoholPurchaseModelClass&highlight=4)]

İlkeler, [Yetkilendirme kuralı](xref:security/authorization/razor-pages-authorization)kullanılarak Razor Pages da uygulanabilir.

## <a name="requirements"></a>Gereksinimler

Yetkilendirme gereksinimi, bir ilkenin geçerli kullanıcı sorumlusunu değerlendirmek için kullanabileceği veri parametreleri koleksiyonudur. "AtLeast21" ilkenizde, gereksinim minimum yaş&mdash;tek bir parametredir. Bir gereksinim, boş bir işaret arabirimi olan [ıauthorizationrequirement](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement)'ı uygular. Parametreli en düşük yaş gereksinimi aşağıdaki gibi uygulanabilir:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/MinimumAgeRequirement.cs?name=snippet_MinimumAgeRequirementClass)]

Yetkilendirme ilkesi birden çok yetkilendirme gereksinimi içeriyorsa, ilke değerlendirmesinin başarılı olması için tüm gereksinimlerin geçmesi gerekir. Diğer bir deyişle, tek bir yetkilendirme ilkesine eklenen birden çok yetkilendirme gereksinimi bir **ve** temelinde değerlendirilir.

> [!NOTE]
> Bir gereksinimin veri veya özellikleri olması gerekmez.

<a name="security-authorization-policies-based-authorization-handler"></a>

## <a name="authorization-handlers"></a>Yetkilendirme işleyicileri

Bir yetkilendirme işleyicisi, bir gereksinimin özelliklerinin değerlendirilmesinden sorumludur. Yetkilendirme işleyicisi, erişim izni verilip verilmediğini belirlemede, gereksinimleri belirtilen [Authorizationhandlercontext](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext) 'e göre değerlendirir.

Bir gereksinimin [birden çok işleyicisi](#security-authorization-policies-based-multiple-handlers)olabilir. Bir işleyici, `TRequirement` işlenmesi gereken [\<TRequirement > authorizationhandler](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1)'ı devralınabilir. Alternatif olarak, bir işleyici birden fazla gereksinim türünü işlemek için [ıauthorizationhandler](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationhandler) uygulayabilir.

### <a name="use-a-handler-for-one-requirement"></a>Bir gereksinim için bir işleyici kullanın

<a name="security-authorization-handler-example"></a>

Aşağıda, en az bir yaş işleyicisinin tek bir gereksinimin kullanıldığı bire bir ilişkiye örnek verilmiştir:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/MinimumAgeHandler.cs?name=snippet_MinimumAgeHandlerClass)]

Önceki kod, geçerli kullanıcı sorumlusunun bilinen ve güvenilir bir veren tarafından verilen bir Doğum talebi tarihi olup olmadığını belirler. Talep eksik olduğunda yetkilendirme gerçekleşemez, bu durumda tamamlanan bir görev döndürülür. Bir talep bulunduğunda, kullanıcının yaşı hesaplanır. Kullanıcı gereksinim tarafından tanımlanan en düşük yaşı karşılıyorsa, Yetkilendirme başarılı olur. Yetkilendirme başarılı olduğunda, `context.Succeed` tek parametresi olarak tatmin eden gereksinimle çağrılır.

### <a name="use-a-handler-for-multiple-requirements"></a>Birden çok gereksinim için bir işleyici kullanın

Aşağıda, bir izin işleyicisinin üç farklı gereksinim türünü işleyebileceği bir çoktan çoğa ilişkiye örnek verilmiştir:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/PermissionHandler.cs?name=snippet_PermissionHandlerClass)]

Yukarıdaki kod, başarılı olarak işaretlenmemiş gereksinimleri içeren bir özellik&mdash;[Pendingrequirements](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.pendingrequirements#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_PendingRequirements) 'e geçer. `ReadPermission` gereksinim için, kullanıcı istenen kaynağa erişmek için bir sahip veya sponsor olmalıdır. Bir `EditPermission` veya `DeletePermission` gereksinimi söz konusu olduğunda, istenen kaynağa erişmek için sahip olması gerekir.

<a name="security-authorization-policies-based-handler-registration"></a>

### <a name="handler-registration"></a>İşleyici kaydı

İşleyiciler, yapılandırma sırasında hizmetler koleksiyonuna kaydedilir. Örneğin:

[!code-csharp[](policies/samples/3.0PoliciesAuthApp1/Startup.cs?range=31-32,39-40,42-45, 53-55, 58)]

Yukarıdaki kod, `services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();`çağırarak tek bir `MinimumAgeHandler` olarak kaydedilir. İşleyiciler, yerleşik [hizmet yaşam sürelerinin](xref:fundamentals/dependency-injection#service-lifetimes)herhangi biri kullanılarak kaydedilebilir.

## <a name="what-should-a-handler-return"></a>İşleyici ne döndürmelidir?

[İşleyici örnekteki](#security-authorization-handler-example) `Handle` yönteminin hiçbir değer döndürmediğini unutmayın. Başarı ya da hatanın durumu nasıl belirtilir?

* İşleyici, başarıyla doğrulanan gereksinimi geçirerek `context.Succeed(IAuthorizationRequirement requirement)`çağırarak başarıyı gösterir.

* Aynı gereksinim için diğer işleyiciler başarılı olabileceğinden, işleyicinin sorunları genellikle işlemesi gerekmez.

* Diğer gereksinim işleyicileri başarılı olsa bile hatayı güvence altına almak için `context.Fail`çağırın.

Bir işleyici `context.Succeed` veya `context.Fail`çağırırsa, diğer tüm işleyiciler yine de çağırılır. Bu, başka bir işleyicinin bir gereksinimi başarıyla doğrulayan veya başarısız olsa bile, gereksinimlerin günlüğe kaydetme gibi yan etkileri üretmesine olanak tanır. `false`olarak ayarlandığında, [ınvokehandlersafterfailure](/dotnet/api/microsoft.aspnetcore.authorization.authorizationoptions.invokehandlersafterfailure#Microsoft_AspNetCore_Authorization_AuthorizationOptions_InvokeHandlersAfterFailure) özelliği (ASP.NET Core 1,1 ve üzeri sürümlerde bulunur) `context.Fail` çağrıldığında işleyicilerin yürütülmesi kısa devre dışı olur. `true`varsayılan olarak `InvokeHandlersAfterFailure`, bu durumda tüm işleyiciler çağrılır.

> [!NOTE]
> Yetkilendirme işleyicileri, kimlik doğrulama başarısız olsa bile çağrılır.

<a name="security-authorization-policies-based-multiple-handlers"></a>

## <a name="why-would-i-want-multiple-handlers-for-a-requirement"></a>Bir gereksinim için neden birden çok işleyici istiyorum?

Değerlendirmenin bir **veya** temelinde olmasını istediğiniz durumlarda, tek bir gereksinim için birden çok işleyici uygulayın. Örneğin, Microsoft yalnızca ana kartlarla açık olan kapılara sahiptir. Ana kartınızı evden bırakırsanız, alıcı geçici bir etiket yazdırır ve kapıyı sizin için açar. Bu senaryoda, tek bir gereksinimi incelerken tek bir gereksinimin, *Buildingentry*, ancak birden çok işleyici vardır.

*BuildingEntryRequirement.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/BuildingEntryRequirement.cs?name=snippet_BuildingEntryRequirementClass)]

*BadgeEntryHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/BadgeEntryHandler.cs?name=snippet_BadgeEntryHandlerClass)]

*TemporaryStickerHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/TemporaryStickerHandler.cs?name=snippet_TemporaryStickerHandlerClass)]

Her iki işleyicinin de [kaydedildiğinden](xref:security/authorization/policies#security-authorization-policies-based-handler-registration)emin olun. Bir ilke `BuildingEntryRequirement`değerlendirdiğinde işleyicinin her ikisi de başarılı olursa, ilke değerlendirmesi başarılı olur.

## <a name="using-a-func-to-fulfill-a-policy"></a>İlkeyi yerine getirmek için bir Func kullanma

Kodun kodda hızlı bir şekilde kullanılması için bir ilkeyi karşıladığı durumlar olabilir. İlkenizi `RequireAssertion` ilkesi Oluşturucusu ile yapılandırırken bir `Func<AuthorizationHandlerContext, bool>` sağlamak mümkündür.

Örneğin, önceki `BadgeEntryHandler` şu şekilde yeniden yazılabilir:

[!code-csharp[](policies/samples/3.0PoliciesAuthApp1/Startup.cs?range=42-43,47-53)]

## <a name="accessing-mvc-request-context-in-handlers"></a>İşleyicilerde MVC istek bağlamına erişme

Bir yetkilendirme işleyicisinde uyguladığınız `HandleRequirementAsync` yöntemi iki parametreye sahiptir: bir `AuthorizationHandlerContext` ve işlemekte olduğunuz `TRequirement`. MVC veya Jabbr gibi çerçeveler, ek bilgi iletmek için `AuthorizationHandlerContext` `Resource` özelliğine herhangi bir nesne eklemek ücretsizdir.

Örneğin, MVC `Resource` özelliğinde [Authorizationfiltercontext](/dotnet/api/?term=AuthorizationFilterContext) örneğini geçirir. Bu özellik `HttpContext`, `RouteData`ve MVC ve Razor Pages tarafından sağlanan diğer her şeye erişim sağlar.

`Resource` özelliğinin kullanımı Framework 'e özgüdür. `Resource` özelliğindeki bilgilerin kullanılması, yetkilendirme ilkelerinizi belirli çerçeveler ile sınırlandırır. `is` anahtar sözcüğünü kullanarak `Resource` özelliğini atamalısınız ve sonra, kodunuzun diğer çerçeveler üzerinde çalıştırıldığında bir `InvalidCastException` kilitlenmediğinden emin olmak için dönüştürme başarılı olduğunu doğrulayın:

```csharp
// Requires the following import:
//     using Microsoft.AspNetCore.Mvc.Filters;
if (context.Resource is AuthorizationFilterContext mvcContext)
{
    // Examine MVC-specific things like routing data.
}
```

::: moniker-end


::: moniker range="< aspnetcore-3.0"

Kapakların altında [rol tabanlı yetkilendirme](xref:security/authorization/roles) ve [talep tabanlı yetkilendirme](xref:security/authorization/claims) , bir gereksinim, gereksinim işleyicisi ve önceden yapılandırılmış bir ilke kullanır. Bu yapı taşları, koddaki yetkilendirme değerlendirmeleri ifadesini destekler. Sonuç daha zengin, yeniden kullanılabilir ve test edilebilir bir yetkilendirme yapısıdır.

Yetkilendirme ilkesi bir veya daha fazla gereksinimden oluşur. `Startup.ConfigureServices` yönteminde yetkilendirme hizmeti yapılandırmasının bir parçası olarak kaydedilir:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=32-33,48-53,61,66)]

Yukarıdaki örnekte, "AtLeast21" ilkesi oluşturulur. Gereksinime bir parametre olarak sağlanan minimum yaş&mdash;tek bir gereksinime sahiptir.

## <a name="iauthorizationservice"></a>IAuthorizationService 

Yetkilendirmenin başarılı olup olmadığını belirleyen birincil hizmet <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService>:

[!code-csharp[](policies/samples/stubs/copy_of_IAuthorizationService.cs?highlight=24-25,48-49&name=snippet)]

Yukarıdaki kod, [IAuthorizationService](https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationService.cs)'in iki yöntemini vurgular.

<xref:Microsoft.AspNetCore.Authorization.IAuthorizationRequirement>, yöntemi olmayan bir işaret hizmetidir ve yetkilendirme işleminin başarılı olup olmadığını izlemeye yönelik mekanizmaya yöneliktir.

Her bir <xref:Microsoft.AspNetCore.Authorization.IAuthorizationHandler> gereksinimlerin karşılanıp karşılanmadığını denetmaktan sorumludur:
<!--The following code is a copy/paste from 
https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationHandler.cs -->

```csharp
/// <summary>
/// Classes implementing this interface are able to make a decision if authorization
/// is allowed.
/// </summary>
public interface IAuthorizationHandler
{
    /// <summary>
    /// Makes a decision if authorization is allowed.
    /// </summary>
    /// <param name="context">The authorization information.</param>
    Task HandleAsync(AuthorizationHandlerContext context);
}
```

<xref:Microsoft.AspNetCore.Authorization.AuthorizationHandlerContext> sınıfı, işleyicinin gereksinimlerin karşılanıp karşılanmadığını işaretlemek için kullandığı şeydir:

```csharp
 context.Succeed(requirement)
```

Aşağıdaki kod, yetkilendirme hizmetinin Basitleştirilmiş (ve açıklamalar ile açıklamalı) varsayılan uygulamasını gösterir:

```csharp
public async Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user, 
             object resource, IEnumerable<IAuthorizationRequirement> requirements)
{
    // Create a tracking context from the authorization inputs.
    var authContext = _contextFactory.CreateContext(requirements, user, resource);

    // By default this returns an IEnumerable<IAuthorizationHandlers> from DI.
    var handlers = await _handlers.GetHandlersAsync(authContext);

    // Invoke all handlers.
    foreach (var handler in handlers)
    {
        await handler.HandleAsync(authContext);
    }

    // Check the context, by default success is when all requirements have been met.
    return _evaluator.Evaluate(authContext);
}
```

Aşağıdaki kod tipik bir `ConfigureServices`gösterir:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add all of your handlers to DI.
    services.AddSingleton<IAuthorizationHandler, MyHandler1>();
    // MyHandler2, ...

    services.AddSingleton<IAuthorizationHandler, MyHandlerN>();

    // Configure your policies
    services.AddAuthorization(options =>
          options.AddPolicy("Something",
          policy => policy.RequireClaim("Permission", "CanViewPage", "CanViewAnything")));


    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
}
```

Yetkilendirme için <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService> veya `[Authorize(Policy = "Something")]` kullanın.

## <a name="applying-policies-to-mvc-controllers"></a>MVC denetleyicilerine ilke uygulama

Razor Pages kullanıyorsanız, bkz. bu belgedeki [Razor Pages Ilkeleri uygulama](#applying-policies-to-razor-pages) .

İlkeler, ilke adı ile `[Authorize]` özniteliği kullanılarak denetleyicilere uygulanır. Örneğin:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Controllers/AlcoholPurchaseController.cs?name=snippet_AlcoholPurchaseControllerClass&highlight=4)]

## <a name="applying-policies-to-razor-pages"></a>Razor Pages ilke uygulanıyor

İlkeler, ilke adı ile `[Authorize]` özniteliği kullanılarak Razor Pages uygulanır. Örneğin:

[!code-csharp[](policies/samples/PoliciesAuthApp2/Pages/AlcoholPurchase.cshtml.cs?name=snippet_AlcoholPurchaseModelClass&highlight=4)]

İlkeler, [Yetkilendirme kuralı](xref:security/authorization/razor-pages-authorization)kullanılarak Razor Pages da uygulanabilir.

## <a name="requirements"></a>Gereksinimler

Yetkilendirme gereksinimi, bir ilkenin geçerli kullanıcı sorumlusunu değerlendirmek için kullanabileceği veri parametreleri koleksiyonudur. "AtLeast21" ilkenizde, gereksinim minimum yaş&mdash;tek bir parametredir. Bir gereksinim, boş bir işaret arabirimi olan [ıauthorizationrequirement](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement)'ı uygular. Parametreli en düşük yaş gereksinimi aşağıdaki gibi uygulanabilir:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/MinimumAgeRequirement.cs?name=snippet_MinimumAgeRequirementClass)]

Yetkilendirme ilkesi birden çok yetkilendirme gereksinimi içeriyorsa, ilke değerlendirmesinin başarılı olması için tüm gereksinimlerin geçmesi gerekir. Diğer bir deyişle, tek bir yetkilendirme ilkesine eklenen birden çok yetkilendirme gereksinimi bir **ve** temelinde değerlendirilir.

> [!NOTE]
> Bir gereksinimin veri veya özellikleri olması gerekmez.

<a name="security-authorization-policies-based-authorization-handler"></a>

## <a name="authorization-handlers"></a>Yetkilendirme işleyicileri

Bir yetkilendirme işleyicisi, bir gereksinimin özelliklerinin değerlendirilmesinden sorumludur. Yetkilendirme işleyicisi, erişim izni verilip verilmediğini belirlemede, gereksinimleri belirtilen [Authorizationhandlercontext](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext) 'e göre değerlendirir.

Bir gereksinimin [birden çok işleyicisi](#security-authorization-policies-based-multiple-handlers)olabilir. Bir işleyici, `TRequirement` işlenmesi gereken [\<TRequirement > authorizationhandler](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1)'ı devralınabilir. Alternatif olarak, bir işleyici birden fazla gereksinim türünü işlemek için [ıauthorizationhandler](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationhandler) uygulayabilir.

### <a name="use-a-handler-for-one-requirement"></a>Bir gereksinim için bir işleyici kullanın

<a name="security-authorization-handler-example"></a>

Aşağıda, en az bir yaş işleyicisinin tek bir gereksinimin kullanıldığı bire bir ilişkiye örnek verilmiştir:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/MinimumAgeHandler.cs?name=snippet_MinimumAgeHandlerClass)]

Önceki kod, geçerli kullanıcı sorumlusunun bilinen ve güvenilir bir veren tarafından verilen bir Doğum talebi tarihi olup olmadığını belirler. Talep eksik olduğunda yetkilendirme gerçekleşemez, bu durumda tamamlanan bir görev döndürülür. Bir talep bulunduğunda, kullanıcının yaşı hesaplanır. Kullanıcı gereksinim tarafından tanımlanan en düşük yaşı karşılıyorsa, Yetkilendirme başarılı olur. Yetkilendirme başarılı olduğunda, `context.Succeed` tek parametresi olarak tatmin eden gereksinimle çağrılır.

### <a name="use-a-handler-for-multiple-requirements"></a>Birden çok gereksinim için bir işleyici kullanın

Aşağıda, bir izin işleyicisinin üç farklı gereksinim türünü işleyebileceği bir çoktan çoğa ilişkiye örnek verilmiştir:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/PermissionHandler.cs?name=snippet_PermissionHandlerClass)]

Yukarıdaki kod, başarılı olarak işaretlenmemiş gereksinimleri içeren bir özellik&mdash;[Pendingrequirements](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.pendingrequirements#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_PendingRequirements) 'e geçer. `ReadPermission` gereksinim için, kullanıcı istenen kaynağa erişmek için bir sahip veya sponsor olmalıdır. Bir `EditPermission` veya `DeletePermission` gereksinimi söz konusu olduğunda, istenen kaynağa erişmek için sahip olması gerekir.

<a name="security-authorization-policies-based-handler-registration"></a>

### <a name="handler-registration"></a>İşleyici kaydı

İşleyiciler, yapılandırma sırasında hizmetler koleksiyonuna kaydedilir. Örneğin:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=32-33,48-53,61,62-63,66)]

Yukarıdaki kod, `services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();`çağırarak tek bir `MinimumAgeHandler` olarak kaydedilir. İşleyiciler, yerleşik [hizmet yaşam sürelerinin](xref:fundamentals/dependency-injection#service-lifetimes)herhangi biri kullanılarak kaydedilebilir.

## <a name="what-should-a-handler-return"></a>İşleyici ne döndürmelidir?

[İşleyici örnekteki](#security-authorization-handler-example) `Handle` yönteminin hiçbir değer döndürmediğini unutmayın. Başarı ya da hatanın durumu nasıl belirtilir?

* İşleyici, başarıyla doğrulanan gereksinimi geçirerek `context.Succeed(IAuthorizationRequirement requirement)`çağırarak başarıyı gösterir.

* Aynı gereksinim için diğer işleyiciler başarılı olabileceğinden, işleyicinin sorunları genellikle işlemesi gerekmez.

* Diğer gereksinim işleyicileri başarılı olsa bile hatayı güvence altına almak için `context.Fail`çağırın.

Bir işleyici `context.Succeed` veya `context.Fail`çağırırsa, diğer tüm işleyiciler yine de çağırılır. Bu, başka bir işleyicinin bir gereksinimi başarıyla doğrulayan veya başarısız olsa bile, gereksinimlerin günlüğe kaydetme gibi yan etkileri üretmesine olanak tanır. `false`olarak ayarlandığında, [ınvokehandlersafterfailure](/dotnet/api/microsoft.aspnetcore.authorization.authorizationoptions.invokehandlersafterfailure#Microsoft_AspNetCore_Authorization_AuthorizationOptions_InvokeHandlersAfterFailure) özelliği (ASP.NET Core 1,1 ve üzeri sürümlerde bulunur) `context.Fail` çağrıldığında işleyicilerin yürütülmesi kısa devre dışı olur. `true`varsayılan olarak `InvokeHandlersAfterFailure`, bu durumda tüm işleyiciler çağrılır.

> [!NOTE]
> Yetkilendirme işleyicileri, kimlik doğrulama başarısız olsa bile çağrılır.

<a name="security-authorization-policies-based-multiple-handlers"></a>

## <a name="why-would-i-want-multiple-handlers-for-a-requirement"></a>Bir gereksinim için neden birden çok işleyici istiyorum?

Değerlendirmenin bir **veya** temelinde olmasını istediğiniz durumlarda, tek bir gereksinim için birden çok işleyici uygulayın. Örneğin, Microsoft yalnızca ana kartlarla açık olan kapılara sahiptir. Ana kartınızı evden bırakırsanız, alıcı geçici bir etiket yazdırır ve kapıyı sizin için açar. Bu senaryoda, tek bir gereksinimi incelerken tek bir gereksinimin, *Buildingentry*, ancak birden çok işleyici vardır.

*BuildingEntryRequirement.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/BuildingEntryRequirement.cs?name=snippet_BuildingEntryRequirementClass)]

*BadgeEntryHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/BadgeEntryHandler.cs?name=snippet_BadgeEntryHandlerClass)]

*TemporaryStickerHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/TemporaryStickerHandler.cs?name=snippet_TemporaryStickerHandlerClass)]

Her iki işleyicinin de [kaydedildiğinden](xref:security/authorization/policies#security-authorization-policies-based-handler-registration)emin olun. Bir ilke `BuildingEntryRequirement`değerlendirdiğinde işleyicinin her ikisi de başarılı olursa, ilke değerlendirmesi başarılı olur.

## <a name="using-a-func-to-fulfill-a-policy"></a>İlkeyi yerine getirmek için bir Func kullanma

Kodun kodda hızlı bir şekilde kullanılması için bir ilkeyi karşıladığı durumlar olabilir. İlkenizi `RequireAssertion` ilkesi Oluşturucusu ile yapılandırırken bir `Func<AuthorizationHandlerContext, bool>` sağlamak mümkündür.

Örneğin, önceki `BadgeEntryHandler` şu şekilde yeniden yazılabilir:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=50-51,55-61)]

## <a name="accessing-mvc-request-context-in-handlers"></a>İşleyicilerde MVC istek bağlamına erişme

Bir yetkilendirme işleyicisinde uyguladığınız `HandleRequirementAsync` yöntemi iki parametreye sahiptir: bir `AuthorizationHandlerContext` ve işlemekte olduğunuz `TRequirement`. MVC veya Jabbr gibi çerçeveler, ek bilgi iletmek için `AuthorizationHandlerContext` `Resource` özelliğine herhangi bir nesne eklemek ücretsizdir.

Örneğin, MVC `Resource` özelliğinde [Authorizationfiltercontext](/dotnet/api/?term=AuthorizationFilterContext) örneğini geçirir. Bu özellik `HttpContext`, `RouteData`ve MVC ve Razor Pages tarafından sağlanan diğer her şeye erişim sağlar.

`Resource` özelliğinin kullanımı Framework 'e özgüdür. `Resource` özelliğindeki bilgilerin kullanılması, yetkilendirme ilkelerinizi belirli çerçeveler ile sınırlandırır. `is` anahtar sözcüğünü kullanarak `Resource` özelliğini atamalısınız ve sonra, kodunuzun diğer çerçeveler üzerinde çalıştırıldığında bir `InvalidCastException` kilitlenmediğinden emin olmak için dönüştürme başarılı olduğunu doğrulayın:

```csharp
// Requires the following import:
//     using Microsoft.AspNetCore.Mvc.Filters;
if (context.Resource is AuthorizationFilterContext mvcContext)
{
    // Examine MVC-specific things like routing data.
}
```

::: moniker-end