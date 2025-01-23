<html>

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
     * Embedded Messaging 初期化用の関数。
     * Salesforce が提供する bootstrap.min.js が読み込まれたら呼び出される想定。
     */
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');

      // 安全策: 万一の再初期化を防ぐため、既に存在したら削除
      if (window.embeddedservice_bootstrap) {
        console.warn('[initEmbeddedMessaging] embeddedservice_bootstrap already exists. Attempting to remove existing instance...');
        destroyEmbeddedMessaging(false); 
        // 第2引数で「ログを細かく出すか」切り替える例
      }

      try {
        if (!window.embeddedservice_bootstrap) {
          console.log('[initEmbeddedMessaging] embeddedservice_bootstrap is not defined yet. Proceeding normally.');
        }
        // 言語設定（例：日本語）
        // ※ Salesforceがサポートしている言語コード('ja','en','en-US'など)を指定
        embeddedservice_bootstrap.settings.language = 'ja';

        // 実際の init メソッド呼び出し
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn', // Org ID (例)
          'MIAW4',           // Deployment ID (例)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // Embedded Service URL
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initEmbeddedMessaging] SUCCESS: Embedded Messaging has been initialized.');
      } catch (err) {
        console.error('[initEmbeddedMessaging] ERROR loading Embedded Messaging: ', err);
      }

      console.log('[initEmbeddedMessaging] END');
    }

    /**
     * Embedded Messaging の DOM 要素やスクリプト、ローカルストレージを削除し、
     * セッション情報もクリアして「次回初期化を新しいセッション」として扱わせる。
     *
     * @param {boolean} verbose - ログの詳細出力をするかどうか(デフォルト: true)
     */
    function destroyEmbeddedMessaging(verbose = true) {
      if (verbose) console.log('[destroyEmbeddedMessaging] START');

      // 1. スクリプトタグを削除 (src に 'bootstrap.min.js' を含むものを想定)
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        script.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed Embedded Messaging script tag.');
      } else {
        if (verbose) console.warn('[destroyEmbeddedMessaging] No Embedded Messaging script found to remove.');
      }

      // 2. Embedded Messaging が生成した要素を削除 (#embeddedMessaging など)
      const container = document.getElementById('embeddedMessaging');
      if (container) {
        container.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed #embeddedMessaging container.');
      } else {
        if (verbose) console.warn('[destroyEmbeddedMessaging] No #embeddedMessaging container found.');
      }

      // 3. localStorage / sessionStorage に保存されている会話データを削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
        if (verbose) console.log('[destroyEmbeddedMessaging] Cleared localStorage for embeddedMessaging data.');
      } catch (e) {
        console.warn('[destroyEmbeddedMessaging] Error clearing localStorage for embedded messaging:', e);
      }

      // 4. window オブジェクト上の embeddedservice_bootstrap を削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[destroyEmbeddedMessaging] Deleted window.embeddedservice_bootstrap reference.');
      } else {
        if (verbose) console.log('[destroyEmbeddedMessaging] No embeddedservice_bootstrap found on window object.');
      }

      if (verbose) console.log('[destroyEmbeddedMessaging] END');
    }

    /**
     * Embedded Messaging を再度読み込む。
     * 破棄後にもう一度実行することで、新しいセッションとして初期化される想定。
     */
    function reinitEmbeddedMessaging() {
      console.log('[reinitEmbeddedMessaging] START');

      // 1. 既にあるかもしれないので念のためdestroy
      console.log('[reinitEmbeddedMessaging] First, destroying any existing instance...');
      destroyEmbeddedMessaging();

      // 2. 新しい <script> タグを生成
      const scriptTag = document.createElement('script');
      scriptTag.type = 'text/javascript';

      // 3. 以前と同じ bootstrap.min.js の URL を設定
      scriptTag.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';

      // 4. 読み込み完了時に initEmbeddedMessaging() を呼ぶ
      scriptTag.onload = function() {
        console.log('[reinitEmbeddedMessaging] Script loaded. Now calling initEmbeddedMessaging()...');
        initEmbeddedMessaging();
      };

      scriptTag.onerror = function(err) {
        console.error('[reinitEmbeddedMessaging] ERROR loading new script:', err);
      };

      // 5. body の最後に追加
      document.body.appendChild(scriptTag);

      console.log('[reinitEmbeddedMessaging] END (script appended, waiting for onload...).');
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
