<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Embedded Service Chat Test</title>
</head>
<body>
  <h1>Salesforce Embedded Service Chat Test</h1>
  <p>
    こちらは、Salesforce の Embedded Service デプロイメントから
    取得したコードスニペットを埋め込んだテスト用のページです。
  </p>

  <!-- 
    ここから下の <script> タグは、取得したコードスニペットを
    そのまま貼り付けたものです。
    （実際の環境によってはデプロイメントIDなどが異なる場合があります）
  -->

  <script type='text/javascript'>
    function initEmbeddedMessaging() {
      try {
        embeddedservice_bootstrap.settings.language = 'ja'; // 言語設定

        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',  // 組織やサイト固有のID
          'Test_Embedded_Chat', // デプロイメント名
          'https://daihachi20240927.my.site.com/ESWTestEmbeddedChat1737513492641', // デプロイメントURL
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );
      } catch (err) {
        console.error('Error loading Embedded Messaging: ', err);
      }
    };
  </script>

  <script
    type='text/javascript'
    src='https://daihachi20240927.my.site.com/ESWTestEmbeddedChat1737513492641/assets/js/bootstrap.min.js'
    onload='initEmbeddedMessaging()'>
  </script>
</body>
</html>
