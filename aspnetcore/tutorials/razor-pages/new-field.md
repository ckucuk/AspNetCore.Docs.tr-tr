---
title: ASP.NET Core Razor sayfasına yeni bir alan ekleyin
author: rick-anderson
description: Entity Framework Core ile Razor sayfasına nasıl yeni bir alan ekleneceğini gösterir
ms.author: riande
ms.custom: mvc
ms.date: 7/23/2019
uid: tutorials/razor-pages/new-field
ms.openlocfilehash: b31711eb6f797de2de1559a3303e14b32a88f1ff
ms.sourcegitcommit: b3ebf96560b75b752d0e71161d788da800ad0999
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 10/24/2019
ms.locfileid: "72822377"
---
# <a name="add-a-new-field-to-a-razor-page-in-aspnet-core"></a>ASP.NET Core Razor sayfasına yeni bir alan ekleyin

Tarafından [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

[!INCLUDE[](~/includes/rp/download.md)]

Bu bölümde [Entity Framework](/ef/core/get-started/aspnetcore/new-db) için Code First Migrations kullanılır:

* Modele yeni bir alan ekleyin.
* Yeni alan şeması değişikliğini veritabanına geçirin.

Bir veritabanını otomatik olarak oluşturmak için EF Code First kullanırken Code First:

* Veritabanı şemasının oluşturulduğu model sınıflarıyla eşitlenmiş olup olmadığını izlemek için veritabanına bir `__EFMigrationsHistory` tablosu ekler.
* Model sınıfları DB ile eşitlenmiyorsa, EF bir özel durum oluşturur.

Şema/modelin eşitlemede otomatik olarak doğrulanması, tutarsız veritabanı/kod sorunlarını bulmayı kolaylaştırır.

## <a name="adding-a-rating-property-to-the-movie-model"></a>Film modeline bir derecelendirme özelliği ekleme

*Modeller/film. cs* dosyasını açın ve bir `Rating` özelliği ekleyin:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRating.cs?highlight=13&name=snippet)]

Uygulamayı derleyin.

*Sayfaları/filmleri/dizini. cshtml*'yi düzenleyin ve bir `Rating` alanı ekleyin:

<a name="addrat"></a>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie30/SnapShots/IndexRating.cshtml?highlight=40-42,62-64)]

Aşağıdaki sayfaları güncelleştirin:

* Silme ve Ayrıntılar sayfalarına `Rating` alanını ekleyin.
* [Create. cshtml](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Create.cshtml) dosyasını bir `Rating` alanla güncelleştirin.
* Düzenleme sayfasına `Rating` alanını ekleyin.

VERITABANı yeni alanı içerecek şekilde güncelleştirilene kadar uygulama çalışmaz. Veritabanını güncelleştirmeden uygulamayı çalıştırmak bir `SqlException`oluşturur:

`SqlException: Invalid column name 'Rating'.`

`SqlException` özel durumu, güncelleştirilmiş film modeli sınıfının, veritabanının film tablosunun şemasından farklı olmasından kaynaklanır. (Veritabanı tablosunda `Rating` sütunu yoktur.)

Hatayı çözmek için birkaç yaklaşım vardır:

1. Yeni model sınıfı şemasını kullanarak veritabanını otomatik olarak bırakıp yeniden oluşturmaya Entity Framework. Bu yaklaşım, geliştirme döngüsünün başlarında daha erken bir yoldur; modeli ve veritabanı şemasını birlikte hızla gelişmenize olanak tanır. Downsıde, veritabanında var olan verileri kaybetmeniz. Bu yaklaşımı bir üretim veritabanında kullanmayın! DB 'yi şema değişikliklerinde bırakıp bir başlatıcı kullanarak veritabanının test verileriyle otomatik olarak çekirdeğini oluşturmak, genellikle bir uygulama geliştirmeye yönelik üretken bir yoldur.

2. Mevcut veritabanının şemasını model sınıflarıyla eşleşecek şekilde açıkça değiştirin. Bu yaklaşımın avantajı, verilerinizi tutmanızı kullanmaktır. Bu değişikliği el ile ya da bir veritabanı değişiklik betiği oluşturarak yapabilirsiniz.

3. Veritabanı şemasını güncelleştirmek için Code First Migrations kullanın.

Bu öğretici için Code First Migrations kullanın.

`SeedData` sınıfını yeni sütun için bir değer sağlayacak şekilde güncelleştirin. Aşağıda örnek bir değişiklik gösterilmektedir, ancak her bir `new Movie` bloğu için bu değişikliği yapmak isteyeceksiniz.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

[Tamamlanan SeedData.cs dosyasına](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Models/SeedDataRating.cs)bakın.

Çözümü oluşturun.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a>Derecelendirme alanı için bir geçiş ekleyin

**Araçlar** menüsünde **NuGet Paket Yöneticisi > Paket Yöneticisi konsolu**' nu seçin.
PMC'de aşağıdaki komutları girin:

```powershell
Add-Migration Rating
Update-Database
```

`Add-Migration` komutu, çerçeveye şunları belirtir:

* `Movie` modelini `Movie` DB şemasıyla karşılaştırın.
* DB şemasını yeni modele geçirmek için kod oluşturun.

"Derecelendirme" adı rastgele olur ve geçiş dosyasını adlandırmak için kullanılır. Geçiş dosyası için anlamlı bir ad kullanılması yararlı olur.

`Update-Database` komutu, çerçeveye şema değişikliklerini uygulamaya ve var olan verileri korumanıza bildirir.

<a name="ssox"></a>

VERITABANıNDAKI tüm kayıtları silerseniz, başlatıcı DB 'yi temel alır ve `Rating` alanını içerir. Bunu, tarayıcıda veya [SQL Server Nesne Gezgini](xref:tutorials/razor-pages/sql#ssox) (ssox) silme bağlantılarıyla yapabilirsiniz.

Başka bir seçenek de veritabanını silmek ve geçişleri kullanarak veritabanını yeniden oluşturmaktır. SSOX 'te veritabanını silmek için:

* SSOX 'te veritabanını seçin.
* Veritabanına sağ tıklayın ve *Sil*' i seçin.
* **Mevcut bağlantıları kapat**' a bakın.
* Seçin **Tamam**.
* [PMC](xref:tutorials/razor-pages/new-field#pmc)'de veritabanını güncelleştirin:

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code/Mac için Visual Studio](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a>Veritabanını bırakıp yeniden oluşturun

[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

Geçiş klasörünü silin.  Veritabanını yeniden oluşturmak için aşağıdaki komutları kullanın.

```dotnetcli
dotnet ef database drop
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

Uygulamayı çalıştırın ve bir `Rating` alanı ile film oluşturabileceğiniz/düzenleyebileceğiniz/görüntüleydiğinizi doğrulayın. Veritabanı birlikte olmazsa, `SeedData.Initialize` yönteminde bir kesme noktası ayarlayın.

## <a name="additional-resources"></a>Ek kaynaklar

* [Bu öğreticinin YouTube sürümü](https://youtu.be/3i7uMxiGGR8)

> [!div class="step-by-step"]
> [Önceki: arama](xref:tutorials/razor-pages/search)
> ekleme [Sonraki: doğrulama ekleme](xref:tutorials/razor-pages/validation)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!INCLUDE[](~/includes/rp/download.md)]

Bu bölümde [Entity Framework](/ef/core/get-started/aspnetcore/new-db) için Code First Migrations kullanılır:

* Modele yeni bir alan ekleyin.
* Yeni alan şeması değişikliğini veritabanına geçirin.

Bir veritabanını otomatik olarak oluşturmak için EF Code First kullanırken Code First:

* Veritabanı şemasının oluşturulduğu model sınıflarıyla uyumlu olup olmadığını izlemek için veritabanına bir tablo ekler.
* Model sınıfları DB ile eşitlenmiyorsa, EF bir özel durum oluşturur.

Şema/modelin eşitlemede otomatik olarak doğrulanması, tutarsız veritabanı/kod sorunlarını bulmayı kolaylaştırır.

## <a name="adding-a-rating-property-to-the-movie-model"></a>Film modeline bir derecelendirme özelliği ekleme

*Modeller/film. cs* dosyasını açın ve bir `Rating` özelliği ekleyin:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/MovieDateRating.cs?highlight=13&name=snippet)]

Uygulamayı derleyin.

*Sayfaları/filmleri/dizini. cshtml*'yi düzenleyin ve bir `Rating` alanı ekleyin:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexRating.cshtml?highlight=40-42,61-63)]

Aşağıdaki sayfaları güncelleştirin:

* Silme ve Ayrıntılar sayfalarına `Rating` alanını ekleyin.
* [Create. cshtml](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml) dosyasını bir `Rating` alanla güncelleştirin.
* Düzenleme sayfasına `Rating` alanını ekleyin.

VERITABANı yeni alanı içerecek şekilde güncelleştirilene kadar uygulama çalışmaz. Şimdi çalıştırırsanız, uygulama bir `SqlException`oluşturur:

`SqlException: Invalid column name 'Rating'.`

Bu hata, güncelleştirilmiş film modeli sınıfının, veritabanının film tablosunun şemasından farklı olmasından kaynaklanır. (Veritabanı tablosunda `Rating` sütunu yoktur.)

Hatayı çözmek için birkaç yaklaşım vardır:

1. Yeni model sınıfı şemasını kullanarak veritabanını otomatik olarak bırakıp yeniden oluşturmaya Entity Framework. Bu yaklaşım, geliştirme döngüsünün başlarında daha erken bir yoldur; modeli ve veritabanı şemasını birlikte hızla gelişmenize olanak tanır. Downsıde, veritabanında var olan verileri kaybetmeniz. Bu yaklaşımı bir üretim veritabanında kullanmayın! DB 'yi şema değişikliklerinde bırakıp bir başlatıcı kullanarak veritabanının test verileriyle otomatik olarak çekirdeğini oluşturmak, genellikle bir uygulama geliştirmeye yönelik üretken bir yoldur.

2. Mevcut veritabanının şemasını model sınıflarıyla eşleşecek şekilde açıkça değiştirin. Bu yaklaşımın avantajı, verilerinizi tutmanızı kullanmaktır. Bu değişikliği el ile ya da bir veritabanı değişiklik betiği oluşturarak yapabilirsiniz.

3. Veritabanı şemasını güncelleştirmek için Code First Migrations kullanın.

Bu öğretici için Code First Migrations kullanın.

`SeedData` sınıfını yeni sütun için bir değer sağlayacak şekilde güncelleştirin. Aşağıda örnek bir değişiklik gösterilmektedir, ancak her bir `new Movie` bloğu için bu değişikliği yapmak isteyeceksiniz.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

[Tamamlanan SeedData.cs dosyasına](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs)bakın.

Çözümü oluşturun.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a>Derecelendirme alanı için bir geçiş ekleyin

**Araçlar** menüsünde **NuGet Paket Yöneticisi > Paket Yöneticisi konsolu**' nu seçin.
PMC'de aşağıdaki komutları girin:

```powershell
Add-Migration Rating
Update-Database
```

`Add-Migration` komutu, çerçeveye şunları belirtir:

* `Movie` modelini `Movie` DB şemasıyla karşılaştırın.
* DB şemasını yeni modele geçirmek için kod oluşturun.

"Derecelendirme" adı rastgele olur ve geçiş dosyasını adlandırmak için kullanılır. Geçiş dosyası için anlamlı bir ad kullanılması yararlı olur.

`Update-Database` komutu, çerçeveye şema değişikliklerini veritabanına uygulamasını söyler.

<a name="ssox"></a>

VERITABANıNDAKI tüm kayıtları silerseniz, başlatıcı DB 'yi temel alır ve `Rating` alanını içerir. Bunu, tarayıcıda veya [SQL Server Nesne Gezgini](xref:tutorials/razor-pages/sql#ssox) (ssox) silme bağlantılarıyla yapabilirsiniz.

Başka bir seçenek de veritabanını silmek ve geçişleri kullanarak veritabanını yeniden oluşturmaktır. SSOX 'te veritabanını silmek için:

* SSOX 'te veritabanını seçin.
* Veritabanına sağ tıklayın ve *Sil*' i seçin.
* **Mevcut bağlantıları kapat**' a bakın.
* Seçin **Tamam**.
* [PMC](xref:tutorials/razor-pages/new-field#pmc)'de veritabanını güncelleştirin:

  ```powershell
  Update-Database
  ```

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code/Mac için Visual Studio](#tab/visual-studio-code+visual-studio-mac)

### <a name="drop-and-re-create-the-database"></a>Veritabanını bırakıp yeniden oluşturun

[!INCLUDE[](~/includes/RP-mvc-shared/sqlite-warn.md)]

Veritabanını silin ve geçişleri kullanarak veritabanını yeniden oluşturun. Veritabanını silmek için veritabanı dosyasını (*Mvcmovie. db*) silin. Sonra `ef database update` komutunu çalıştırın:

```dotnetcli
dotnet ef database update
```

---

Uygulamayı çalıştırın ve bir `Rating` alanı ile film oluşturabileceğiniz/düzenleyebileceğiniz/görüntüleydiğinizi doğrulayın. Veritabanı birlikte olmazsa, `SeedData.Initialize` yönteminde bir kesme noktası ayarlayın.

## <a name="additional-resources"></a>Ek kaynaklar

* [Bu öğreticinin YouTube sürümü](https://youtu.be/3i7uMxiGGR8)

> [!div class="step-by-step"]
> [Önceki: arama](xref:tutorials/razor-pages/search)
> ekleme [Sonraki: doğrulama ekleme](xref:tutorials/razor-pages/validation)

::: moniker-end
