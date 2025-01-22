<html lang="ja">

<body>
  <h1>Salesforce Embedded Service Chat Test</h1>
  <p>
    こちらは、Salesforce の Embedded Service デプロイメントから
    取得したコードスニペットを埋め込んだテスト用のページです。
  </p>

  <script type='text/javascript'>
    function initEmbeddedMessaging() {
      try {
        embeddedservice_bootstrap.settings.language = 'ja'; // For example, enter 'en' or 'en-US'
  
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',
          'MIAW4',
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136',
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );
      } catch (err) {
        console.error('Error loading Embedded Messaging: ', err);
      }
    };
  </script>
  <script type='text/javascript' src='https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js' onload='initEmbeddedMessaging()'></script>
  

</body>
</html>
