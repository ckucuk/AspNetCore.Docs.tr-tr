---
title: HTTP REPL ile Web API 'Lerini test etme
author: scottaddie
description: HTTP REPL .NET Core küresel aracının bir ASP.NET Core Web API 'sini taramak ve test etmek için nasıl kullanılacağını öğrenin.
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 12/11/2019
uid: web-api/http-repl
ms.openlocfilehash: 15899917826fb6559244998766d99d00f56e0521
ms.sourcegitcommit: f259889044d1fc0f0c7e3882df0008157ced4915
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 01/21/2020
ms.locfileid: "76294727"
---
# <a name="test-web-apis-with-the-http-repl"></a>HTTP REPL ile Web API 'Lerini test etme

[Scott Ade](https://twitter.com/Scott_Addie) tarafından

HTTP okuma-değerlendirme-yazdırma döngüsü (REPL):

* .NET Core 'un her yerde desteklenen basit, platformlar arası bir komut satırı aracı desteklenir.
* ASP.NET Core Web API 'Lerini (ve non-ASP.NET çekirdek Web API 'Leri) test etmek ve sonuçlarını görüntülemek için kullanılır.
* Localhost ve Azure App Service dahil olmak üzere herhangi bir ortamda barındırılan Web API 'Lerini test etme özelliğine sahiptir.

Aşağıdaki [http fiilleri](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods) desteklenir:

* [DELETE](#test-http-delete-requests)
* [GET](#test-http-get-requests)
* [BAŞLı](#test-http-head-requests)
* [Seçenekler](#test-http-options-requests)
* [DÜZELTMESI](#test-http-patch-requests)
* [POST](#test-http-post-requests)
* [PUT](#test-http-put-requests)

Takip etmek için, [örnek ASP.NET Core Web API](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/http-repl/samples) 'sini ([indirme](xref:index#how-to-download-a-sample)) görüntüleyin veya indirin.

## <a name="prerequisites"></a>Prerequisites

* [!INCLUDE [2.1-SDK](~/includes/2.1-SDK.md)]

## <a name="installation"></a>Yükleme

HTTP REPL 'u yüklemek için aşağıdaki komutu çalıştırın:

```dotnetcli
dotnet tool install -g Microsoft.dotnet-httprepl
```

[.NET Core küresel aracı](/dotnet/core/tools/global-tools#install-a-global-tool) , [Microsoft. DotNet-httprepl](https://www.nuget.org/packages/Microsoft.dotnet-httprepl) NuGet paketinden yüklenir.

## <a name="usage"></a>Kullanım

Aracın başarıyla yüklenmesinden sonra, HTTP REPL 'u başlatmak için aşağıdaki komutu çalıştırın:

```console
httprepl
```

Kullanılabilir HTTP REPL komutlarını görüntülemek için aşağıdaki komutlardan birini çalıştırın:

```console
httprepl -h
```

```console
httprepl --help
```

Aşağıdaki çıktı görüntülenir:

```console
Usage:
  httprepl [<BASE_ADDRESS>] [options]

Arguments:
  <BASE_ADDRESS> - The initial base address for the REPL.

Options:
  -h|--help - Show help information.

Once the REPL starts, these commands are valid:

Setup Commands:
Use these commands to configure the tool for your API server

connect        Configures the directory structure and base address of the api server
set header     Sets or clears a header for all requests. e.g. `set header content-type application/json`

HTTP Commands:
Use these commands to execute requests against your application.

GET            get - Issues a GET request
POST           post - Issues a POST request
PUT            put - Issues a PUT request
DELETE         delete - Issues a DELETE request
PATCH          patch - Issues a PATCH request
HEAD           head - Issues a HEAD request
OPTIONS        options - Issues a OPTIONS request

Navigation Commands:
The REPL allows you to navigate your URL space and focus on specific APIs that you are working on.

set base       Set the base URI. e.g. `set base http://locahost:5000`
ls             Show all endpoints for the current path
cd             Append the given directory to the currently selected path, or move up a path when using `cd ..`

Shell Commands:
Use these commands to interact with the REPL shell.

clear          Removes all text from the shell
echo [on/off]  Turns request echoing on or off, show the request that was made when using request commands
exit           Exit the shell

REPL Customization Commands:
Use these commands to customize the REPL behavior.

pref [get/set] Allows viewing or changing preferences, e.g. 'pref set editor.command.default 'C:\\Program Files\\Microsoft VS Code\\Code.exe'`
run            Runs the script at the given path. A script is a set of commands that can be typed with one command per line
ui             Displays the Swagger UI page, if available, in the default browser

Use `help <COMMAND>` for more detail on an individual command. e.g. `help get`.
For detailed tool info, see https://aka.ms/http-repl-doc.
```

HTTP REPL komut tamamlama sağlar. <kbd>Sekme</kbd> tuşuna basıldığında, yazdığınız KARAKTERLERI veya API uç noktasını tamamlayacak komutların listesi üzerinden yinelenir. Aşağıdaki bölümlerde kullanılabilir CLı komutları ana hatlarıyla verilmiştir.

## <a name="connect-to-the-web-api"></a>Web API 'sine bağlanma

Aşağıdaki komutu çalıştırarak bir Web API 'sine bağlanın:

```console
httprepl <ROOT URI>
```

`<ROOT URI>`, Web API 'sinin temel URI 'sidir. Örneğin:

```console
httprepl https://localhost:5001
```

Alternatif olarak, HTTP REPL çalışırken istediğiniz zaman aşağıdaki komutu çalıştırın:

```console
connect <ROOT URI>
```

Örneğin:

```console
(Disconnected)~ connect https://localhost:5001
```

## <a name="manually-point-to-the-swagger-document-for-the-web-api"></a>Web API 'SI için Swagger belgesine el ile işaret edin

Yukarıdaki Connect komutu Swagger belgesini otomatik olarak bulmaya çalışacaktır. Bir nedenden dolayı yapamaması durumunda, `--swagger` seçeneğini kullanarak Web API 'SI için Swagger belgesinin URI 'sini belirtebilirsiniz:

```console
connect <ROOT URI> --swagger <SWAGGER URI>
```

Örneğin:

```console
(Disconnected)~ connect https://localhost:5001 --swagger /swagger/v1/swagger.json
```

## <a name="navigate-the-web-api"></a>Web API 'sinde gezin

### <a name="view-available-endpoints"></a>Kullanılabilir uç noktaları görüntüle

Web API adresinin geçerli yolundaki farklı uç noktaları (denetleyiciler) listelemek için `ls` veya `dir` komutunu çalıştırın:

```console
https://localhot:5001/~ ls
```

Aşağıdaki çıkış biçimi görüntülenir:

```console
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/~
```

Yukarıdaki çıkış iki denetleyicinin bulunduğunu gösterir: `Fruits` ve `People`. Her iki denetleyici de parametresiz HTTP GET ve POST işlemlerini destekler.

Belirli bir denetleyicide gezinmek daha ayrıntılı bilgi gösterir. Örneğin, aşağıdaki komut çıktısı, `Fruits` denetleyicisinin HTTP GET, PUT ve DELETE işlemlerini de desteklediğini gösterir. Bu işlemlerin her biri, rotada bir `id` parametresi bekler:

```console
https://localhost:5001/fruits~ ls
.      [get|post]
..     []
{id}   [get|put|delete]

https://localhost:5001/fruits~
```

Alternatif olarak, Web API 'sinin Swagger Kullanıcı Arabirimi sayfasını bir tarayıcıda açmak için `ui` komutunu çalıştırın. Örneğin:

```console
https://localhost:5001/~ ui
```

### <a name="navigate-to-an-endpoint"></a>Bir uç noktaya gitme

Web API 'sindeki farklı bir uç noktaya gitmek için `cd` komutunu çalıştırın:

```console
https://localhost:5001/~ cd people
```

`cd` komutundan sonraki yol büyük/küçük harfe duyarlıdır. Aşağıdaki çıkış biçimi görüntülenir:

```console
/people    [get|post]

https://localhost:5001/people~
```

## <a name="customize-the-http-repl"></a>HTTP REPL 'ı özelleştirme

HTTP REPL 'un varsayılan [renkleri](#set-color-preferences) özelleştirilebilir. Ayrıca, [varsayılan bir metin Düzenleyicisi](#set-the-default-text-editor) tanımlanabilir. HTTP REPL tercihleri geçerli oturum genelinde kalıcı hale getirilir ve gelecekteki oturumlarda kabul edilir. Değiştirildikten sonra, Tercihler aşağıdaki dosyada depolanır:

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

*% GIRIŞ%/. httpreplprefs*

# <a name="macostabmacos"></a>[macOS](#tab/macos)

*% GIRIŞ%/. httpreplprefs*

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

*% USERPROFILE%\\. httpreplprefs*

---

*. Httpreplprefs* dosyası başlangıçta yüklendi ve çalışma zamanında değişiklikler için izlenmiyor. Dosyada el ile yapılan değişiklikler yalnızca araç yeniden başlatıldıktan sonra devreye girer.

### <a name="view-the-settings"></a>Ayarları görüntüleyin

Kullanılabilir ayarları görüntülemek için `pref get` komutunu çalıştırın. Örneğin:

```console
https://localhost:5001/~ pref get
```

Yukarıdaki komut, kullanılabilir anahtar-değer çiftlerini görüntüler:

```console
colors.json=Green
colors.json.arrayBrace=BoldCyan
colors.json.comma=BoldYellow
colors.json.name=BoldMagenta
colors.json.nameSeparator=BoldWhite
colors.json.objectBrace=Cyan
colors.protocol=BoldGreen
colors.status=BoldYellow
```

### <a name="set-color-preferences"></a>Renk tercihlerini ayarla

Yanıt renklendirme Şu anda yalnızca JSON için destekleniyor. Varsayılan HTTP REPL aracı renklendirmesini özelleştirmek için, değiştirilecek renge karşılık gelen anahtarı bulun. Anahtarları bulma hakkında yönergeler için bkz. [ayarları görüntüleme](#view-the-settings) bölümü. Örneğin, `Green` `colors.json` anahtar değerini aşağıdaki gibi `White` olarak değiştirin:

```console
https://localhost:5001/people~ pref set colors.json White
```

Yalnızca [izin verilen renkler](https://github.com/dotnet/HttpRepl/blob/01d5c3c3373e98fe566ff5ef8a17c571de880293/src/Microsoft.Repl/ConsoleHandling/AllowedColors.cs) kullanılabilir. Sonraki HTTP istekleri, yeni renklendirmesi ile çıktıyı görüntüler.

Belirli renk anahtarları ayarlanmamışsa, daha genel anahtarlar kabul edilir. Bu geri dönüş davranışını göstermek için aşağıdaki örneği göz önünde bulundurun:

* `colors.json.name` bir değere sahip değilse, `colors.json.string` kullanılır.
* `colors.json.string` bir değere sahip değilse, `colors.json.literal` kullanılır.
* `colors.json.literal` bir değere sahip değilse, `colors.json` kullanılır. 
* `colors.json` bir değere sahip değilse, komut kabuğun varsayılan metin rengi (`AllowedColors.None`) kullanılır.

### <a name="set-indentation-size"></a>Girinti boyutunu ayarla

Yanıt girintileme boyut özelleştirmesi Şu anda yalnızca JSON için destekleniyor. Varsayılan boyut iki boşluklardan oluşamaz. Örneğin:

```json
[
  {
    "id": 1,
    "name": "Apple"
  },
  {
    "id": 2,
    "name": "Orange"
  },
  {
    "id": 3,
    "name": "Strawberry"
  }
]
```

Varsayılan boyutu değiştirmek için `formatting.json.indentSize` anahtarını ayarlayın. Örneğin, her zaman dört boşluk kullanmak için:

```console
pref set formatting.json.indentSize 4
```

Sonraki yanıtlar dört boşluk ayarına uyar:

```json
[
    {
        "id": 1,
        "name": "Apple"
    },
    {
        "id": 2,
        "name": "Orange"
    },
    {
        "id": 3,
        "name": "Strawberry"
    }
]
```

### <a name="set-the-default-text-editor"></a>Varsayılan metin düzenleyiciyi ayarlama

Varsayılan olarak, HTTP REPL 'un kullanılmak üzere yapılandırılmış metin Düzenleyicisi yok. HTTP istek gövdesi gerektiren Web API yöntemlerini test etmek için varsayılan metin Düzenleyicisi ayarlanmalıdır. HTTP REPL Aracı, istek gövdesini oluşturma amacıyla yapılandırılmış metin düzenleyicisini başlatır. Tercih ettiğiniz metin düzenleyiciyi varsayılan olarak ayarlamak için aşağıdaki komutu çalıştırın:

```console
pref set editor.command.default "<EXECUTABLE>"
```

Yukarıdaki komutta, `<EXECUTABLE>` metin düzenleyicisinin yürütülebilir dosyasının tam yoludur. Örneğin, Visual Studio Code varsayılan metin düzenleyicisi olarak ayarlamak için aşağıdaki komutu çalıştırın:

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

```console
pref set editor.command.default "/usr/bin/code"
```

# <a name="macostabmacos"></a>[macOS](#tab/macos)

```console
pref set editor.command.default "/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code"
```

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

```console
pref set editor.command.default "C:\Program Files\Microsoft VS Code\Code.exe"
```

---

Varsayılan metin düzenleyiciyi belirli CLı bağımsız değişkenleriyle başlatmak için `editor.command.default.arguments` anahtarını ayarlayın. Örneğin, Visual Studio Code varsayılan metin Düzenleyicisi olduğunu ve her zaman HTTP REPL 'un uzantıları devre dışı bırakılmış yeni bir oturumda Visual Studio Code açmasını istediğinizi varsayalım. Şu komutu çalıştırın:

```console
pref set editor.command.default.arguments "--disable-extensions --new-window"
```

### <a name="set-the-swagger-search-paths"></a>Swagger arama yollarını ayarla

Varsayılan olarak, HTTP REPL, `--swagger` seçeneği olmadan `connect` komutunu yürütürken Swagger belgesini bulmak için kullandığı bir göreli yollar kümesine sahiptir. Bu göreli yollar, `connect` komutunda belirtilen kök ve taban yollarla birleştirilir. Varsayılan göreli yollar şunlardır:

- *Swagger. JSON*
- *Swagger/v1/Swagger. JSON*
- */swagger.json*
- */swagger/v1/swagger.json*

Ortamınızda farklı bir arama yolları kümesi kullanmak için `swagger.searchPaths` tercihini ayarlayın. Değer, göreli yolların kanal ile ayrılmış bir listesi olmalıdır. Örneğin:

```console
pref set swagger.searchPaths "swagger/v2/swagger.json|swagger/v3/swagger.json"
```

## <a name="test-http-get-requests"></a>HTTP GET isteklerini test etme

### <a name="synopsis"></a>Özeti

```console
get <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Varsa, ilişkili denetleyici eylem yöntemi tarafından beklenen rota parametresi.

### <a name="options"></a>Seçenekler

`get` komutu için aşağıdaki seçenekler kullanılabilir:

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a>Örnek

HTTP GET isteği vermek için:

1. `get` komutunu bunu destekleyen bir uç noktada çalıştırın:

    ```console
    https://localhost:5001/people~ get
    ```

    Yukarıdaki komut aşağıdaki çıkış biçimini görüntüler:

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 03:38:45 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "name": "Scott Hunter"
      },
      {
        "id": 2,
        "name": "Scott Hanselman"
      },
      {
        "id": 3,
        "name": "Scott Guthrie"
      }
    ]


    https://localhost:5001/people~
    ```

1. `get` komutuna bir parametre geçirerek belirli bir kaydı alın:

    ```console
    https://localhost:5001/people~ get 2
    ```

    Yukarıdaki komut aşağıdaki çıkış biçimini görüntüler:

    ```console
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 21 Jun 2019 06:17:57 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 2,
        "name": "Scott Hanselman"
      }
    ]


    https://localhost:5001/people~
    ```

## <a name="test-http-post-requests"></a>HTTP POST isteklerini test et

### <a name="synopsis"></a>Özeti

```console
post <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Varsa, ilişkili denetleyici eylem yöntemi tarafından beklenen rota parametresi.

### <a name="options"></a>Seçenekler

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a>Örnek

HTTP POST isteği vermek için:

1. `post` komutunu bunu destekleyen bir uç noktada çalıştırın:

    ```console
    https://localhost:5001/people~ post -h Content-Type=application/json
    ```

    Önceki komutta, `Content-Type` HTTP istek üst bilgisi bir istek gövdesi medya türü olan JSON belirten şekilde ayarlanır. Varsayılan metin Düzenleyicisi, HTTP istek gövdesini temsil eden bir JSON şablonuyla bir *. tmp* dosyası açar. Örneğin:

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > Varsayılan metin düzenleyicisini ayarlamak için [varsayılan metin düzenleyiciyi ayarlama](#set-the-default-text-editor) bölümüne bakın.

1. JSON şablonunu model doğrulama gereksinimlerini karşılayacak şekilde değiştirin:

    ```json
    {
      "id": 0,
      "name": "Scott Addie"
    }
    ```

1. *. Tmp* dosyasını kaydedin ve metin düzenleyicisini kapatın. Aşağıdaki çıktı komut kabuğu 'nda görünür:

    ```console
    HTTP/1.1 201 Created
    Content-Type: application/json; charset=utf-8
    Date: Thu, 27 Jun 2019 21:24:18 GMT
    Location: https://localhost:5001/people/4
    Server: Kestrel
    Transfer-Encoding: chunked

    {
      "id": 4,
      "name": "Scott Addie"
    }


    https://localhost:5001/people~
    ```

## <a name="test-http-put-requests"></a>HTTP PUT isteklerini test etme

### <a name="synopsis"></a>Özeti

```console
put <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Varsa, ilişkili denetleyici eylem yöntemi tarafından beklenen rota parametresi.

### <a name="options"></a>Seçenekler

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

### <a name="example"></a>Örnek

HTTP PUT isteği vermek için:

1. *Isteğe bağlı*: verileri değiştirmeden önce görüntülemek için `get` komutunu çalıştırın:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]
    ```

1. `put` komutunu bunu destekleyen bir uç noktada çalıştırın:

    ```console
    https://localhost:5001/fruits~ put 2 -h Content-Type=application/json
    ```

    Önceki komutta, `Content-Type` HTTP istek üst bilgisi bir istek gövdesi medya türü olan JSON belirten şekilde ayarlanır. Varsayılan metin Düzenleyicisi, HTTP istek gövdesini temsil eden bir JSON şablonuyla bir *. tmp* dosyası açar. Örneğin:

    ```json
    {
      "id": 0,
      "name": ""
    }
    ```

    > [!TIP]
    > Varsayılan metin düzenleyicisini ayarlamak için [varsayılan metin düzenleyiciyi ayarlama](#set-the-default-text-editor) bölümüne bakın.

1. JSON şablonunu model doğrulama gereksinimlerini karşılayacak şekilde değiştirin:

    ```json
    {
      "id": 2,
      "name": "Cherry"
    }
    ```

1. *. Tmp* dosyasını kaydedin ve metin düzenleyicisini kapatın. Aşağıdaki çıktı komut kabuğu 'nda görünür:

    ```console
    [main 2019-06-28T17:27:01.805Z] update#setState idle
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:28:21 GMT
    Server: Kestrel
    ```

1. *Isteğe bağlı*: değişiklikleri görmek için bir `get` komutu verin. Örneğin, metin düzenleyicisinde "Chraz" yazdıysanız, `get` aşağıdakileri döndürür:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:08:20 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Cherry"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits~
    ```

## <a name="test-http-delete-requests"></a>HTTP SILME isteklerini test etme

### <a name="synopsis"></a>Özeti

```console
delete <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Varsa, ilişkili denetleyici eylem yöntemi tarafından beklenen rota parametresi.

### <a name="options"></a>Seçenekler

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

### <a name="example"></a>Örnek

HTTP SILME isteği vermek için:

1. *Isteğe bağlı*: verileri değiştirmeden önce görüntülemek için `get` komutunu çalıştırın:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:07:32 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 2,
        "data": "Orange"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]
    ```

1. `delete` komutunu bunu destekleyen bir uç noktada çalıştırın:

    ```console
    https://localhost:5001/fruits~ delete 2
    ```

    Yukarıdaki komut aşağıdaki çıkış biçimini görüntüler:

    ```console
    HTTP/1.1 204 No Content
    Date: Fri, 28 Jun 2019 17:36:42 GMT
    Server: Kestrel
    ```

1. *Isteğe bağlı*: değişiklikleri görmek için bir `get` komutu verin. Bu örnekte, bir `get` aşağıdakini döndürür:

    ```console
    https://localhost:5001/fruits~ get
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Sat, 22 Jun 2019 00:16:30 GMT
    Server: Kestrel
    Transfer-Encoding: chunked

    [
      {
        "id": 1,
        "data": "Apple"
      },
      {
        "id": 3,
        "data": "Strawberry"
      }
    ]


    https://localhost:5001/fruits~
    ```

## <a name="test-http-patch-requests"></a>HTTP PATCH isteklerini test etme

### <a name="synopsis"></a>Özeti

```console
patch <PARAMETER> [-c|--content] [-f|--file] [-h|--header] [--no-body] [-F|--no-formatting] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Varsa, ilişkili denetleyici eylem yöntemi tarafından beklenen rota parametresi.

### <a name="options"></a>Seçenekler

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

[!INCLUDE [HTTP request body CLI options](~/includes/http-repl/requires-body-options.md)]

## <a name="test-http-head-requests"></a>HTTP HEAD isteklerini test etme

### <a name="synopsis"></a>Özeti

```console
head <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Varsa, ilişkili denetleyici eylem yöntemi tarafından beklenen rota parametresi.

### <a name="options"></a>Seçenekler

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="test-http-options-requests"></a>Sınama HTTP SEÇENEKLERI istekleri

### <a name="synopsis"></a>Özeti

```console
options <PARAMETER> [-F|--no-formatting] [-h|--header] [--response] [--response:body] [--response:headers] [-s|--streaming]
```

### <a name="arguments"></a>Arguments

`PARAMETER`

Varsa, ilişkili denetleyici eylem yöntemi tarafından beklenen rota parametresi.

### <a name="options"></a>Seçenekler

[!INCLUDE [standard CLI options](~/includes/http-repl/standard-options.md)]

## <a name="set-http-request-headers"></a>HTTP istek üst bilgilerini ayarla

Bir HTTP istek üst bilgisi ayarlamak için aşağıdaki yaklaşımlardan birini kullanın:

* HTTP isteğiyle satır içi ayarlayın. Örneğin:

    ```console
    https://localhost:5001/people~ post -h Content-Type=application/json
    ```
    
    Önceki yaklaşımla, her ayrı HTTP istek üst bilgisi kendi `-h` seçeneğini gerektirir.

* HTTP isteğini göndermeden önce ayarlayın. Örneğin:

    ```console
    https://localhost:5001/people~ set header Content-Type application/json
    ```
    
    Bir isteği göndermeden önce üst bilgi ayarlanırken üst bilgi, komut kabuğu oturumunun süresi boyunca ayarlanmış olarak kalır. Üstbilgiyi temizlemek için boş bir değer sağlayın. Örneğin:
    
    ```console
    https://localhost:5001/people~ set header Content-Type
    ```

## <a name="test-secured-endpoints"></a>Güvenli uç noktaları test et

HTTP REPL, HTTP istek üst bilgilerinin kullanımı aracılığıyla güvenli uç noktaların sınamasını destekler. Desteklenen kimlik doğrulama ve yetkilendirme düzenlerine örnek olarak temel kimlik doğrulaması, JWT taşıyıcı belirteçleri ve Özet kimlik doğrulaması verilebilir. Örneğin, aşağıdaki komutla bir uç noktaya bir taşıyıcı belirteci gönderebilirsiniz:

```console
set header Authorization "bearer <TOKEN VALUE>"
```

Azure 'da barındırılan bir uç noktaya erişmek veya [azure REST API](/rest/api/azure/)kullanmak için bir taşıyıcı belirtecine ihtiyacınız vardır. Azure [CLI](/cli/azure/)aracılığıyla Azure Aboneliğinize yönelik bir taşıyıcı belirteci almak için aşağıdaki adımları kullanın. HTTP REPL, bir HTTP istek üstbilgisindeki taşıyıcı belirtecini ayarlar ve Azure App Service Web Apps listesini alır.

1. Azure 'da oturum açın:

    ```azcli
    az login
    ```

1. Aşağıdaki komutla abonelik KIMLIĞINIZI alın:

    ```azcli
    az account show --query id
    ```

1. Abonelik KIMLIĞINIZI kopyalayın ve şu komutu çalıştırın:

    ```azcli
    az account set --subscription "<SUBSCRIPTION ID>"
    ```

1. Aşağıdaki komutla taşıyıcı belirtecinizi alın:

    ```azcli
    az account get-access-token --query accessToken
    ```

1. HTTP REPL aracılığıyla Azure REST API bağlanma:

    ```console
    httprepl https://management.azure.com
    ```

1. `Authorization` HTTP istek üst bilgisini ayarlayın:

    ```console
    https://management.azure.com/> set header Authorization "bearer <ACCESS TOKEN>"
    ```

1. Aboneliğe gidin:

    ```console
    https://management.azure.com/> cd subscriptions/<SUBSCRIPTION ID>
    ```

1. Aboneliğinizin Azure App Service Web Apps bir listesini alın:

    ```console
    https://management.azure.com/subscriptions/{SUBSCRIPTION ID}> get providers/Microsoft.Web/sites?api-version=2016-08-01
    ```

    Aşağıdaki yanıt görüntülenir:

    ```console
    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Content-Length: 35948
    Content-Type: application/json; charset=utf-8
    Date: Thu, 19 Sep 2019 23:04:03 GMT
    Expires: -1
    Pragma: no-cache
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    X-Content-Type-Options: nosniff
    x-ms-correlation-request-id: <em>xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</em>
    x-ms-original-request-ids: <em>xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx;xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</em>
    x-ms-ratelimit-remaining-subscription-reads: 11999
    x-ms-request-id: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    x-ms-routing-request-id: WESTUS:xxxxxxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx
    {
      "value": [
        <AZURE RESOURCES LIST>
      ]
    }
    ```

## <a name="toggle-http-request-display"></a>HTTP istek görüntüsünü değiştirme

Varsayılan olarak, gönderilmekte olan HTTP isteğinin görüntüsü bastırılır. Komut kabuğu oturumunun süresi boyunca ilgili ayarı değiştirmek mümkündür.

### <a name="enable-request-display"></a>İstek görüntülemesini etkinleştir

`echo on` komutunu çalıştırarak gönderilmekte olan HTTP isteğini görüntüleyin. Örneğin:

```console
https://localhost:5001/people~ echo on
Request echoing is on
```

Geçerli oturumdaki sonraki HTTP istekleri, istek üst bilgilerini görüntüler. Örneğin:

```console
https://localhost:5001/people~ post

[main 2019-06-28T18:50:11.930Z] update#setState idle
Request to https://localhost:5001...

POST /people HTTP/1.1
Content-Length: 41
Content-Type: application/json
User-Agent: HTTP-REPL

{
  "id": 0,
  "name": "Scott Addie"
}

Response from https://localhost:5001...

HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Fri, 28 Jun 2019 18:50:21 GMT
Location: https://localhost:5001/people/4
Server: Kestrel
Transfer-Encoding: chunked

{
  "id": 4,
  "name": "Scott Addie"
}


https://localhost:5001/people~
```

### <a name="disable-request-display"></a>İstek görüntüsünü devre dışı bırak

`echo off` komutu çalıştırılarak gönderilmekte olan HTTP isteğinin görüntülenmesini gizleyin. Örneğin:

```console
https://localhost:5001/people~ echo off
Request echoing is off
```

## <a name="run-a-script"></a>Betik çalıştır

Aynı HTTP REPL komutları kümesini sıklıkla yürütüyorsanız bunları bir metin dosyasında depolamayı göz önünde bulundurun. Dosyadaki komutlar, komut satırında el ile çalıştıranlarla aynı formu alır. Komutlar, `run` komutu kullanılarak toplanmış bir biçimde yürütülebilir. Örneğin:

1. Yeni satır için ayrılmış komutlar kümesini içeren bir metin dosyası oluşturun. Göstermek için aşağıdaki komutları içeren bir *People-Script. txt* dosyası düşünün:

    ```text
    set base https://localhost:5001
    ls
    cd People
    ls
    get 1
    ```

1. Metin dosyasının yolunu geçirerek `run` komutunu yürütün. Örneğin:

    ```console
    https://localhost:5001/~ run C:\http-repl-scripts\people-script.txt
    ```

    Şu çıktı görünür:

    ```console
    https://localhost:5001/~ set base https://localhost:5001
    Using swagger metadata from https://localhost:5001/swagger/v1/swagger.json
    
    https://localhost:5001/~ ls
    .        []
    Fruits   [get|post]
    People   [get|post]
    
    https://localhost:5001/~ cd People
    /People    [get|post]
    
    https://localhost:5001/People~ ls
    .      [get|post]
    ..     []
    {id}   [get|put|delete]
    
    https://localhost:5001/People~ get 1
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Fri, 12 Jul 2019 19:20:10 GMT
    Server: Kestrel
    Transfer-Encoding: chunked
    
    {
      "id": 1,
      "name": "Scott Hunter"
    }
    
    
    https://localhost:5001/People~
    ```

## <a name="clear-the-output"></a>Çıktıyı temizle

HTTP REPL aracı tarafından komut kabuğu 'na yazılan tüm çıktıyı kaldırmak için `clear` veya `cls` komutunu çalıştırın. Göstermek için komut kabuğu 'nun aşağıdaki çıktıyı içerdiğini düşünün:

```console
httprepl https://localhost:5001
(Disconnected)~ set base "https://localhost:5001"
Using swagger metadata from https://localhost:5001/swagger/v1/swagger.json

https://localhost:5001/~ ls
.        []
Fruits   [get|post]
People   [get|post]

https://localhost:5001/~
```

Çıktıyı temizlemek için aşağıdaki komutu çalıştırın:

```console
https://localhost:5001/~ clear
```

Yukarıdaki komutu çalıştırdıktan sonra, komut kabuğu yalnızca şu çıktıyı içerir:

```console
https://localhost:5001/~
```

## <a name="additional-resources"></a>Ek kaynaklar

* [REST API istekleri](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#74-supported-methods)
* [HTTP REPL GitHub deposu](https://github.com/dotnet/HttpRepl)
