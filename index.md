<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Salesforce Embedded Service Chat Test</title>
</head>
<body>
  <h1>Salesforce Embedded Service Chat Test</h1>
  <p>
    こちらは、Salesforce の Embedded Service デプロイメントから
    取得したコードスニペットを埋め込んだテスト用のページです。
  </p>

  <!-- ボタン操作などでデモできるよう、destroy → reinit の処理を行うUIを用意 -->
  <button onclick="destroyEmbeddedMessaging()">Destroy Embedded Messaging</button>
  <button onclick="reinitEmbeddedMessaging()">Reinit Embedded Messaging</button>

  <script type="text/javascript">
    /**
     * 初期化用の関数。
     * Salesforce が提供する bootstrap.min.js が読み込まれたら呼び出される想定。
     */
    function initEmbeddedMessaging() {
      try {
        // 言語設定（例：日本語）
        embeddedservice_bootstrap.settings.language = 'ja';

        // 実際の init メソッド呼び出し
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn', // Org ID (例)
          'MIAW4',          // Deployment ID (例)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // Embedded Service URL
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );
        console.log('Embedded Messaging initialized.');
      } catch (err) {
        console.error('Error loading Embedded Messaging: ', err);
      }
    }

    /**
     * Embedded Messaging の DOM 要素やスクリプト、ローカルストレージを削除し、
     * セッション情報もクリアして「次回初期化を新しいセッション」として扱わせる。
     */
    function destroyEmbeddedMessaging() {
      // 1. スクリプトタグを削除 (src に 'bootstrap.min.js' を含むものを想定)
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        script.remove();
        console.log('Embedded Messaging script removed.');
      }

      // 2. Embedded Messaging が生成した要素を削除 (#embeddedMessaging など)
      const container = document.getElementById('embeddedMessaging');
      if (container) {
        container.remove();
        console.log('Embedded Messaging container removed.');
      }

      // 3. localStorage / sessionStorage に保存されている会話データを削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (e) {
        console.warn('Error clearing localStorage for embedded messaging:', e);
      }

      // 4. window オブジェクト上の embeddedservice_bootstrap を削除 (任意)
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        console.log('Deleted window.embeddedservice_bootstrap reference.');
      }

      console.log('Embedded Messaging destroyed (DOM/script/storage cleared).');
    }

    /**
     * Embedded Messaging を再度読み込む。
     * 破棄後にもう一度実行することで、新しいセッションとして初期化される想定。
     */
    function reinitEmbeddedMessaging() {
      console.log('Re-initializing Embedded Messaging...');

      // 1. 新しい <script> タグを生成
      const scriptTag = document.createElement('script');
      scriptTag.type = 'text/javascript';

      // 2. 以前と同じ bootstrap.min.js の URL
      scriptTag.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';

      // 3. 読み込み完了時に initEmbeddedMessaging() を呼ぶ
      scriptTag.onload = function() {
        console.log('Re-added Embedded Messaging script. Now calling init.');
        initEmbeddedMessaging();
      };

      // 4. body の最後に追加
      document.body.appendChild(scriptTag);
    }
  </script>

  <!-- 初回読み込み用のスクリプト -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>
</body>
</html>
