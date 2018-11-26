---
title: Web API Çözümleyicileri kullanma
author: pranavkm
description: Web API Çözümleyicileri Microsoft.AspNetCore.Mvc.Api.Analyzers içinde hakkında bilgi edinin.
monikerRange: '>= aspnetcore-2.2'
ms.author: pranavkm
ms.custom: mvc
ms.date: 11/13/2018
uid: web-api/advanced/analyzers
ms.openlocfilehash: 89424d89ec2b3125fd3c6b7c86fed2d292b153e6
ms.sourcegitcommit: f202864efca81a72ea7120c0692940c40d9d0630
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 11/14/2018
ms.locfileid: "51635368"
---
# <a name="use-web-api-analyzers"></a>Web API Çözümleyicileri kullanma

ASP.NET Core 2.2 tanıtır [Microsoft.AspNetCore.Mvc.Api.Analyzers](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Api.Analyzers) web API'leri için çözümleyicilerini içeren bir NuGet paketi. Denetleyicileri ile açıklanan Çözümleyicileri çalışmak <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute>, oluşturmaya çalışırken [API kuralları](xref:web-api/advanced/conventions).

## <a name="package-installation"></a>Paket yüklemesi

`Microsoft.AspNetCore.Mvc.Api.Analyzers` aşağıdaki yaklaşımlardan birini eklenebilir:

### <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Gelen **Paket Yöneticisi Konsolu** penceresi:
  * Git **görünümü** > **diğer Windows** > **Paket Yöneticisi Konsolu**.
  * Dizin gidin *ApiConventions.csproj* dosya yok.
  * Aşağıdaki komutu yürütün:

    ```powershell
    Install-Package Microsoft.AspNetCore.Mvc.Api.Analyzers
    ```

* Gelen **NuGet paketlerini Yönet** iletişim:
  * Projeye sağ **Çözüm Gezgini** > **NuGet paketlerini Yönet**.
  * Ayarlama **paket kaynağı** "nuget.org'da".
  * Arama kutusuna "Microsoft.AspNetCore.Mvc.Api.Analyzers" girin.
  * "Microsoft.AspNetCore.Mvc.Api.Analyzers" paketinden seçin **Gözat** sekmesine **yükleme**.

### <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Mac için Visual Studio](#tab/visual-studio-mac)

* Sağ *paketleri* klasöründe **çözüm bölmesi** > **paketleri Ekle...** .
* Ayarlama **paketleri Ekle** pencerenin **kaynak** "nuget.org'da" açılır.
* Arama kutusuna "Microsoft.AspNetCore.Mvc.Api.Analyzers" girin.
* Sonuçlar bölmesinde "Microsoft.AspNetCore.Mvc.Api.Analyzers" paketi seçin ve tıklayın **Paketi Ekle**.

### <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Aşağıdaki komutu çalıştırın **tümleşik Terminalini**:

```console
dotnet add ApiConventions.csproj package Microsoft.AspNetCore.Mvc.Api.Analyzers
```

### <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

Şu komutu çalıştırın:

```console
dotnet add ApiConventions.csproj package Microsoft.AspNetCore.Mvc.Api.Analyzers
```

---

## <a name="analyzers-for-api-conventions"></a>API kuralları için Çözümleyicileri

Açık API belgeleri, durum kodları ve eylem döndürebilir ve yanıt türleri içerir. ASP.NET Core MVC gibi öznitelikleri <xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute> ve <xref:Microsoft.AspNetCore.Mvc.ProducesAttribute> eylem belgelemek için kullanılır. <xref:tutorials/web-api-help-pages-using-swagger> API'nizi belgeleme üzerinde daha fazla ayrıntıya gider.

Paketteki Çözümleyicileri birini inceler denetleyicileri ile açıklanan <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> ve yanıtlarını tamamen belge yok eylemleri tanımlar. Aşağıdaki örnek göz önünde bulundurun:

[!code-csharp[](conventions/sample/Controllers/ContactsController.cs?name=missing404docs&highlight=9)]

Önceki eylemi HTTP başarı dönüş türü, ancak belge olmayan HTTP 404 hatası durum kodu 200 belgeler. Çözümleyici eksik HTTP 404 durum kodu belgelerine bir uyarı olarak bildirir. Sorunu düzeltmek için bir seçenek sağlanır.