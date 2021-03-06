---
title: ASP.NET Core Blazor kullanmaya başlama
author: guardrex
description: Seçtiğiniz araç ile Blazor bir uygulama oluşturarak Blazor kullanmaya başlayın.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/18/2019
no-loc:
- Blazor
- SignalR
uid: blazor/get-started
ms.openlocfilehash: 642881b5400a70a99f6e7e262d2a2f1038389ce7
ms.sourcegitcommit: eca76bd065eb94386165a0269f1e95092f23fa58
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 01/24/2020
ms.locfileid: "76726849"
---
# <a name="get-started-with-aspnet-core-opno-locblazor"></a>ASP.NET Core Blazor kullanmaya başlama

[Daniel Roth](https://github.com/danroth27) ve [Luke Latham](https://github.com/guardrex) tarafından

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Blazorkullanmaya başlayın:

1. [.NET Core 3,1 SDK 'sını](https://dotnet.microsoft.com/download/dotnet-core/3.1)yükler.

1. İsteğe bağlı olarak [Blazor WebAssembly](xref:blazor/hosting-models#blazor-webassembly) şablonunu yükler:
   * [.NET Core 3,1 veya üzeri (Önizleme) SDK 'sını](https://dotnet.microsoft.com/download/dotnet-core/3.1)yükler.
   * Komut kabuğu 'nda aşağıdaki komutu çalıştırın. [Microsoft. AspNetCore.Blazor. Şablon](https://www.nuget.org/packages/Microsoft.AspNetCore.Blazor.Templates/) paketinin önizleme sürümü varsa Blazor WebAssembly önizleme aşamasındadır.

   ```dotnetcli
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.1.0-preview4.19579.2
   ```

1. Araç seçiminiz için yönergeleri izleyin:

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1 \. **ASP.net ve Web geliştirme** iş yüküyle [Visual Studio 2019 sürüm 16,4 veya üstünü yükledikten sonra](https://visualstudio.microsoft.com/vs/preview/) .

   2 \. Yeni bir proje oluşturun.

   3 \. **Blazor uygulama**' yı seçin. **İleri**'yi seçin.

   4 \. **Proje adı** alanında bir proje adı girin veya varsayılan proje adını kabul edin. **Konum** girişinin doğru olduğunu onaylayın veya proje için bir konum belirtin. Seçin **oluşturma**.

   5 \. Blazor Weelsembly deneyimi için, **Webassembly uygulama şablonunuBlazor** seçin. Blazor sunucusu deneyimi için **Blazor sunucusu uygulama** şablonunu seçin. Seçin **oluşturma**. Blazor, *Blazor sunucusu* ve *Blazor webassembly*'yi barındıran iki hakkında bilgi için bkz. <xref:blazor/hosting-models>.

   6 \. Uygulamayı çalıştırmak için **Ctrl**+**F5** tuşuna basın.

   > [!NOTE]
   > Blazor Visual Studio uzantısını ASP.NET Core Blazor önceki bir önizleme sürümü için yüklediyseniz (Preview 6 veya daha önceki bir sürümü), uzantıyı kaldırabilirsiniz. Blazor şablonlarının bir komut kabuğu 'na yüklenmesi artık Visual Studio 'daki şablonları yüzeye eklemek yeterlidir.

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1 \. [Visual Studio Code](https://code.visualstudio.com/)'i yükler.

   2 \. En son [ C# Visual Studio Code uzantısını](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)yükler.

   3 \. Blazor Weelsembly deneyimi için, komut kabuğu 'nda aşağıdaki komutu yürütün:

      ```dotnetcli
      dotnet new blazorwasm -o WebApplication1
      ```

      Blazor sunucusu deneyimi için, komut kabuğu 'nda aşağıdaki komutu yürütün:

      ```dotnetcli
      dotnet new blazorserver -o WebApplication1
      ```

      Blazor, *Blazor sunucusu* ve *Blazor webassembly*'yi barındıran iki hakkında bilgi için bkz. <xref:blazor/hosting-models>.

   4 \. Visual Studio Code 'de *WebApplication1* klasörünü açın.

   5 \. Bir Blazor Server projesi için IDE, projeyi derlemek ve hatalarını ayıklamak için varlık eklemenizi ister. **Evet**' i seçin.

   6 \. Blazor sunucusu uygulaması kullanıyorsanız, Visual Studio Code hata ayıklayıcıyı kullanarak uygulamayı çalıştırın. Blazor WebAssembly uygulaması kullanıyorsanız, uygulamanın proje klasöründen `dotnet run` yürütün.

   7 \. Bir tarayıcıda `https://localhost:5001`' a gidin.

   # <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Mac için Visual Studio](#tab/visual-studio-mac)

   1 \. [Mac için Visual Studio](https://visualstudio.microsoft.com/vs/mac/)'i yükler.

   2 \. **Yeni çözüm** > **Dosya** seçin veya yeni bir **Proje**oluşturun.

   3 \. Kenar çubuğunda **.NET Core** > **uygulaması**' nı seçin.

   4 \. **Blazor sunucusu uygulama** şablonunu seçin. Şu anda yalnızca Blazor sunucusu şablonu Mac için Visual Studio kullanılabilir. Blazor Weelsembly deneyimi için **.NET Core CLI** sekmesindeki yönergeleri izleyin. Blazor sunucusu şablonunu seçtikten sonra **İleri**' yi seçin. Blazor, *Blazor sunucusu* ve *Blazor webassembly*'yi barındıran iki hakkında bilgi için bkz. <xref:blazor/hosting-models>.

   <!-- For a Blazor WebAssembly experience, select the **Blazor WebAssembly App** template. Select **Next**. -->

   5 \. **Hedef Framework 'ü** **.NET Core 3,1** olarak ayarlayın ve **İleri ' yi**seçin.

   6 \. **Proje adı** alanında, uygulamayı `WebApplication1`olarak adlandırın. Seçin **oluşturma**.

   7 \. Uygulamayı *hata ayıklayıcısı olmadan*çalıştırmak Için **hata ayıklama olmadan** **Çalıştır > Çalıştır** ' ı seçin. Uygulamayı hata *ayıklayıcıyla*çalıştırmak Için, **hata ayıklamayı Başlat** ile uygulamayı çalıştırın.

   Geliştirme sertifikasına güvenmek için bir istem görünürse, sertifikaya güvenin ve devam edin.

   # <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli/)

   Blazor Weelsembly deneyimi için, komut kabuğu 'nda aşağıdaki komutları yürütün:

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Blazor sunucusu deneyimi için, komut kabuğu 'nda aşağıdaki komutları yürütün:

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Blazor, *Blazor sunucusu* ve *Blazor webassembly*'yi barındıran iki hakkında bilgi için bkz. <xref:blazor/hosting-models>.

   Bir tarayıcıda `https://localhost:5001`' a gidin.

   ---

Kenar çubuğu 'ndaki sekmelerde birden çok sayfa mevcuttur:

* Ana Sayfası
* Sayaç
* Verileri getir

Sayaç sayfasında, bir sayfa yenilemesi olmadan sayacı artırmak için **bana tıklama** düğmesini seçin. Bir Web sayfasında normal olarak bir sayacı artırma, JavaScript yazmayı gerektirir, ancak kullanabilirsiniz C#Blazor.

*Pages/Counter. Razor*:

[!code-razor[](get-started/samples_snapshot/3.x/Counter1.razor?highlight=7,12-15)]

Tarayıcıda `/counter` için bir istek, en üstteki `@page` yönergesi tarafından belirtilen şekilde `Counter` bileşeninin içeriğini işlemesine neden olur. Bileşenler, daha sonra, Kullanıcı arabirimini esnek ve verimli bir şekilde güncelleştirmek için kullanılabilen işleme ağacının bellek içi gösterimine işlenir.

**Bana tıklama** düğmesi her seçildiğinde:

* `onclick` olayı tetiklenir.
* `IncrementCount` yöntemi çağrılır.
* `currentCount` artırılır.
* Bileşen yeniden işlenir.

Çalışma zamanı, yeni içeriği önceki içerikle karşılaştırır ve yalnızca değiştirilen içeriği Belge Nesne Modeli (DOM) öğesine uygular.

HTML sözdizimini kullanarak başka bir bileşene bileşen ekleyin. Örneğin, `Index` bileşenine bir `<Counter />` öğesi ekleyerek `Counter` bileşenini uygulamanın giriş sayfasına ekleyin.

*Pages/Index. Razor*:

[!code-razor[](get-started/samples_snapshot/3.x/Index1.razor?highlight=7)]

Uygulamayı çalıştırın. Giriş sayfasının `Counter` bileşeni tarafından kendi sayacı vardır.

Bileşen parametreleri, alt bileşende özellikler ayarlamanıza olanak tanıyan öznitelikler veya [alt içerik](xref:blazor/components#child-content)kullanılarak belirtilir. `Counter` bileşenine bir parametre eklemek için, bileşenin `@code` bloğunu güncelleştirin:

* `[Parameter]` özniteliğiyle `IncrementAmount` için ortak özellik ekleyin.
* `currentCount`değerini artırdığınızda `IncrementAmount` kullanmak için `IncrementCount` yöntemini değiştirin.

*Pages/Counter. Razor*:

[!code-razor[](get-started/samples_snapshot/3.x/Counter2.razor?highlight=12-13,17)]

`Index` bileşenin `<Counter>` öğesindeki bir özniteliği kullanarak `IncrementAmount` belirtin.

*Pages/Index. Razor*:

[!code-razor[](get-started/samples_snapshot/3.x/Index2.razor?highlight=7)]

Uygulamayı çalıştırın. `Index` bileşeni, **bana tıklama** düğmesi seçildiğinde her seferinde on ile artan kendi sayacıdır. `/counter` `Counter` bileşeni (*Counter. Razor*), bir tarafından arttırmaya devam eder.

## <a name="next-steps"></a>Sonraki adımlar

<xref:tutorials/first-blazor-app>

## <a name="additional-resources"></a>Ek kaynaklar

* <xref:blazor/templates>
* <xref:signalr/introduction>
