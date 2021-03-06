---
title: LibMan ile ASP.NET Core istemci tarafı kitaplık alma
author: scottaddie
description: Kitaplık Yöneticisi 'Ni (LibMan) kullanarak bir ASP.NET Core projesindeki istemci tarafı kitaplık varlıklarını yüklemeyi öğrenin.
ms.author: scaddie
ms.custom: mvc
ms.date: 08/14/2018
uid: client-side/libman/index
ms.openlocfilehash: 87987446b7f2c625da90951510e697e06569ba36
ms.sourcegitcommit: e7c56e8da5419bbc20b437c2dd531dedf9b0dc6b
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 09/10/2019
ms.locfileid: "70878336"
---
# <a name="client-side-library-acquisition-in-aspnet-core-with-libman"></a>LibMan ile ASP.NET Core istemci tarafı kitaplık alma

[Scott Ade](https://twitter.com/Scott_Addie) tarafından

Kitaplık Yöneticisi (LibMan), basit, istemci tarafı bir kitaplık alma aracıdır. LibMan, dosya sisteminden veya bir [içerik teslim ağından (CDN)](https://wikipedia.org/wiki/Content_delivery_network)popüler kitaplıkları ve çerçeveleri indirir. Desteklenen CDNs, [Cdnjs](https://cdnjs.com/), [jsdelivr](https://www.jsdelivr.com/)ve [unpkg](https://unpkg.com/#/)'yi içerir. Seçilen kitaplık dosyaları getirildi ve ASP.NET Core projesi içindeki uygun konuma yerleştirildi.

## <a name="libman-use-cases"></a>LibMan kullanım örnekleri

LibMan aşağıdaki avantajları sunar:

* Yalnızca ihtiyacınız olan kitaplık dosyaları indirilir.
* [Node. js](https://nodejs.org), [NPM](https://www.npmjs.com)ve [WebPack](https://webpack.js.org)gibi ek araçlar, bir kitaplıktaki dosyaların alt kümesini almak için gerekli değildir.
* Dosyalar, derleme görevleri veya el ile dosya kopyalama işlemleri yapılmadan belirli bir konuma yerleştirilebilir.

Libman 'ın avantajları hakkında daha fazla bilgi için Visual [Studio 2017 ' de modern ön uç Web geliştirmesini izleyin: LibMan segmenti](https://channel9.msdn.com/Events/Build/2017/B8073#time=43m34s).

LibMan bir paket yönetim sistemi değildir. Zaten NPM veya [Yarn](https://yarnpkg.com)gibi bir paket Yöneticisi kullanıyorsanız, bunu yapmaya devam edin. LibMan, bu araçların yerini alacak şekilde geliştirilmedi.

## <a name="additional-resources"></a>Ek kaynaklar

* <xref:client-side/libman/libman-vs>
* <xref:client-side/libman/libman-cli>
* [LibMan GitHub deposu](https://github.com/aspnet/LibraryManager)
