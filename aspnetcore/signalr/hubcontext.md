---
title: SignalR HubContext
author: rachelappel
description: Bir hub dışında istemcilere bildirimleri göndermek için ASP.NET Core SignalR HubContext hizmeti kullanmayı öğrenin.
monikerRange: '>= aspnetcore-2.1'
ms.author: rachelap
ms.custom: mvc
ms.date: 06/13/2018
uid: signalr/hubcontext
ms.openlocfilehash: ccfcdc8337275fd26e09c1a43db36cf9ab90cf46
ms.sourcegitcommit: a1afd04758e663d7062a5bfa8a0d4dca38f42afc
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 06/20/2018
ms.locfileid: "36277767"
---
# <a name="send-messages-from-outside-a-hub"></a><span data-ttu-id="057c2-103">Bir hub dışında ileti gönderme</span><span class="sxs-lookup"><span data-stu-id="057c2-103">Send messages from outside a hub</span></span>

<span data-ttu-id="057c2-104">Tarafından [Mikael Mengistu](https://twitter.com/MikaelM_12)</span><span class="sxs-lookup"><span data-stu-id="057c2-104">By [Mikael Mengistu](https://twitter.com/MikaelM_12)</span></span>

<span data-ttu-id="057c2-105">SignalR hub'ı SignalR sunucuya bağlı olan istemcilerin ileti göndermek için çekirdek soyutlamadır.</span><span class="sxs-lookup"><span data-stu-id="057c2-105">The SignalR hub is the core abstraction for sending messages to clients connected to the SignalR server.</span></span> <span data-ttu-id="057c2-106">Uygulama kullanarak diğer yerlerden iletileri göndermek mümkündür `IHubContext` hizmet.</span><span class="sxs-lookup"><span data-stu-id="057c2-106">It's also possible to send messages from other places in your app using the `IHubContext` service.</span></span> <span data-ttu-id="057c2-107">Bu makalede, bir SignalR erişmek açıklanmaktadır `IHubContext` hub dışında istemcilere bildirimleri göndermek için.</span><span class="sxs-lookup"><span data-stu-id="057c2-107">This article explains how to access a SignalR `IHubContext` to send notifications to clients from outside a hub.</span></span>

<span data-ttu-id="057c2-108">[Görüntülemek veya karşıdan örnek kod](https://github.com/aspnet/Docs/tree/master/aspnetcore/signalr/hubcontext/sample/) [(nasıl indirileceğini)](xref:tutorials/index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="057c2-108">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/signalr/hubcontext/sample/) [(how to download)](xref:tutorials/index#how-to-download-a-sample)</span></span>

## <a name="get-an-instance-of-ihubcontext"></a><span data-ttu-id="057c2-109">Örneği Al `IHubContext`</span><span class="sxs-lookup"><span data-stu-id="057c2-109">Get an instance of `IHubContext`</span></span>

<span data-ttu-id="057c2-110">ASP.NET Core SignalR öğesinde örneği erişebilirsiniz `IHubContext` bağımlılık ekleme aracılığıyla.</span><span class="sxs-lookup"><span data-stu-id="057c2-110">In ASP.NET Core SignalR, you can access an instance of `IHubContext` via dependency injection.</span></span> <span data-ttu-id="057c2-111">Bir örneğini ekleme `IHubContext` bir denetleyicide, ara yazılım veya diğer dı hizmet uygulamasına.</span><span class="sxs-lookup"><span data-stu-id="057c2-111">You can inject an instance of `IHubContext` into a controller, middleware, or other DI service.</span></span> <span data-ttu-id="057c2-112">İstemcilere iletileri göndermek için örneği kullanın.</span><span class="sxs-lookup"><span data-stu-id="057c2-112">Use the instance to send messages to clients.</span></span>

> [!NOTE]
> <span data-ttu-id="057c2-113">Bu GlobalHost erişim sağlamak için kullanılan ASP.NET SignalR farklıdır `IHubContext`.</span><span class="sxs-lookup"><span data-stu-id="057c2-113">This differs from ASP.NET SignalR which used GlobalHost to provide access to the `IHubContext`.</span></span> <span data-ttu-id="057c2-114">ASP.NET Core genel bu singleton gereksinimini ortadan kaldırır bir bağımlılık ekleme framework sahiptir.</span><span class="sxs-lookup"><span data-stu-id="057c2-114">ASP.NET Core has a dependency injection framework that removes the need for this global singleton.</span></span>

### <a name="inject-an-instance-of-ihubcontext-in-a-controller"></a><span data-ttu-id="057c2-115">Bir örneğini ekleme `IHubContext` bir denetleyicide</span><span class="sxs-lookup"><span data-stu-id="057c2-115">Inject an instance of `IHubContext` in a controller</span></span>

<span data-ttu-id="057c2-116">Bir örneğini ekleme `IHubContext` , oluşturucuya ekleyerek bir denetleyici içine:</span><span class="sxs-lookup"><span data-stu-id="057c2-116">You can inject an instance of `IHubContext` into a controller by adding it to your constructor:</span></span>

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=12-19,57)]

<span data-ttu-id="057c2-117">Şimdi, bir örneğine erişimi olan `IHubContext`, hub, değilmiş gibi hub yöntemleri çağırabilirsiniz.</span><span class="sxs-lookup"><span data-stu-id="057c2-117">Now, with access to an instance of `IHubContext`, you can call hub methods as if you were in the hub itself.</span></span>

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=21-25)]

### <a name="get-an-instance-of-ihubcontext-in-middleware"></a><span data-ttu-id="057c2-118">Bir örneğini almak `IHubContext` Ara yazılımında</span><span class="sxs-lookup"><span data-stu-id="057c2-118">Get an instance of `IHubContext` in middleware</span></span>

<span data-ttu-id="057c2-119">Erişim `IHubContext` ara yazılım ardışık düzenini içinde şu şekilde:</span><span class="sxs-lookup"><span data-stu-id="057c2-119">Access the `IHubContext` within the middleware pipeline like so:</span></span>

```csharp
app.Use(next => (context) =>
{
    var hubContext = (IHubContext<MyHub>)context
                        .RequestServices
                        .GetServices<IHubContext<MyHub>>();
    //...
});
```

> [!NOTE]
> <span data-ttu-id="057c2-120">Ne zaman hub yöntemleri çağrılmadan gelen dışında `Hub` sınıfı, çağırmayla ilgili hiçbir çağıran yoktur.</span><span class="sxs-lookup"><span data-stu-id="057c2-120">When hub methods are called from outside of the `Hub` class, there's no caller associated with the invocation.</span></span> <span data-ttu-id="057c2-121">Bu nedenle, hiçbir erişim yoktur `ConnectionId`, `Caller`, ve `Others` özellikleri.</span><span class="sxs-lookup"><span data-stu-id="057c2-121">Therefore, there's no access to the `ConnectionId`, `Caller`, and `Others` properties.</span></span>

## <a name="related-resources"></a><span data-ttu-id="057c2-122">İlgili kaynaklar</span><span class="sxs-lookup"><span data-stu-id="057c2-122">Related resources</span></span>

* [<span data-ttu-id="057c2-123">Kullanmaya başlama</span><span class="sxs-lookup"><span data-stu-id="057c2-123">Get started</span></span>](xref:tutorials/signalr)
* [<span data-ttu-id="057c2-124">Merkezler</span><span class="sxs-lookup"><span data-stu-id="057c2-124">Hubs</span></span>](xref:signalr/hubs)
* [<span data-ttu-id="057c2-125">Azure'a Yayımlama</span><span class="sxs-lookup"><span data-stu-id="057c2-125">Publish to Azure</span></span>](xref:signalr/publish-to-azure-web-app)