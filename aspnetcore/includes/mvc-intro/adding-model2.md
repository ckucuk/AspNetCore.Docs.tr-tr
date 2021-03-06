## <a name="add-initial-migration-and-update-the-database"></a>İlk geçiş ekleyin ve veritabanını güncelleştir

* Bir komut istemi açın ve proje dizinine gidin. (Dizinini içeren *Startup.cs* dosyası).

* Komut isteminde aşağıdaki komutları çalıştırın:

  ```console
  dotnet restore
  dotnet ef migrations add Initial
  dotnet ef database update
  ```

  [.NET core](/dotnet/core/tools/index) bir çoklu platform .NET uygulamasıdır. Bu komutlar neler aşağıda verilmiştir:

  * [DotNet restore](/dotnet/core/tools/dotnet-restore): Belirtilen NuGet paketlerini indirir *.csproj* dosya.
  * `dotnet ef migrations add Initial` Entity Framework .NET Core CLI geçişleri komutu çalıştırır ve ilk geçiş oluşturur. "Ekle" sonra parametre geçişi atadığınız bir addır. Burada, ilk veritabanı geçiş olduğundan "Başlangıç" geçiş adlandırıyorsunuz. Bu işlem oluşturur *geçişlerini / /\<tarih-saat > _Initial.cs* eklemek için geçiş komutları içeren dosya *film* veritabanına tablo.
  * `dotnet ef database update`  Veritabanı, yeni oluşturduğumuz geçiş ile güncelleştirir.

Sonraki öğreticide veritabanı ve bağlantı dizesini hakkında bilgi edineceksiniz. Veri modeli değişikliklerini hakkında bilgi edineceksiniz [bir alan eklemek](xref:tutorials/first-mvc-app/new-field) öğretici.
