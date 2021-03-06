---
title: HTTPS üzerinden Docker ile görüntüleri barındırma ASP.NET Core
author: rick-anderson
description: HTTPS üzerinden Docker ile ASP.NET Core görüntülerini nasıl barındıralabileceğinizi öğrenin
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/05/2019
no-loc:
- Let's Encrypt
uid: security/docker-https
ms.openlocfilehash: 07e2791e5b26975c71323f8cb41a4b0fbe0cdf11
ms.sourcegitcommit: 2388c2a7334ce66b6be3ffbab06dd7923df18f60
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 01/14/2020
ms.locfileid: "75952133"
---
# <a name="hosting-aspnet-core-images-with-docker-over-https"></a>HTTPS üzerinden Docker ile görüntüleri barındırma ASP.NET Core

Tarafından [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core [Varsayılan olarak https](/aspnet/core/security/enforcing-ssl)kullanır. [Https](https://en.wikipedia.org/wiki/HTTPS) , güven, kimlik ve şifreleme için [sertifikalara](https://en.wikipedia.org/wiki/Public_key_certificate) bağımlıdır.

Bu belgede önceden oluşturulmuş kapsayıcı görüntülerinin HTTPS ile nasıl çalıştırılacağı açıklanmaktadır.

Geliştirme senaryoları için bkz. [https üzerinden Docker ile ASP.NET Core uygulamaları geliştirme](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/aspnetcore-docker-https-development.md) .

Bu örnek, [Docker Istemcisinin](https://www.docker.com/products/docker) [Docker 17,06](https://docs.docker.com/release-notes/docker-ce) veya sonraki bir sürümünü gerektirir.

## <a name="prerequisites"></a>Prerequisites

Bu belgedeki bazı yönergeler için [.NET Core 2,2 SDK](https://www.microsoft.com/net/download) veya üzeri gereklidir.

## <a name="certificates"></a>Sertifikalar

Bir etki alanı için [Üretim barındırma](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) için bir [sertifika yetkilisinden](https://wikipedia.org/wiki/Certificate_authority) bir sertifika gereklidir. [Let's Encrypt](https://letsencrypt.org/) , ücretsiz sertifikalar sunan bir sertifika yetkilisindir.

Bu belge, `localhost`üzerinde önceden oluşturulmuş görüntüleri barındırmak için [otomatik olarak imzalanan geliştirme sertifikaları](https://en.wikipedia.org/wiki/Self-signed_certificate) kullanır. Yönergeler, üretim sertifikalarını kullanmaya benzerdir.

Üretim sertifikaları için:

* `dotnet dev-certs` aracı gerekli değildir.
* Sertifikaların, yönergelerde kullanılan konumda depolanması gerekmez. Herhangi bir konumun çalışması gerekir, ancak bu sertifikalar site dizininizde depolanarak önerilmez.

Aşağıdaki bölümde yer alan yönergeler, Docker 'ın `-v` komut satırı seçeneğini kullanarak sertifikaları kapsayıcılara bağlama. Bir *Dockerfile*içinde `COPY` komutuyla kapsayıcı görüntülerine sertifika ekleyebilirsiniz, ancak bu önerilmez. Sertifikaları bir görüntüye kopyalamak aşağıdaki nedenlerden dolayı önerilmez:

* Geliştirici sertifikaları ile test etmek için aynı görüntünün kullanımını zorlaştırır.
* Üretim sertifikaları ile barındırmak için aynı görüntünün kullanımını zorlaştırır.
* Sertifika açıklanmasının önemli bir riski vardır.

## <a name="running-pre-built-container-images-with-https"></a>HTTPS ile önceden oluşturulmuş kapsayıcı görüntülerini çalıştırma

İşletim sistemi yapılandırmanız için aşağıdaki yönergeleri kullanın.

### <a name="windows-using-linux-containers"></a>Linux kapsayıcıları kullanan Windows

Sertifika Oluştur ve yerel makineyi Yapılandır:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

Yukarıdaki komutlarda `{ password here }` bir parolayla değiştirin.

Kapsayıcı görüntüsünü HTTPS için yapılandırılan ASP.NET Core çalıştırın:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

Parolanın, sertifika için kullanılan parolayla eşleşmesi gerekir.

### <a name="macos-or-linux"></a>macOS veya Linux

Sertifika Oluştur ve yerel makineyi Yapılandır:

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

`dotnet dev-certs https --trust` yalnızca macOS ve Windows 'ta desteklenir. Linux 'ta sertifikalarınızın desteklediği şekilde sertifika almanız gerekir. Büyük olasılıkla, tarayıcınızda sertifikaya güvenmeniz gerekir.

Yukarıdaki komutlarda `{ password here }` bir parolayla değiştirin.

Kapsayıcı görüntüsünü HTTPS için yapılandırılan ASP.NET Core çalıştırın:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

Parolanın, sertifika için kullanılan parolayla eşleşmesi gerekir.

### <a name="windows-using-windows-containers"></a>Windows kapsayıcıları kullanan pencereler

Sertifika Oluştur ve yerel makineyi Yapılandır:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

Yukarıdaki komutlarda `{ password here }` bir parolayla değiştirin.

Kapsayıcı görüntüsünü HTTPS için yapılandırılan ASP.NET Core çalıştırın:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=\https\aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:C:\https\ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

Parolanın, sertifika için kullanılan parolayla eşleşmesi gerekir.
