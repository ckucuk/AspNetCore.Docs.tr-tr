---
title: ASP.NET Core içinde URL yeniden yazma ara yazılımı
author: guardrex
description: ASP.NET Core uygulamalarında URL yeniden yazma ve URL yeniden yazma ara yazılımı ile yeniden yönlendirme hakkında bilgi edinin.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/16/2019
uid: fundamentals/url-rewriting
ms.openlocfilehash: e284d2172af723bb80a7be9f6e6f1a87ebe5208e
ms.sourcegitcommit: 41f2c1a6b316e6e368a4fd27a8b18d157cef91e1
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 08/21/2019
ms.locfileid: "69886502"
---
# <a name="url-rewriting-middleware-in-aspnet-core"></a>ASP.NET Core içinde URL yeniden yazma ara yazılımı

, [Luke Latham](https://github.com/guardrex) ve [MIKAEL Mengistu](https://github.com/mikaelm12) tarafından

::: moniker range=">= aspnetcore-3.0"

Bu belgede, ASP.NET Core uygulamalarında URL yeniden yazma ara yazılımı kullanma yönergeleriyle birlikte URL yeniden yazma tanıtılmaktadır.

URL yeniden yazma, istek URL 'Lerini bir veya daha fazla önceden tanımlanmış kurala göre değiştirme işlemidir. URL yeniden yazma, konumların ve adreslerin sıkı bir şekilde bağlanmaması için kaynak konumları ve adresleri arasında bir soyutlama oluşturur. URL yeniden yazma işlemi birkaç senaryoda yararlı olur:

* Sunucu kaynaklarını geçici olarak veya kalıcı olarak taşıyın veya değiştirin ve bu kaynakların kararlı konum belirleyicilerinin bakımını yapın.
* İstek işlemeyi farklı uygulamalar arasında veya bir uygulamanın alanlarında bölme.
* Gelen isteklerde URL segmentlerini kaldırın, ekleyin veya yeniden düzenleyin.
* Arama motoru Iyileştirmesi (SEO) için genel URL 'Leri iyileştirin.
* Ziyaretçilerin bir kaynak isteyerek döndürülen içeriği tahmin etmeye yardımcı olmak için kolay genel URL 'Lerin kullanılmasına izin verme.
* Güvensiz istekleri güvenli uç noktalara yönlendirin.
* Bir dış sitenin varlığı kendi içeriğine bağlayarak başka bir sitede barındırılan statik bir varlık kullandığı Hotlink 'i engelleyin.

> [!NOTE]
> URL yeniden yazma, bir uygulamanın performansını azaltabilir. Uygun yerlerde kuralların sayısını ve karmaşıklığını sınırlayın.

[Görüntüleme veya indirme örnek kodu](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/) ([nasıl indirileceğini](xref:index#how-to-download-a-sample))

## <a name="url-redirect-and-url-rewrite"></a>URL yeniden yönlendirme ve URL yeniden yazma

*URL yeniden yönlendirme* ve *URL yeniden yazma* arasındaki ifade farkı daha hafif ancak istemcilere kaynak sağlamak için önemli etkileri vardır. ASP.NET Core URL yeniden yazma ara yazılımı her ikisine de ihtiyacı verebilir.

*URL yeniden yönlendirme* , istemcinin, ilk olarak istenen istemciden farklı bir adresteki kaynağa erişmesi için bir istemci tarafı işlemi içerir. Bu, sunucuya gidiş dönüş gerektirir. İstemci kaynak için yeni bir istek yaptığında, istemciye döndürülen yeniden yönlendirme URL 'SI tarayıcının adres çubuğunda görüntülenir.

`/different-resource` Öğesine `/resource` yönlendiriliyorsasunucu,istemcinin,yenidenyönlendirmeningeçiciyadakalıcıolduğunubelirtenbirdurumkoduilekaynağıeldeetmesigerektiğiniyanıtverir.`/different-resource`

![Bir WebAPI hizmeti uç noktası, sürüm 1 ' den (v1) sunucudaki sürüm 2 ' ye (v2) geçici olarak değiştirildi. İstemci, Service 1 yolundaki hizmet için bir istek yapar/v1/api. Sunucu, Service 2/v2/apikonumundaki hizmetin yeni, geçici yoluyla bir 302 (bulunan) yanıtını geri gönderir. İstemci, yeniden yönlendirme URL 'sindeki hizmete ikinci bir istek yapar. Sunucu, 200 (Tamam) durum kodu ile yanıt verir.](url-rewriting/_static/url_redirect.png)

İstekleri farklı bir URL 'ye yönlendirirken, yanıtla birlikte durum kodunu belirterek yeniden yönlendirmenin kalıcı mi yoksa geçici mi olduğunu belirtin:

* *301-taşınan kalıcı* durum kodu, kaynağın yeni, kalıcı bir URL olduğu ve istemciye, gelecekteki tüm isteklerin kaynak için tüm istekleri yeni URL 'yi kullanması gerektiğini bildirmek istediğinizde kullanılır. *İstemci, 301 durum kodu alındığında yanıtı önbelleğe alabilir ve yeniden kullanabilir.*

* *302-bulunan* durum kodu, yeniden yönlendirmenin geçici ya da genellikle değişikliğe tabi olduğu durumlarda kullanılır. 302 durum kodu, istemcinin URL 'YI depolayamadığını ve gelecekte kullanacağını gösterir.

Durum kodları hakkında daha fazla bilgi için bkz [. RFC 2616: Durum kodu tanımları](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).

*URL yeniden yazma* , istenen istemciden farklı bir kaynak adresinden kaynak sağlayan sunucu tarafı bir işlemdir. URL yeniden yazma, sunucuya gidiş dönüş gerektirmez. Yeniden yazan URL istemciye döndürülmüyor ve tarayıcının adres çubuğunda görünmüyor.

`/different-resource`Öğesine `/resource` geridönerse,sunucu,kaynağınıdahili`/different-resource`olarak getirir ve döndürür.

İstemci, yeniden yazan URL 'de kaynağı alabiliyor olsa da, istemci isteği yaptığında ve yanıtı aldığında kaynağın yeniden yazan URL 'de bulunduğunu bilgilendirmez.

![Sunucu üzerinde sürüm 1 ' den (v1) sürüm 2 ' ye (v2) bir WebAPI hizmet uç noktası değiştirilmiştir. İstemci, Service 1 yolundaki hizmet için bir istek yapar/v1/api. İstek URL 'SI, hizmet 2 yolu/v2/api'daki hizmete erişmek için yeniden yazıldı. Hizmet, istemciye 200 (Tamam) durum kodu ile yanıt verir.](url-rewriting/_static/url_rewrite.png)

## <a name="url-rewriting-sample-app"></a>URL yeniden yazma örnek uygulaması

[Örnek uygulamayla](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/)birlikte YENIDEN yazma URL 'sinin özelliklerini inceleyebilirsiniz. Uygulama yeniden yönlendirme ve yeniden yazma kuralları uygular ve birkaç senaryo için yeniden yönlendirilen veya yeniden yazan URL 'YI gösterir.

## <a name="when-to-use-url-rewriting-middleware"></a>URL yeniden yazma ara yazılımı ne zaman kullanılır

Aşağıdaki yaklaşımlardan birini kullandığımmdan URL yeniden yazma ara yazılımı kullanın:

* [Windows Server 'da IIS ile URL yeniden yazma modülü](https://www.iis.net/downloads/microsoft/url-rewrite)
* [Apache Server 'da Apache mod_rewrite modülü](https://httpd.apache.org/docs/2.4/rewrite/)
* [NGINX üzerinde URL yeniden yazma](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)

Ayrıca, uygulama [http. sys sunucusunda](xref:fundamentals/servers/httpsys) barındırıldığı zaman ara yazılımı kullanın (eski adıyla webListener olarak adlandırılır).

IIS, Apache ve NGINX 'te sunucu tabanlı URL yeniden yazma teknolojilerini kullanmanın başlıca nedenleri şunlardır:

* Ara yazılım bu modüllerin tüm özelliklerini desteklemez.

  Sunucu modüllerinin bazı özellikleri, `IsFile` IIS yeniden yazma modülünün ve `IsDirectory` kısıtlamaları gibi ASP.NET Core projelerle birlikte çalışmaz. Bu senaryolarda, bunun yerine ara yazılımı kullanın.
* Ara yazılım performansı büyük olasılıkla modüllerle eşleşmiyor.

  Sınama, performansı en iyi şekilde düşürür veya performans düşüklüğü göz ardı edilebilir olduğundan emin olmanın tek yoludur.

## <a name="package"></a>Paket

URL yeniden yazma ara yazılımı, ASP.NET Core uygulamalarında örtük olarak bulunan [Microsoft. AspNetCore. yeniden yazma](https://www.nuget.org/packages/Microsoft.AspNetCore.Rewrite) paketi tarafından sağlanır.

## <a name="extension-and-options"></a>Uzantı ve Seçenekler

Yeniden yazma kurallarınızın her biri için uzantı yöntemleriyle [Rewriteoptions](xref:Microsoft.AspNetCore.Rewrite.RewriteOptions) sınıfının bir ÖRNEĞINI oluşturarak URL yeniden yazma ve yeniden yönlendirme kuralları oluşturun. Birden çok kuralı, işlenmeyi istediğiniz sırada zincirle. , `RewriteOptions` İle istek ardışık düzenine eklendikçe, bu URL 'ye yeniden yazma ara yazılımı ile <xref:Microsoft.AspNetCore.Builder.RewriteBuilderExtensions.UseRewriter*>geçirilir:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1)]

### <a name="redirect-non-www-to-www"></a>Www olmayan www 'e yönlendirme

Üç seçenek, uygulamanın`www` `www`istek olmayan istekleri yeniden yönlendirmesine izin verir:

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWwwPermanent*>İstek değilse isteği alt `www` etki alanına kalıcı olarak yeniden yönlendirin.`www` &ndash; [Status308PermanentRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status308PermanentRedirect) durum kodu ile yeniden yönlendirir.

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWww*>Gelen istek değilse isteği `www` alt etki alanına yönlendirin.`www` &ndash; [Status307TemporaryRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status307TemporaryRedirect) durum kodu ile yeniden yönlendirir. Aşırı yükleme, yanıt için durum kodu sağlamanıza izin verir. Bir durum kodu ataması için <xref:Microsoft.AspNetCore.Http.StatusCodes> sınıfın bir alanını kullanın.

### <a name="url-redirect"></a>URL yeniden yönlendirme

İstekleri <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirect*> yeniden yönlendirmek için kullanın. İlk parametre gelen URL 'nin yolu ile eşleşen Regex içerir. İkinci parametre değiştirme dizesidir. Varsa, üçüncü parametre durum kodunu belirtir. Durum kodunu belirtmezseniz, durum kodu varsayılan olarak *302-bulunur*; bu da kaynağın geçici olarak taşındığını veya değiştirildiğini gösterir.

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=9)]

Geliştirici araçları etkinleştirilmiş bir tarayıcıda, örnek uygulamaya yol `/redirect-rule/1234/5678`ile bir istek oluşturun. Regex, üzerindeki `redirect-rule/(.*)`istek yoluyla eşleşir ve yol ile `/redirected/1234/5678`değiştirilmiştir. Yeniden yönlendirme URL 'SI, *302 tarafından bulunan* bir durum kodu ile istemciye geri gönderilir. Tarayıcı, tarayıcının adres çubuğunda görüntülenen yeniden yönlendirme URL 'SI üzerinde yeni bir istek oluşturur. Örnek uygulamadaki hiçbir kural, yeniden yönlendirme URL 'SI üzerinde eşleşmediğinden:

* İkinci istek uygulamadan *200-Tamam* yanıtı alır.
* Yanıtın gövdesi, yeniden yönlendirme URL 'sini gösterir.

Bir URL *yeniden yönlendirildiğinde*sunucuya gidiş dönüş yapılır.

> [!WARNING]
> Yeniden yönlendirme kuralları oluştururken dikkatli olun. Yeniden yönlendirme kuralları, bir yeniden yönlendirmeden sonra dahil olmak üzere, uygulamaya yapılan her istekte değerlendirilir. Yanlışlıkla *sonsuz yeniden yönlendirmeler döngüsü*oluşturmak kolaydır.

Özgün Istek:`/redirect-rule/1234/5678`

![İstekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect.png)

Parantez içinde yer alan ifadenin kısmına bir *yakalama grubu*denir. İfadenin nokta (`.`), *herhangi bir karakterle eşleşir*anlamına gelir. Yıldız işareti (`*`) *, önceki karakterle sıfır veya daha fazla kez eşleşme*gösterir. Bu nedenle, URL `1234/5678`'nin son iki yol kesimi, yakalama grubu `(.*)`tarafından yakalanır. Sonrasında istek URL 'sinde sağladığınız herhangi bir değer bu `redirect-rule/` tek yakalama grubu tarafından yakalandıktan sonra.

Değiştirme dizesinde, yakalanan gruplar, dolar işareti (`$`) ve ardından yakalamanın sıra numarası ile birlikte dizeye eklenir. İlk yakalama grubu değeri ile `$1`elde edilir, ikincisi ile `$2`ve normal Regex yakalama grupları için sırayla devam eder. Örnek uygulamadaki yeniden yönlendirme kuralı Regex bölümünde yalnızca bir tane yakalanan grup bulunur, bu `$1`nedenle değiştirme dizesinde yalnızca bir tane eklenmiş grup vardır. Kural uygulandığında, URL olur `/redirected/1234/5678`.

### <a name="url-redirect-to-a-secure-endpoint"></a>Güvenli bir uç noktaya URL yönlendirmesi

HTTP <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttps*> isteklerini https protokolünü kullanarak aynı konağa ve yola yeniden yönlendirmek için kullanın. Durum kodu sağlanmazsa, ara yazılım varsayılan olarak *302-bulunur*. Bağlantı noktası sağlanmazsa:

* Ara yazılım varsayılan olarak `null`olur.
* Şema (https Protokolü `https` ) olarak değişir ve istemci, 443 numaralı bağlantı noktasında kaynağa erişir.

Aşağıdaki örnek, durum kodunun *301-kalıcı olarak taşınacağını* ve bağlantı noktasını 5001 olarak nasıl değiştirileceğini gösterir.

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttps(301, 5001);

    app.UseRewriter(options);
}
```

Güvenli <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttpsPermanent*> olmayan istekleri, bağlantı noktası 443 üzerinde güvenli https Protokolü ile aynı konağa ve yola yeniden yönlendirmek için kullanın. Ara yazılım durum kodunu 301 olarak ayarlar ve *kalıcı olarak taşınır*.

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttpsPermanent();

    app.UseRewriter(options);
}
```

> [!NOTE]
> Ek yeniden yönlendirme kuralları gereksinimi olmadan güvenli bir uç noktaya yönlendirilirken, HTTPS yeniden yönlendirme ara yazılımı kullanmanızı öneririz. Daha fazla bilgi için bkz. [https 'Yi zorla](xref:security/enforcing-ssl#require-https) konusu.

Örnek uygulama, veya `AddRedirectToHttps` `AddRedirectToHttpsPermanent`kullanımını gösterme yeteneğine sahiptir. Uzantı yöntemini öğesine `RewriteOptions`ekleyin. Herhangi bir URL 'de uygulamaya güvenli olmayan bir istek oluşturun. Otomatik olarak imzalanan sertifikanın güvenilmeyen tarayıcı güvenlik uyarısını kapatın veya sertifikaya güvenmek için bir özel durum oluşturun.

Kullanarak `AddRedirectToHttps(301, 5001)`özgün istek:`http://localhost:5000/secure`

![İstekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect_to_https.png)

Kullanarak `AddRedirectToHttpsPermanent`özgün istek:`http://localhost:5000/secure`

![İstekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect_to_https_permanent.png)

### <a name="url-rewrite"></a>URL yeniden yazma

URL <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRewrite*> 'leri yeniden yazma kuralı oluşturmak için kullanın. İlk parametre gelen URL yolundaki eşleşme için Regex içerir. İkinci parametre değiştirme dizesidir. Üçüncü parametresi `skipRemainingRules: {true|false}`, geçerli kural uygulanmışsa ek yeniden yazma kurallarının atlanıp atlanmayacağını gösterir.

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=10-11)]

Özgün Istek:`/rewrite-rule/1234/5678`

![İsteği ve yanıtı izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_rewrite.png)

İfadenin başındaki simgeyi seçtiğinizde`^`(), eşleşmesinin URL yolunun başlangıcında başladığı anlamına gelir.

Önceki örnekte, yeniden yönlendirme kuralıyla `redirect-rule/(.*)`, Regex başlangıcında simgeyi seçtiğinizde (`^`) yoktur. Bu nedenle, başarılı bir eşleşme `redirect-rule/` için herhangi bir karakterden önce yol açabilir.

| Yol                               | Eşleştirme |
| ---------------------------------- | :---: |
| `/redirect-rule/1234/5678`         | Evet   |
| `/my-cool-redirect-rule/1234/5678` | Evet   |
| `/anotherredirect-rule/1234/5678`  | Evet   |

Yeniden yazma kuralı `^rewrite-rule/(\d+)/(\d+)`, yalnızca ile `rewrite-rule/`başlarsa yollarla eşleşir. Aşağıdaki tabloda, eşleşen farkı aklınızda.

| Yol                              | Eşleştirme |
| --------------------------------- | :---: |
| `/rewrite-rule/1234/5678`         | Evet   |
| `/my-cool-rewrite-rule/1234/5678` | Hayır    |
| `/anotherrewrite-rule/1234/5678`  | Hayır    |

İfadenin bölümünü takip eden iki yakalama `(\d+)/(\d+)`grubu vardır. `^rewrite-rule/` Belirtir bir *sayıyla (sayı) eşleşir.* `\d` Artı işareti (`+`) *bir veya daha fazla önceki karakterden eşleşiyor*demektir. Bu nedenle, URL bir sayı içermeli ve ardından İleri eğik çizgi ve ardından başka bir sayı içermelidir. Bu yakalama grupları, ve olarak `$1` yeniden `$2`yazan URL 'sine eklenir. Yeniden yazma kuralı değiştirme dizesi yakalanan grupları sorgu dizesine koyar. İstenen yolu `/rewrite-rule/1234/5678` , `/rewritten?var1=1234&var2=5678`kaynağı elde etmek için yeniden yazılır. Özgün istekte bir sorgu dizesi varsa, URL yeniden yazdığınızda korunur.

Kaynağı almak için sunucuya gidiş dönüş yok. Kaynak varsa, bu, alınır ve istemciye *200-ok* durum kodu ile döndürülür. İstemci yeniden yönlendirmediği için tarayıcının adres çubuğundaki URL değişmez. İstemciler, sunucuda bir URL yeniden yazma işleminin gerçekleştiğini algılayamaz.

> [!NOTE]
> Eşleşen `skipRemainingRules: true` kuralların hesaplama maliyeti ve uygulama yanıt süresini arttığı için mümkün olan her durumda kullanın. En hızlı uygulama yanıtı için:
>
> * En sık eşleşen kuraldan en az sıklıkta eşleşen kurala göre yeniden yazma kuralları.
> * Bir eşleşme gerçekleştiğinde ve ek kural işleme gerekli olmadığında kalan kuralların işlenmesini atlayın.

### <a name="apache-mod_rewrite"></a>Apache mod_rewrite

İle <xref:Microsoft.AspNetCore.Rewrite.ApacheModRewriteOptionsExtensions.AddApacheModRewrite*>Apache mod_rewrite kurallarını uygulayın. Kurallar dosyasının uygulamayla birlikte dağıtıldığından emin olun. Daha fazla bilgi ve mod_rewrite kuralları örnekleri için bkz. [Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/).

, ApacheModRewrite. txt kuralları dosyasındaki kuralları okumak için kullanılır: <xref:System.IO.StreamReader>

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=3-4,12)]

Örnek uygulama, istekleri ' den `/apache-mod-rules-redirect/(.\*)` ' `/redirected?id=$1`e yeniden yönlendirir. Yanıt durum kodu *302-bulundu*.

[!code[](url-rewriting/samples/3.x/SampleApp/ApacheModRewrite.txt)]

Özgün Istek:`/apache-mod-rules-redirect/1234`

![İstekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_apache_mod_redirect.png)

Ara yazılım aşağıdaki Apache mod_rewrite Server değişkenlerini destekler:

* CONN_REMOTE_ADDR
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_FORWARDED
* HTTP_HOST
* HTTP_REFERER
* HTTP_USER_AGENT
* HTTPS
* IPV6
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_METHOD
* REQUEST_SCHEME
* REQUEST_URI
* SCRIPT_FILENAME
* SERVER_ADDR
* SERVER_PORT
* SERVER_PROTOCOL
* TIME
* TIME_DAY
* TIME_HOUR
* TIME_MIN
* TIME_MON
* TIME_SEC
* TIME_WDAY
* TIME_YEAR

### <a name="iis-url-rewrite-module-rules"></a>IIS URL yeniden yazma modülü kuralları

IIS URL yeniden yazma modülü için geçerli olan kural kümesini kullanmak için kullanın <xref:Microsoft.AspNetCore.Rewrite.IISUrlRewriteOptionsExtensions.AddIISUrlRewrite*>. Kurallar dosyasının uygulamayla birlikte dağıtıldığından emin olun. Windows Server IIS 'de çalışırken uygulamanın *Web. config* dosyasını kullanmak için ara yazılımı yönlendirmeyin. IIS ile, IIS yeniden yazma modülüyle çakışmalardan kaçınmak için bu kuralların uygulamanın *Web. config* dosyası dışında depolanması gerekir. Daha fazla bilgi ve IIS URL yeniden yazma modülü kurallarının örnekleri için bkz. [Using URL yeniden yazma modülü 2,0](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20) ve [URL yeniden yazma modülü yapılandırma başvurusu](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference).

, Iisurlyeniden *yazma. xml* kuralları dosyasındaki kuralları okumak için kullanılır: <xref:System.IO.StreamReader>

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=5-6,13)]

Örnek uygulama, ' den ' `/iis-rules-rewrite/(.*)` e `/rewritten?id=$1`olan istekleri yeniden yazar. Yanıt, istemciye *200-ok* durum kodu ile gönderilir.

[!code-xml[](url-rewriting/samples/3.x/SampleApp/IISUrlRewrite.xml)]

Özgün Istek:`/iis-rules-rewrite/1234`

![İsteği ve yanıtı izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_iis_url_rewrite.png)

Uygulamanızı istenmeyen yollarla etkileyebilecek sunucu düzeyi kurallara sahip etkin bir IIS yeniden yazma modülünüzün olması halinde, bir uygulama için IIS yeniden yazma modülünü devre dışı bırakabilirsiniz. Daha fazla bilgi için bkz. [IIS modüllerini devre dışı bırakma](xref:host-and-deploy/iis/modules#disabling-iis-modules).

#### <a name="unsupported-features"></a>Desteklenmeyen özellikler

ASP.NET Core 2. x ile yayınlanan ara yazılım, aşağıdaki IIS URL yeniden yazma modülü özelliklerini desteklemez:

* Giden kuralları
* Özel sunucu değişkenleri
* Karakterlerini
* LogRewrittenUrl 'Si

#### <a name="supported-server-variables"></a>Desteklenen sunucu değişkenleri

Ara yazılım aşağıdaki IIS URL yeniden yazma modülü sunucu değişkenlerini destekler:

* CONTENT_LENGTH
* CONTENT_TYPE
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_HOST
* HTTP_REFERER
* HTTP_URL
* HTTP_USER_AGENT
* HTTPS
* LOCAL_ADDR
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_URI

> [!NOTE]
> Ayrıca bir <xref:Microsoft.Extensions.FileProviders.IFileProvider> <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider>ile elde edebilirsiniz. Bu yaklaşım, yeniden yazma kuralları dosyalarınızın konumu için daha fazla esneklik sağlayabilir. Yeniden yazma kuralları dosyalarınızın sağladığınız yoldaki sunucuya dağıtıldığından emin olun.
>
> ```csharp
> PhysicalFileProvider fileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory());
> ```

### <a name="method-based-rule"></a>Yöntem tabanlı kural

Bir <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> yöntemde kendi kural mantığınızı uygulamak için kullanın. `Add`, metodunda kullanım <xref:Microsoft.AspNetCore.Http.HttpContext> için kullanılabilir hale getiren öğesinigösterir.<xref:Microsoft.AspNetCore.Rewrite.RewriteContext> [Rewritecontext. Result](xref:Microsoft.AspNetCore.Rewrite.RewriteContext.Result*) , ek ardışık düzen işlemenin nasıl işlendiğini belirler. Değeri aşağıdaki tabloda açıklanan <xref:Microsoft.AspNetCore.Rewrite.RuleResult> alanlardan birine ayarlayın.

| `RewriteContext.Result`              | Eylem                                                           |
| ------------------------------------ | ---------------------------------------------------------------- |
| `RuleResult.ContinueRules`varsayılanını | Kuralları uygulamaya devam edin.                                         |
| `RuleResult.EndResponse`             | Kuralları uygulamayı durdurun ve yanıtı gönderin.                       |
| `RuleResult.SkipRemainingRules`      | Kuralları uygulamayı durdurun ve bağlamı bir sonraki ara yazılıma gönderin. |

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=14)]

Örnek uygulama, *. xml*ile biten yollar için istekleri yeniden yönlendiren bir yöntemi gösterir. İçin `/file.xml`bir istek yapılırsa, istek öğesine `/xmlfiles/file.xml`yeniden yönlendirilir. Durum kodu *301*olarak ayarlanır. Tarayıcı */xmlfiles/File.xml*için yeni bir istek yaptığında, statik dosya ara yazılımı dosyayı *Wwwroot/xmlfiles* klasöründen istemciye sunar. Yeniden yönlendirme için, yanıtın durum kodunu açık olarak ayarlayın. Aksi takdirde, *200-ok* durum kodu döndürülür ve yeniden yönlendirme istemcide gerçekleşmez.

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/RewriteRules.cs?name=snippet_RedirectXmlFileRequests&highlight=14-18)]

Bu yaklaşım ayrıca istekleri yeniden yazabilir. Örnek uygulama, *dosya. txt* metin dosyasına *Wwwroot* klasöründen hizmeti sağlamak için herhangi bir metin dosyası isteğinin yolunu yeniden yazmayı gösterir. Statik dosya ara yazılımı, güncelleştirilmiş istek yoluna göre dosyayı sunar:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=15,22)]

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/RewriteRules.cs?name=snippet_RewriteTextFileRequests&highlight=7-8)]

### <a name="irule-based-rule"></a>Irule tabanlı kural

Arabirimini uygulayan bir sınıfta kural mantığını kullanmak için kullanın <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*>. <xref:Microsoft.AspNetCore.Rewrite.IRule> `IRule`Yöntem tabanlı kural yaklaşımını kullanarak daha fazla esneklik sağlar. Uygulama sınıfınız, <xref:Microsoft.AspNetCore.Rewrite.IRule.ApplyRule*> yöntemi için parametreleri geçirebilmeniz için bir Oluşturucu içerebilir.

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=16-17)]

`extension` Ve içinörnekuygulamadakiparametrelerindeğerleri,çeşitlikoşullarauyacakşekildedenetlenir.`newPath` Bir değer içermeli ve değer *. png*, *. jpg*veya *. gif*olmalıdır. `extension` Geçerli değilse, bir <xref:System.ArgumentException> oluşturulur. `newPath` *Image. png*için bir istek yapılırsa, istek öğesine `/png-images/image.png`yeniden yönlendirilir. *Image. jpg*için bir istek yapılırsa, istek öğesine `/jpg-images/image.jpg`yeniden yönlendirilir. Durum kodu 301 olarak ayarlanır ve *kalıcı olarak taşınır*ve `context.Result` kuralları işlemeyi durdur ve yanıtı gönder olarak ayarlanır.

[!code-csharp[](url-rewriting/samples/3.x/SampleApp/RewriteRules.cs?name=snippet_RedirectImageRequests)]

Özgün Istek:`/image.png`

![Görüntü. png için istekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect_png_requests.png)

Özgün Istek:`/image.jpg`

![Image. jpg için istekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect_jpg_requests.png)

## <a name="regex-examples"></a>Regex örnekleri

| Hedef | Regex dize &<br>Match örneği | Değiştirme dizesi &<br>Çıkış örneği |
| ---- | ------------------------------- | -------------------------------------- |
| Yolu QueryString 'e yeniden yazın | `^path/(.*)/(.*)`<br>`/path/abc/123` | `path?var1=$1&var2=$2`<br>`/path?var1=abc&var2=123` |
| Eğik çizgiyi çıkar | `(.*)/$`<br>`/path/` | `$1`<br>`/path` |
| Sondaki eğik çizgiyi zorla | `(.*[^/])$`<br>`/path` | `$1/`<br>`/path/` |
| Belirli istekleri yeniden yazmayı önleyin | `^(.*)(?<!\.axd)$` veya `^(?!.*\.axd$)(.*)$`<br>Yes`/resource.htm`<br>Eşleşen`/resource.axd` | `rewritten/$1`<br>`/rewritten/resource.htm`<br>`/resource.axd` |
| URL segmentlerini yeniden Düzenle | `path/(.*)/(.*)/(.*)`<br>`path/1/2/3` | `path/$3/$2/$1`<br>`path/3/2/1` |
| URL segmentini değiştirme | `^(.*)/segment2/(.*)`<br>`/segment1/segment2/segment3` | `$1/replaced/$2`<br>`/segment1/replaced/segment3` |

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Bu belgede, ASP.NET Core uygulamalarında URL yeniden yazma ara yazılımı kullanma yönergeleriyle birlikte URL yeniden yazma tanıtılmaktadır.

URL yeniden yazma, istek URL 'Lerini bir veya daha fazla önceden tanımlanmış kurala göre değiştirme işlemidir. URL yeniden yazma, konumların ve adreslerin sıkı bir şekilde bağlanmaması için kaynak konumları ve adresleri arasında bir soyutlama oluşturur. URL yeniden yazma işlemi birkaç senaryoda yararlı olur:

* Sunucu kaynaklarını geçici olarak veya kalıcı olarak taşıyın veya değiştirin ve bu kaynakların kararlı konum belirleyicilerinin bakımını yapın.
* İstek işlemeyi farklı uygulamalar arasında veya bir uygulamanın alanlarında bölme.
* Gelen isteklerde URL segmentlerini kaldırın, ekleyin veya yeniden düzenleyin.
* Arama motoru Iyileştirmesi (SEO) için genel URL 'Leri iyileştirin.
* Ziyaretçilerin bir kaynak isteyerek döndürülen içeriği tahmin etmeye yardımcı olmak için kolay genel URL 'Lerin kullanılmasına izin verme.
* Güvensiz istekleri güvenli uç noktalara yönlendirin.
* Bir dış sitenin varlığı kendi içeriğine bağlayarak başka bir sitede barındırılan statik bir varlık kullandığı Hotlink 'i engelleyin.

> [!NOTE]
> URL yeniden yazma, bir uygulamanın performansını azaltabilir. Uygun yerlerde kuralların sayısını ve karmaşıklığını sınırlayın.

[Görüntüleme veya indirme örnek kodu](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/) ([nasıl indirileceğini](xref:index#how-to-download-a-sample))

## <a name="url-redirect-and-url-rewrite"></a>URL yeniden yönlendirme ve URL yeniden yazma

*URL yeniden yönlendirme* ve *URL yeniden yazma* arasındaki ifade farkı daha hafif ancak istemcilere kaynak sağlamak için önemli etkileri vardır. ASP.NET Core URL yeniden yazma ara yazılımı her ikisine de ihtiyacı verebilir.

*URL yeniden yönlendirme* , istemcinin, ilk olarak istenen istemciden farklı bir adresteki kaynağa erişmesi için bir istemci tarafı işlemi içerir. Bu, sunucuya gidiş dönüş gerektirir. İstemci kaynak için yeni bir istek yaptığında, istemciye döndürülen yeniden yönlendirme URL 'SI tarayıcının adres çubuğunda görüntülenir.

`/different-resource` Öğesine `/resource` yönlendiriliyorsasunucu,istemcinin,yenidenyönlendirmeningeçiciyadakalıcıolduğunubelirtenbirdurumkoduilekaynağıeldeetmesigerektiğiniyanıtverir.`/different-resource`

![Bir WebAPI hizmeti uç noktası, sürüm 1 ' den (v1) sunucudaki sürüm 2 ' ye (v2) geçici olarak değiştirildi. İstemci, Service 1 yolundaki hizmet için bir istek yapar/v1/api. Sunucu, Service 2/v2/apikonumundaki hizmetin yeni, geçici yoluyla bir 302 (bulunan) yanıtını geri gönderir. İstemci, yeniden yönlendirme URL 'sindeki hizmete ikinci bir istek yapar. Sunucu, 200 (Tamam) durum kodu ile yanıt verir.](url-rewriting/_static/url_redirect.png)

İstekleri farklı bir URL 'ye yönlendirirken, yanıtla birlikte durum kodunu belirterek yeniden yönlendirmenin kalıcı mi yoksa geçici mi olduğunu belirtin:

* *301-taşınan kalıcı* durum kodu, kaynağın yeni, kalıcı bir URL olduğu ve istemciye, gelecekteki tüm isteklerin kaynak için tüm istekleri yeni URL 'yi kullanması gerektiğini bildirmek istediğinizde kullanılır. *İstemci, 301 durum kodu alındığında yanıtı önbelleğe alabilir ve yeniden kullanabilir.*

* *302-bulunan* durum kodu, yeniden yönlendirmenin geçici ya da genellikle değişikliğe tabi olduğu durumlarda kullanılır. 302 durum kodu, istemcinin URL 'YI depolayamadığını ve gelecekte kullanacağını gösterir.

Durum kodları hakkında daha fazla bilgi için bkz [. RFC 2616: Durum kodu tanımları](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).

*URL yeniden yazma* , istenen istemciden farklı bir kaynak adresinden kaynak sağlayan sunucu tarafı bir işlemdir. URL yeniden yazma, sunucuya gidiş dönüş gerektirmez. Yeniden yazan URL istemciye döndürülmüyor ve tarayıcının adres çubuğunda görünmüyor.

`/different-resource`Öğesine `/resource` geridönerse,sunucu,kaynağınıdahili`/different-resource`olarak getirir ve döndürür.

İstemci, yeniden yazan URL 'de kaynağı alabiliyor olsa da, istemci isteği yaptığında ve yanıtı aldığında kaynağın yeniden yazan URL 'de bulunduğunu bilgilendirmez.

![Sunucu üzerinde sürüm 1 ' den (v1) sürüm 2 ' ye (v2) bir WebAPI hizmet uç noktası değiştirilmiştir. İstemci, Service 1 yolundaki hizmet için bir istek yapar/v1/api. İstek URL 'SI, hizmet 2 yolu/v2/api'daki hizmete erişmek için yeniden yazıldı. Hizmet, istemciye 200 (Tamam) durum kodu ile yanıt verir.](url-rewriting/_static/url_rewrite.png)

## <a name="url-rewriting-sample-app"></a>URL yeniden yazma örnek uygulaması

[Örnek uygulamayla](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/url-rewriting/samples/)birlikte YENIDEN yazma URL 'sinin özelliklerini inceleyebilirsiniz. Uygulama yeniden yönlendirme ve yeniden yazma kuralları uygular ve birkaç senaryo için yeniden yönlendirilen veya yeniden yazan URL 'YI gösterir.

## <a name="when-to-use-url-rewriting-middleware"></a>URL yeniden yazma ara yazılımı ne zaman kullanılır

Aşağıdaki yaklaşımlardan birini kullandığımmdan URL yeniden yazma ara yazılımı kullanın:

* [Windows Server 'da IIS ile URL yeniden yazma modülü](https://www.iis.net/downloads/microsoft/url-rewrite)
* [Apache Server 'da Apache mod_rewrite modülü](https://httpd.apache.org/docs/2.4/rewrite/)
* [NGINX üzerinde URL yeniden yazma](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)

Ayrıca, uygulama [http. sys sunucusunda](xref:fundamentals/servers/httpsys) barındırıldığı zaman ara yazılımı kullanın (eski adıyla webListener olarak adlandırılır).

IIS, Apache ve NGINX 'te sunucu tabanlı URL yeniden yazma teknolojilerini kullanmanın başlıca nedenleri şunlardır:

* Ara yazılım bu modüllerin tüm özelliklerini desteklemez.

  Sunucu modüllerinin bazı özellikleri, `IsFile` IIS yeniden yazma modülünün ve `IsDirectory` kısıtlamaları gibi ASP.NET Core projelerle birlikte çalışmaz. Bu senaryolarda, bunun yerine ara yazılımı kullanın.
* Ara yazılım performansı büyük olasılıkla modüllerle eşleşmiyor.

  Sınama, performansı en iyi şekilde düşürür veya performans düşüklüğü göz ardı edilebilir olduğundan emin olmanın tek yoludur.

## <a name="package"></a>Paket

Ara yazılımı projenize dahil etmek için, Microsoft. AspNetCore. [yeniden yazma](https://www.nuget.org/packages/Microsoft.AspNetCore.Rewrite) paketini içeren proje dosyasındaki [Microsoft. Aspnetcore. app metapackage](xref:fundamentals/metapackage-app) öğesine bir paket başvurusu ekleyin.

`Microsoft.AspNetCore.App` Metapackage 'i kullanmıyorsanız, `Microsoft.AspNetCore.Rewrite` pakete bir proje başvurusu ekleyin.

## <a name="extension-and-options"></a>Uzantı ve Seçenekler

Yeniden yazma kurallarınızın her biri için uzantı yöntemleriyle [Rewriteoptions](xref:Microsoft.AspNetCore.Rewrite.RewriteOptions) sınıfının bir ÖRNEĞINI oluşturarak URL yeniden yazma ve yeniden yönlendirme kuralları oluşturun. Birden çok kuralı, işlenmeyi istediğiniz sırada zincirle. , `RewriteOptions` İle istek ardışık düzenine eklendikçe, bu URL 'ye yeniden yazma ara yazılımı ile <xref:Microsoft.AspNetCore.Builder.RewriteBuilderExtensions.UseRewriter*>geçirilir:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1)]

### <a name="redirect-non-www-to-www"></a>Www olmayan www 'e yönlendirme

Üç seçenek, uygulamanın`www` `www`istek olmayan istekleri yeniden yönlendirmesine izin verir:

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWwwPermanent*>İstek değilse isteği alt `www` etki alanına kalıcı olarak yeniden yönlendirin.`www` &ndash; [Status308PermanentRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status308PermanentRedirect) durum kodu ile yeniden yönlendirir.

* <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToWww*>Gelen istek değilse isteği `www` alt etki alanına yönlendirin.`www` &ndash; [Status307TemporaryRedirect](xref:Microsoft.AspNetCore.Http.StatusCodes.Status307TemporaryRedirect) durum kodu ile yeniden yönlendirir. Aşırı yükleme, yanıt için durum kodu sağlamanıza izin verir. Bir durum kodu ataması için <xref:Microsoft.AspNetCore.Http.StatusCodes> sınıfın bir alanını kullanın.

### <a name="url-redirect"></a>URL yeniden yönlendirme

İstekleri <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirect*> yeniden yönlendirmek için kullanın. İlk parametre gelen URL 'nin yolu ile eşleşen Regex içerir. İkinci parametre değiştirme dizesidir. Varsa, üçüncü parametre durum kodunu belirtir. Durum kodunu belirtmezseniz, durum kodu varsayılan olarak *302-bulunur*; bu da kaynağın geçici olarak taşındığını veya değiştirildiğini gösterir.

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=9)]

Geliştirici araçları etkinleştirilmiş bir tarayıcıda, örnek uygulamaya yol `/redirect-rule/1234/5678`ile bir istek oluşturun. Regex, üzerindeki `redirect-rule/(.*)`istek yoluyla eşleşir ve yol ile `/redirected/1234/5678`değiştirilmiştir. Yeniden yönlendirme URL 'SI, *302 tarafından bulunan* bir durum kodu ile istemciye geri gönderilir. Tarayıcı, tarayıcının adres çubuğunda görüntülenen yeniden yönlendirme URL 'SI üzerinde yeni bir istek oluşturur. Örnek uygulamadaki hiçbir kural, yeniden yönlendirme URL 'SI üzerinde eşleşmediğinden:

* İkinci istek uygulamadan *200-Tamam* yanıtı alır.
* Yanıtın gövdesi, yeniden yönlendirme URL 'sini gösterir.

Bir URL *yeniden yönlendirildiğinde*sunucuya gidiş dönüş yapılır.

> [!WARNING]
> Yeniden yönlendirme kuralları oluştururken dikkatli olun. Yeniden yönlendirme kuralları, bir yeniden yönlendirmeden sonra dahil olmak üzere, uygulamaya yapılan her istekte değerlendirilir. Yanlışlıkla *sonsuz yeniden yönlendirmeler döngüsü*oluşturmak kolaydır.

Özgün Istek:`/redirect-rule/1234/5678`

![İstekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect.png)

Parantez içinde yer alan ifadenin kısmına bir *yakalama grubu*denir. İfadenin nokta (`.`), *herhangi bir karakterle eşleşir*anlamına gelir. Yıldız işareti (`*`) *, önceki karakterle sıfır veya daha fazla kez eşleşme*gösterir. Bu nedenle, URL `1234/5678`'nin son iki yol kesimi, yakalama grubu `(.*)`tarafından yakalanır. Sonrasında istek URL 'sinde sağladığınız herhangi bir değer bu `redirect-rule/` tek yakalama grubu tarafından yakalandıktan sonra.

Değiştirme dizesinde, yakalanan gruplar, dolar işareti (`$`) ve ardından yakalamanın sıra numarası ile birlikte dizeye eklenir. İlk yakalama grubu değeri ile `$1`elde edilir, ikincisi ile `$2`ve normal Regex yakalama grupları için sırayla devam eder. Örnek uygulamadaki yeniden yönlendirme kuralı Regex bölümünde yalnızca bir tane yakalanan grup bulunur, bu `$1`nedenle değiştirme dizesinde yalnızca bir tane eklenmiş grup vardır. Kural uygulandığında, URL olur `/redirected/1234/5678`.

### <a name="url-redirect-to-a-secure-endpoint"></a>Güvenli bir uç noktaya URL yönlendirmesi

HTTP <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttps*> isteklerini https protokolünü kullanarak aynı konağa ve yola yeniden yönlendirmek için kullanın. Durum kodu sağlanmazsa, ara yazılım varsayılan olarak *302-bulunur*. Bağlantı noktası sağlanmazsa:

* Ara yazılım varsayılan olarak `null`olur.
* Şema (https Protokolü `https` ) olarak değişir ve istemci, 443 numaralı bağlantı noktasında kaynağa erişir.

Aşağıdaki örnek, durum kodunun *301-kalıcı olarak taşınacağını* ve bağlantı noktasını 5001 olarak nasıl değiştirileceğini gösterir.

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttps(301, 5001);

    app.UseRewriter(options);
}
```

Güvenli <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRedirectToHttpsPermanent*> olmayan istekleri, bağlantı noktası 443 üzerinde güvenli https Protokolü ile aynı konağa ve yola yeniden yönlendirmek için kullanın. Ara yazılım durum kodunu 301 olarak ayarlar ve *kalıcı olarak taşınır*.

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttpsPermanent();

    app.UseRewriter(options);
}
```

> [!NOTE]
> Ek yeniden yönlendirme kuralları gereksinimi olmadan güvenli bir uç noktaya yönlendirilirken, HTTPS yeniden yönlendirme ara yazılımı kullanmanızı öneririz. Daha fazla bilgi için bkz. [https 'Yi zorla](xref:security/enforcing-ssl#require-https) konusu.

Örnek uygulama, veya `AddRedirectToHttps` `AddRedirectToHttpsPermanent`kullanımını gösterme yeteneğine sahiptir. Uzantı yöntemini öğesine `RewriteOptions`ekleyin. Herhangi bir URL 'de uygulamaya güvenli olmayan bir istek oluşturun. Otomatik olarak imzalanan sertifikanın güvenilmeyen tarayıcı güvenlik uyarısını kapatın veya sertifikaya güvenmek için bir özel durum oluşturun.

Kullanarak `AddRedirectToHttps(301, 5001)`özgün istek:`http://localhost:5000/secure`

![İstekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect_to_https.png)

Kullanarak `AddRedirectToHttpsPermanent`özgün istek:`http://localhost:5000/secure`

![İstekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect_to_https_permanent.png)

### <a name="url-rewrite"></a>URL yeniden yazma

URL <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.AddRewrite*> 'leri yeniden yazma kuralı oluşturmak için kullanın. İlk parametre gelen URL yolundaki eşleşme için Regex içerir. İkinci parametre değiştirme dizesidir. Üçüncü parametresi `skipRemainingRules: {true|false}`, geçerli kural uygulanmışsa ek yeniden yazma kurallarının atlanıp atlanmayacağını gösterir.

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=10-11)]

Özgün Istek:`/rewrite-rule/1234/5678`

![İsteği ve yanıtı izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_rewrite.png)

İfadenin başındaki simgeyi seçtiğinizde`^`(), eşleşmesinin URL yolunun başlangıcında başladığı anlamına gelir.

Önceki örnekte, yeniden yönlendirme kuralıyla `redirect-rule/(.*)`, Regex başlangıcında simgeyi seçtiğinizde (`^`) yoktur. Bu nedenle, başarılı bir eşleşme `redirect-rule/` için herhangi bir karakterden önce yol açabilir.

| Yol                               | Eşleştirme |
| ---------------------------------- | :---: |
| `/redirect-rule/1234/5678`         | Evet   |
| `/my-cool-redirect-rule/1234/5678` | Evet   |
| `/anotherredirect-rule/1234/5678`  | Evet   |

Yeniden yazma kuralı `^rewrite-rule/(\d+)/(\d+)`, yalnızca ile `rewrite-rule/`başlarsa yollarla eşleşir. Aşağıdaki tabloda, eşleşen farkı aklınızda.

| Yol                              | Eşleştirme |
| --------------------------------- | :---: |
| `/rewrite-rule/1234/5678`         | Evet   |
| `/my-cool-rewrite-rule/1234/5678` | Hayır    |
| `/anotherrewrite-rule/1234/5678`  | Hayır    |

İfadenin bölümünü takip eden iki yakalama `(\d+)/(\d+)`grubu vardır. `^rewrite-rule/` Belirtir bir *sayıyla (sayı) eşleşir.* `\d` Artı işareti (`+`) *bir veya daha fazla önceki karakterden eşleşiyor*demektir. Bu nedenle, URL bir sayı içermeli ve ardından İleri eğik çizgi ve ardından başka bir sayı içermelidir. Bu yakalama grupları, ve olarak `$1` yeniden `$2`yazan URL 'sine eklenir. Yeniden yazma kuralı değiştirme dizesi yakalanan grupları sorgu dizesine koyar. İstenen yolu `/rewrite-rule/1234/5678` , `/rewritten?var1=1234&var2=5678`kaynağı elde etmek için yeniden yazılır. Özgün istekte bir sorgu dizesi varsa, URL yeniden yazdığınızda korunur.

Kaynağı almak için sunucuya gidiş dönüş yok. Kaynak varsa, bu, alınır ve istemciye *200-ok* durum kodu ile döndürülür. İstemci yeniden yönlendirmediği için tarayıcının adres çubuğundaki URL değişmez. İstemciler, sunucuda bir URL yeniden yazma işleminin gerçekleştiğini algılayamaz.

> [!NOTE]
> Eşleşen `skipRemainingRules: true` kuralların hesaplama maliyeti ve uygulama yanıt süresini arttığı için mümkün olan her durumda kullanın. En hızlı uygulama yanıtı için:
>
> * En sık eşleşen kuraldan en az sıklıkta eşleşen kurala göre yeniden yazma kuralları.
> * Bir eşleşme gerçekleştiğinde ve ek kural işleme gerekli olmadığında kalan kuralların işlenmesini atlayın.

### <a name="apache-mod_rewrite"></a>Apache mod_rewrite

İle <xref:Microsoft.AspNetCore.Rewrite.ApacheModRewriteOptionsExtensions.AddApacheModRewrite*>Apache mod_rewrite kurallarını uygulayın. Kurallar dosyasının uygulamayla birlikte dağıtıldığından emin olun. Daha fazla bilgi ve mod_rewrite kuralları örnekleri için bkz. [Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/).

, ApacheModRewrite. txt kuralları dosyasındaki kuralları okumak için kullanılır: <xref:System.IO.StreamReader>

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=3-4,12)]

Örnek uygulama, istekleri ' den `/apache-mod-rules-redirect/(.\*)` ' `/redirected?id=$1`e yeniden yönlendirir. Yanıt durum kodu *302-bulundu*.

[!code[](url-rewriting/samples/2.x/SampleApp/ApacheModRewrite.txt)]

Özgün Istek:`/apache-mod-rules-redirect/1234`

![İstekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_apache_mod_redirect.png)

Ara yazılım aşağıdaki Apache mod_rewrite Server değişkenlerini destekler:

* CONN_REMOTE_ADDR
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_FORWARDED
* HTTP_HOST
* HTTP_REFERER
* HTTP_USER_AGENT
* HTTPS
* IPV6
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_METHOD
* REQUEST_SCHEME
* REQUEST_URI
* SCRIPT_FILENAME
* SERVER_ADDR
* SERVER_PORT
* SERVER_PROTOCOL
* TIME
* TIME_DAY
* TIME_HOUR
* TIME_MIN
* TIME_MON
* TIME_SEC
* TIME_WDAY
* TIME_YEAR

### <a name="iis-url-rewrite-module-rules"></a>IIS URL yeniden yazma modülü kuralları

IIS URL yeniden yazma modülü için geçerli olan kural kümesini kullanmak için kullanın <xref:Microsoft.AspNetCore.Rewrite.IISUrlRewriteOptionsExtensions.AddIISUrlRewrite*>. Kurallar dosyasının uygulamayla birlikte dağıtıldığından emin olun. Windows Server IIS 'de çalışırken uygulamanın *Web. config* dosyasını kullanmak için ara yazılımı yönlendirmeyin. IIS ile, IIS yeniden yazma modülüyle çakışmalardan kaçınmak için bu kuralların uygulamanın *Web. config* dosyası dışında depolanması gerekir. Daha fazla bilgi ve IIS URL yeniden yazma modülü kurallarının örnekleri için bkz. [Using URL yeniden yazma modülü 2,0](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20) ve [URL yeniden yazma modülü yapılandırma başvurusu](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference).

, Iisurlyeniden *yazma. xml* kuralları dosyasındaki kuralları okumak için kullanılır: <xref:System.IO.StreamReader>

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=5-6,13)]

Örnek uygulama, ' den ' `/iis-rules-rewrite/(.*)` e `/rewritten?id=$1`olan istekleri yeniden yazar. Yanıt, istemciye *200-ok* durum kodu ile gönderilir.

[!code-xml[](url-rewriting/samples/2.x/SampleApp/IISUrlRewrite.xml)]

Özgün Istek:`/iis-rules-rewrite/1234`

![İsteği ve yanıtı izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_iis_url_rewrite.png)

Uygulamanızı istenmeyen yollarla etkileyebilecek sunucu düzeyi kurallara sahip etkin bir IIS yeniden yazma modülünüzün olması halinde, bir uygulama için IIS yeniden yazma modülünü devre dışı bırakabilirsiniz. Daha fazla bilgi için bkz. [IIS modüllerini devre dışı bırakma](xref:host-and-deploy/iis/modules#disabling-iis-modules).

#### <a name="unsupported-features"></a>Desteklenmeyen özellikler

ASP.NET Core 2. x ile yayınlanan ara yazılım, aşağıdaki IIS URL yeniden yazma modülü özelliklerini desteklemez:

* Giden kuralları
* Özel sunucu değişkenleri
* Karakterlerini
* LogRewrittenUrl 'Si

#### <a name="supported-server-variables"></a>Desteklenen sunucu değişkenleri

Ara yazılım aşağıdaki IIS URL yeniden yazma modülü sunucu değişkenlerini destekler:

* CONTENT_LENGTH
* CONTENT_TYPE
* HTTP_ACCEPT
* HTTP_CONNECTION
* HTTP_COOKIE
* HTTP_HOST
* HTTP_REFERER
* HTTP_URL
* HTTP_USER_AGENT
* HTTPS
* LOCAL_ADDR
* QUERY_STRING
* REMOTE_ADDR
* REMOTE_PORT
* REQUEST_FILENAME
* REQUEST_URI

> [!NOTE]
> Ayrıca bir <xref:Microsoft.Extensions.FileProviders.IFileProvider> <xref:Microsoft.Extensions.FileProviders.PhysicalFileProvider>ile elde edebilirsiniz. Bu yaklaşım, yeniden yazma kuralları dosyalarınızın konumu için daha fazla esneklik sağlayabilir. Yeniden yazma kuralları dosyalarınızın sağladığınız yoldaki sunucuya dağıtıldığından emin olun.
>
> ```csharp
> PhysicalFileProvider fileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory());
> ```

### <a name="method-based-rule"></a>Yöntem tabanlı kural

Bir <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*> yöntemde kendi kural mantığınızı uygulamak için kullanın. `Add`, metodunda kullanım <xref:Microsoft.AspNetCore.Http.HttpContext> için kullanılabilir hale getiren öğesinigösterir.<xref:Microsoft.AspNetCore.Rewrite.RewriteContext> [Rewritecontext. Result](xref:Microsoft.AspNetCore.Rewrite.RewriteContext.Result*) , ek ardışık düzen işlemenin nasıl işlendiğini belirler. Değeri aşağıdaki tabloda açıklanan <xref:Microsoft.AspNetCore.Rewrite.RuleResult> alanlardan birine ayarlayın.

| `RewriteContext.Result`              | Eylem                                                           |
| ------------------------------------ | ---------------------------------------------------------------- |
| `RuleResult.ContinueRules`varsayılanını | Kuralları uygulamaya devam edin.                                         |
| `RuleResult.EndResponse`             | Kuralları uygulamayı durdurun ve yanıtı gönderin.                       |
| `RuleResult.SkipRemainingRules`      | Kuralları uygulamayı durdurun ve bağlamı bir sonraki ara yazılıma gönderin. |

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=14)]

Örnek uygulama, *. xml*ile biten yollar için istekleri yeniden yönlendiren bir yöntemi gösterir. İçin `/file.xml`bir istek yapılırsa, istek öğesine `/xmlfiles/file.xml`yeniden yönlendirilir. Durum kodu *301*olarak ayarlanır. Tarayıcı */xmlfiles/File.xml*için yeni bir istek yaptığında, statik dosya ara yazılımı dosyayı *Wwwroot/xmlfiles* klasöründen istemciye sunar. Yeniden yönlendirme için, yanıtın durum kodunu açık olarak ayarlayın. Aksi takdirde, *200-ok* durum kodu döndürülür ve yeniden yönlendirme istemcide gerçekleşmez.

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/RewriteRules.cs?name=snippet_RedirectXmlFileRequests&highlight=14-18)]

Bu yaklaşım ayrıca istekleri yeniden yazabilir. Örnek uygulama, *dosya. txt* metin dosyasına *Wwwroot* klasöründen hizmeti sağlamak için herhangi bir metin dosyası isteğinin yolunu yeniden yazmayı gösterir. Statik dosya ara yazılımı, güncelleştirilmiş istek yoluna göre dosyayı sunar:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=15,22)]

*RewriteRules.cs*:

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/RewriteRules.cs?name=snippet_RewriteTextFileRequests&highlight=7-8)]

### <a name="irule-based-rule"></a>Irule tabanlı kural

Arabirimini uygulayan bir sınıfta kural mantığını kullanmak için kullanın <xref:Microsoft.AspNetCore.Rewrite.RewriteOptionsExtensions.Add*>. <xref:Microsoft.AspNetCore.Rewrite.IRule> `IRule`Yöntem tabanlı kural yaklaşımını kullanarak daha fazla esneklik sağlar. Uygulama sınıfınız, <xref:Microsoft.AspNetCore.Rewrite.IRule.ApplyRule*> yöntemi için parametreleri geçirebilmeniz için bir Oluşturucu içerebilir.

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=16-17)]

`extension` Ve içinörnekuygulamadakiparametrelerindeğerleri,çeşitlikoşullarauyacakşekildedenetlenir.`newPath` Bir değer içermeli ve değer *. png*, *. jpg*veya *. gif*olmalıdır. `extension` Geçerli değilse, bir <xref:System.ArgumentException> oluşturulur. `newPath` *Image. png*için bir istek yapılırsa, istek öğesine `/png-images/image.png`yeniden yönlendirilir. *Image. jpg*için bir istek yapılırsa, istek öğesine `/jpg-images/image.jpg`yeniden yönlendirilir. Durum kodu 301 olarak ayarlanır ve *kalıcı olarak taşınır*ve `context.Result` kuralları işlemeyi durdur ve yanıtı gönder olarak ayarlanır.

[!code-csharp[](url-rewriting/samples/2.x/SampleApp/RewriteRules.cs?name=snippet_RedirectImageRequests)]

Özgün Istek:`/image.png`

![Görüntü. png için istekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect_png_requests.png)

Özgün Istek:`/image.jpg`

![Image. jpg için istekleri ve yanıtları izleyen Geliştirici Araçları tarayıcı penceresi](url-rewriting/_static/add_redirect_jpg_requests.png)

## <a name="regex-examples"></a>Regex örnekleri

| Hedef | Regex dize &<br>Match örneği | Değiştirme dizesi &<br>Çıkış örneği |
| ---- | ------------------------------- | -------------------------------------- |
| Yolu QueryString 'e yeniden yazın | `^path/(.*)/(.*)`<br>`/path/abc/123` | `path?var1=$1&var2=$2`<br>`/path?var1=abc&var2=123` |
| Eğik çizgiyi çıkar | `(.*)/$`<br>`/path/` | `$1`<br>`/path` |
| Sondaki eğik çizgiyi zorla | `(.*[^/])$`<br>`/path` | `$1/`<br>`/path/` |
| Belirli istekleri yeniden yazmayı önleyin | `^(.*)(?<!\.axd)$` veya `^(?!.*\.axd$)(.*)$`<br>Yes`/resource.htm`<br>Eşleşen`/resource.axd` | `rewritten/$1`<br>`/rewritten/resource.htm`<br>`/resource.axd` |
| URL segmentlerini yeniden Düzenle | `path/(.*)/(.*)/(.*)`<br>`path/1/2/3` | `path/$3/$2/$1`<br>`path/3/2/1` |
| URL segmentini değiştirme | `^(.*)/segment2/(.*)`<br>`/segment1/segment2/segment3` | `$1/replaced/$2`<br>`/segment1/replaced/segment3` |

::: moniker-end

## <a name="additional-resources"></a>Ek kaynaklar

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [.NET 'teki normal ifadeler](/dotnet/articles/standard/base-types/regular-expressions)
* [Normal ifade dili-hızlı başvuru](/dotnet/articles/standard/base-types/quick-ref)
* [Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/)
* [URL yeniden yazma modülünü kullanma 2,0 (IIS için)](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20)
* [URL yeniden yazma modülü yapılandırma başvurusu](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference)
* [IIS URL yeniden yazma modülü Forumu](https://forums.iis.net/1152.aspx)
* [Basit URL yapısını saklama](https://support.google.com/webmasters/answer/76329?hl=en)
* [10 URL yeniden yazma Ipuçları ve püf noktaları](https://ruslany.net/2009/04/10-url-rewriting-tips-and-tricks/)
* [Eğik çizgi veya eğik çizgi](https://webmasters.googleblog.com/2010/04/to-slash-or-not-to-slash.html)
