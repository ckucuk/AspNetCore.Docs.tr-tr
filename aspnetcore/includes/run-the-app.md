# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* Hata ayıklayıcı olmadan çalıştırmak için CTRL + F5 tuşlarına basın.

  [!INCLUDE[](~/includes/trustCertVS.md)]

  Visual Studio [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) başlar ve uygulamayı çalıştırır. Adres çubuğu `example.com`gibi değil `localhost:port#` gösterir. Bunun nedeni, `localhost` yerel bilgisayar için Standart ana bilgisayar adıdır. Localhost yalnızca yerel bilgisayardan Web isteklerine hizmet verir. Visual Studio bir web projesi oluşturduğunda, web sunucusu için rastgele bir bağlantı noktası kullanılır.
 
# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

  [!INCLUDE[](~/includes/trustCertVSC.md)]

* Hata ayıklayıcı olmadan çalıştırmak için **CTRL-F5** tuşlarına basın.

  Visual Studio Code, [Kestrel](xref:fundamentals/servers/kestrel)başlatır, bir tarayıcı başlatır ve `http://localhost:5001`gider. Adres çubuğu `example.com`gibi değil `localhost:port#` gösterir. Bunun nedeni, `localhost` yerel bilgisayar için Standart ana bilgisayar adıdır. Localhost yalnızca yerel bilgisayardan Web isteklerine hizmet verir.

  
# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Mac için Visual Studio](#tab/visual-studio-mac)

  [!INCLUDE[](~/includes/trustCertMac.md)]

* Visual Studio 'dan **opt-cmd-** hata ayıklayıcı olmadan çalışacak şekilde Döndür ' e basın. Alternatif olarak, menü çubuğuna gidin ve **hata ayıklama olmadan başlat > Çalıştır**' a gidin.

  Visual Studio, [Kestrel](xref:fundamentals/servers/kestrel)başlatır, bir tarayıcı başlatır ve `http://localhost:5001`gider.

<!-- End of VS tabs -->

---