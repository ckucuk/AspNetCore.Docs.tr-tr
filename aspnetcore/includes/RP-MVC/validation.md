<!-- USED in RP and MVC tutorial -->

## <a name="add-validation-rules-to-the-movie-model"></a>Film modeline doğrulama kuralları ekleme

Dataaçıklamalarda ad alanı, bir sınıfa veya özelliğe bildirimli olarak uygulanan bir yerleşik doğrulama öznitelikleri kümesi sağlar. Dataaçıklamalarda, biçimlendirme ile ilgili Yardım `DataType` ve herhangi bir doğrulama sağlamayan gibi biçimlendirme öznitelikleri de bulunur.

Yerleşik`Required`, ,`RegularExpression`vedoğrulama özniteliklerinden yararlanmak için sınıfıgüncelleştirin.`Movie` `StringLength` `Range`

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Models/MovieDateRatingDA.cs?name=snippet1)]

Doğrulama öznitelikleri, uygulanan model özellikleri üzerinde zorlamak istediğiniz davranışı belirtir:

* `Required` Ve`MinimumLength` öznitelikleri bir özelliğin bir değere sahip olması gerektiğini belirtir; ancak hiçbir şey, bir kullanıcının bu doğrulamayı karşılamak için boşluk girmesini engeller.
* `RegularExpression` Öznitelik, hangi karakterlerin girişi yapabileceğini sınırlamak için kullanılır. Yukarıdaki kodda, "tarz":

  * Yalnızca harfler kullanılmalıdır.
  * İlk harfin büyük harfle olması gerekir. Boşluk, sayı ve özel karakterlere izin verilmez.

* `RegularExpression` "Derecelendirme":

  * İlk karakterin büyük harf olmasını gerektirir.
  * Sonraki boşlukların içindeki özel karakter ve sayılara izin verir. "PG-13" bir derecelendirme için geçerlidir, ancak bir "tarz" için başarısız olur.

* `Range` Özniteliği bir değeri belirtilen bir Aralık içinde kısıtlar.
* `StringLength` Özniteliği, bir dize özelliğinin en büyük uzunluğunu ve isteğe bağlı olarak en düşük uzunluğunu ayarlamanıza olanak sağlar.
* Değer türleri (örneğin, `decimal`, `int`, `float` `DateTime`), doğal olarak gereklidir ve `[Required]` özniteliğe gerek kalmaz.

Doğrulama kurallarının otomatik olarak uygulanmasını ASP.NET Core uygulamanızın daha sağlam olmasına yardımcı olur. Ayrıca, bir şeyi doğrulamayı unutmanızı ve veritabanına yanlışlıkla veri vermemesini de sağlar.
