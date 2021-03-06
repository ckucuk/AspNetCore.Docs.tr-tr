---
title: ASP.NET core'da ortam etiketi Yardımcısı
author: pkellner
description: Tüm özellikler dahil olmak üzere tanımlanan ASP.NET Core ortam etiketi Yardımcısı
ms.author: riande
ms.custom: mvc
ms.date: 10/10/2018
uid: mvc/views/tag-helpers/builtin-th/environment-tag-helper
ms.openlocfilehash: e2e038fe69da696b67f7aef61795e23dc8512fdf
ms.sourcegitcommit: 7a40c56bf6a6aaa63a7ee83a2cac9b3a1d77555e
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 07/12/2019
ms.locfileid: "67856130"
---
# <a name="environment-tag-helper-in-aspnet-core"></a>ASP.NET core'da ortam etiketi Yardımcısı

Tarafından [Peter Kellner](https://peterkellner.net), [Hisham Bin Ateya](https://twitter.com/hishambinateya), ve [Luke Latham](https://github.com/guardrex)

Ortam etiketi Yardımcısı koşullu olarak geçerli göre kapalı içeriğini işler [barındırma ortamı](xref:fundamentals/environments). Ortam etiketi Yardımcısı'nın tek öznitelik `names`, ortam adlarının virgülle ayrılmış listesidir. Sağlanan ortam adları geçerli ortamı hiçbiriyle, ekteki içeriğin işlenir.

Etiket Yardımcıları genel bakış için bkz. <xref:mvc/views/tag-helpers/intro>.

## <a name="environment-tag-helper-attributes"></a>Ortam etiketi Yardımcısı öznitelikleri

### <a name="names"></a>adlar

`names` Tek bir barındırma ortamı adı veya işleme ekteki içeriğin tetikleyen ortam adları barındırma virgülle ayrılmış listesini kabul eder.

Tarafından döndürülen değeri, geçerli ortam değerlerini karşılaştırılır [IHostingEnvironment.EnvironmentName](xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.EnvironmentName*). Karşılaştırma çalışması yoksayar.

Aşağıdaki örnek, bir ortam etiketi Yardımcısı kullanır. Barındırma ortamı hazırlık veya üretim olması durumunda içeriğinin işlenip:

```cshtml
<environment names="Staging,Production">
    <strong>HostingEnvironment.EnvironmentName is Staging or Production</strong>
</environment>
```

::: moniker range=">= aspnetcore-2.0"

## <a name="include-and-exclude-attributes"></a>dahil etme ve dışlama öznitelikleri

`include` & `exclude` dahil veya hariç tutulan barındırma ortamı adlarını temel alarak ekteki içeriğin işleme öznitelikleri denetimi.

### <a name="include"></a>include

`include` Özelliği için benzer bir davranış gösteriyor `names` özniteliği. Listelenen bir ortam `include` öznitelik değeri, uygulamanın barındırma ortamına eşleşmelidir ([IHostingEnvironment.EnvironmentName](xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.EnvironmentName*)) içeriği işlemek için `<environment>` etiketi.

```cshtml
<environment include="Staging,Production">
    <strong>HostingEnvironment.EnvironmentName is Staging or Production</strong>
</environment>
```

### <a name="exclude"></a>exclude

Tersine `include` özniteliği, içeriğini `<environment>` etiket listede bir ortam barındırma ortamı eşleşmediğinde işlenir `exclude` öznitelik değeri.

```cshtml
<environment exclude="Development">
    <strong>HostingEnvironment.EnvironmentName is not Development</strong>
</environment>
```

::: moniker-end

## <a name="additional-resources"></a>Ek kaynaklar

* <xref:fundamentals/environments>
