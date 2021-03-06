---
title: Kimlik doğrulaması ve kimliği ASP.NET Core geçir
author: ardalis
description: Bir ASP.NET MVC projesinden kimlik doğrulaması ve kimliği ASP.NET Core MVC projesine geçirmeyi öğrenin.
ms.author: riande
ms.date: 10/14/2016
uid: migration/identity
ms.openlocfilehash: f821930dbd36de18db31104cddf34c563009a506
ms.sourcegitcommit: 476ea5ad86a680b7b017c6f32098acd3414c0f6c
ms.translationtype: MT
ms.contentlocale: tr-TR
ms.lasthandoff: 08/14/2019
ms.locfileid: "69022275"
---
# <a name="migrate-authentication-and-identity-to-aspnet-core"></a>Kimlik doğrulaması ve kimliği ASP.NET Core geçir

[Steve Smith](https://ardalis.com/) tarafından

Önceki makalede, [yapılandırmayı ASP.NET Core MVC 'ye bir ASP.NET MVC projesinden geçirdik](xref:migration/configuration). Bu makalede, kayıt, oturum açma ve Kullanıcı yönetimi özelliklerini geçiririz.

## <a name="configure-identity-and-membership"></a>Kimliği ve üyeliği yapılandırma

ASP.NET MVC 'de, kimlik doğrulama ve kimlik özellikleri *App_Start* klasöründe bulunan *Startup.auth.cs* ve *IdentityConfig.cs*' de ASP.NET Identity kullanılarak yapılandırılır. ASP.NET Core MVC 'de, bu özellikler *Startup.cs*' de yapılandırılır.

`Microsoft.AspNetCore.Identity.EntityFrameworkCore` Ve`Microsoft.AspNetCore.Authentication.Cookies` NuGet paketlerini yükler.

Ardından, *Startup.cs* açın ve Entity Framework ve `Startup.ConfigureServices` kimlik hizmetlerini kullanmak için yöntemi güncelleştirin:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add EF services to the services container.
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

     services.AddMvc();
}
```

Bu noktada, yukarıdaki kodda henüz ASP.NET MVC projesinden geçirdiğimiz iki tür vardır: `ApplicationDbContext` ve. `ApplicationUser` ASP.NET Core projesinde yeni *modeller* klasörü oluşturun ve bu türlere karşılık gelen kendisine iki sınıf ekleyin. Bu sınıfların ASP.NET MVC sürümlerini */models/ıdentitymodels.cs*içinde bulacaksınız, ancak geçirilen projede sınıf başına bir dosya kullanacağız çünkü bu daha net bir şekilde yapılır.

*ApplicationUser.cs*:

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;

namespace NewMvcProject.Models
{
  public class ApplicationUser : IdentityUser
  {
  }
}
```

*ApplicationDbContext.cs*:

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.Data.Entity;

namespace NewMvcProject.Models
{
    public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            base.OnModelCreating(builder);
            // Customize the ASP.NET Core Identity model and override the defaults if needed.
            // For example, you can rename the ASP.NET Core Identity table names and more.
            // Add your customizations after calling base.OnModelCreating(builder);
        }
    }
}
```

ASP.NET Core MVC Başlatıcı Web projesi, kullanıcıların çok fazla özelleştirmesini veya `ApplicationDbContext`' i içermez. Gerçek bir uygulamayı geçirirken uygulamanızın kullanıcı ve `DbContext` sınıflarının tüm özel özelliklerini ve yöntemlerini, uygulamanızın de tüm diğer model sınıflarını geçirmeniz gerekir. Örneğin `DbContext` `Album` , varsa, sınıfı geçirmeniz gerekir. `DbSet<Album>`

Bu dosyalar söz konusu olduğunda, *Startup.cs* dosyası `using` deyimlerini güncelleştirerek derleme yapmak için kullanılabilir:

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Hosting;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
```

Uygulamamız artık kimlik doğrulama ve kimlik hizmetlerini desteklemeye hazırdır. Yalnızca bu özelliklerin kullanıcılara açık olması gerekir.

## <a name="migrate-registration-and-login-logic"></a>Kayıt ve oturum açma mantığını geçirme

Uygulama için yapılandırılmış kimlik hizmetleri ve Entity Framework ve SQL Server kullanılarak yapılandırılan veri erişimi sayesinde, kayıt ve uygulamaya oturum açma desteği eklemeye hazırız. [Daha önce geçiş sürecinde,](xref:migration/mvc#migrate-the-layout-file) *_Layout. cshtml*içinde *_loginpartial* öğesine bir başvuru yorumlayacağız. Artık bu koda geri dönmeli, bu kodun açıklamasını kaldırın ve oturum açma işlevlerini desteklemek için gerekli denetleyiciler ve görünümlerde ekleme zamanı vardır.

*_Layout. cshtml*içindeki satırınaçıklamasınıkaldırın:`@Html.Partial`

```cshtml
      <li>@Html.ActionLink("Contact", "Contact", "Home")</li>
    </ul>
    @*@Html.Partial("_LoginPartial")*@
  </div>
</div>
```

Şimdi *Görünümler/paylaşılan* klasöre *_loginpartial* adlı yeni bir Razor görünümü ekleyin:

*_Loginpartial. cshtml* dosyasını aşağıdaki kodla güncelleştirin (tüm içeriğini değiştirin):

```cshtml
@inject SignInManager<ApplicationUser> SignInManager
@inject UserManager<ApplicationUser> UserManager

@if (SignInManager.IsSignedIn(User))
{
    <form asp-area="" asp-controller="Account" asp-action="Logout" method="post" id="logoutForm" class="navbar-right">
        <ul class="nav navbar-nav navbar-right">
            <li>
                <a asp-area="" asp-controller="Manage" asp-action="Index" title="Manage">Hello @UserManager.GetUserName(User)!</a>
            </li>
            <li>
                <button type="submit" class="btn btn-link navbar-btn navbar-link">Log out</button>
            </li>
        </ul>
    </form>
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li><a asp-area="" asp-controller="Account" asp-action="Register">Register</a></li>
        <li><a asp-area="" asp-controller="Account" asp-action="Login">Log in</a></li>
    </ul>
}
```

Bu noktada, tarayıcıda siteyi yenileyebilmelisiniz.

## <a name="summary"></a>Özet

ASP.NET Core, ASP.NET Identity özelliklerinde yapılan değişiklikleri tanıtır. Bu makalede, ASP.NET Identity kimlik doğrulaması ve Kullanıcı Yönetimi özelliklerinin ASP.NET Core 'ye nasıl geçirileceğiyle karşılaşmış olursunuz.
