# Android SSL 证书校验（未验证）



客户端验证 SSL 一般通过 SSLContext 方式创建（适用于 HttpsURLConnection请求）。



示例代码：

```java
public class HttpUtil {
	private static final String KEY_STORE_TYPE_P12 = "PKCS12";//证书类型 固定值
  private static final String KEY_STORE_CLIENT_PATH = "client.p12";//客户端要给服务器端认证的证书
  private static final String KEY_STORE_SERVER_PATH = "server.crt";//客户端验证服务器端的证书库
  private static final String KEY_STORE_PASSWORD = "123456";// 客户端证书密码
    
	/**
    * 获取SSLContext
    *
    * @param context 上下文
    * @return SSLContext
    */
  public static SSLContext getSSLContext(Context context) {
    try {
      //参考 https://developer.android.com/training/articles/security-ssl.html
      CertificateFactory  certificateFactory = CertificateFactory.getInstance("X.509");
      //客户端验证服务器端的证书库，使用 server.crt 文件的形式
      InputStream serverIn = context.getAssets().open(KEY_STORE_SERVER_PATH); 
      //通过流创建一个证书
      Certificate cer = certificateFactory.generateCertificate(serverIn);  
      //创建一个证书库，并将证书导入证书库
      KeyStore trustStore = KeyStore.getInstance(KeyStore.getDefaultType()); 
      //参数 null 表示使用系统默认 keystore，也可使用其他 keystore（需事先将 server.crt 证书导入keystore 里）
      trustStore.load(null, null);
      trustStore.setCertificateEntry("trust", cer);  
        	     
      //服务器端验证客户端的证书
      InputStream clientIn = context.getResources().getAssets().open(KEY_STORE_CLIENT_PATH);
      KeyStore clientKeyStore = KeyStore.getInstance(KEY_STORE_TYPE_P12);
      
      try {
        clientKeyStore.load(clientIn, KEY_STORE_PASSWORD.toCharArray());
      } catch (Exception e) {
        Log.e("Exception", e.getMessage(), e);
      } finally {
        try {
          clientIn.close();
        } catch (Exception e) {}
        try {
         	serverIn.close();
        } catch (Exception e) {  }
      }        
      
      TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
      trustManagerFactory.init(trustStore);
           
      KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance("X509");
      keyManagerFactory.init(keyStore, KEY_STORE_PASSWORD.toCharArray());
      
      //使用传输层安全协议TLS(transport layer secure)
      SSLContext sslContext = SSLContext.getInstance("TLS");
      sslContext.init(keyManagerFactory.getKeyManagers(), trustManagerFactory.getTrustManagers(), null); //param3: new SecureRandom()
      return sslContext;
    } catch (Exception e) {
      Log.e("tag", e.getMessage(), e);
    }
    return null;
}
```



server.crt 上面代码是放到 Android 项目 assets 资源目录下的，容易被反编译取出。

在代码中可以将 server.crt 转为字符串代替：

```java
keytool -printcert -rfc -file server.crt
```

则下面代码：

```java
InputStream inputStream = context.getAssets().open(KEY_STORE_SERVER_PATH);
```

改为：

```
ByteArrayInputStream certInputStream = new ByteArrayInputStream(CERT_RFC.getBytes());
```

CERT_RFC 为 server.crt 字符串形式。



> 参考
>
> [Android HTTPS 双向认证实现]([http://zhangyida.cn/2017/12/13/Android%20HTTPS%20%E5%8F%8C%E5%90%91%E8%AE%A4%E8%AF%81%E5%AE%9E%E7%8E%B0/](http://zhangyida.cn/2017/12/13/Android HTTPS 双向认证实现/))
>
> [HTTPS工作原理以及Android中如何防止抓包](https://juejin.im/post/5d8c593ee51d45783544b9ac)
>
> [Android安全加密：Https编程]([https://jackchan1999.github.io/2017/05/01/%E5%AE%89%E5%85%A8%E5%8A%A0%E5%AF%86/Android%E5%AE%89%E5%85%A8%E5%8A%A0%E5%AF%86%EF%BC%9AHttps%E7%BC%96%E7%A8%8B/](https://jackchan1999.github.io/2017/05/01/安全加密/Android安全加密：Https编程/))

