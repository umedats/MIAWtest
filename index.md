<html>
<body>
  <h1>Salesforce Embedded Messaging Test</h1>
  <p>
    ページ読み込み時に一度だけ init を実行。<br>
    Destroy ボタンで破棄し、Reinit ボタンで再初期化できます。
  </p>

  <!-- 操作用のボタン -->
  <button onclick="destroyEmbeddedMessaging()">Destroy (Force Close)</button>
  <button onclick="reinitEmbeddedMessaging()">Reinit Embedded Messaging</button>

  <script>
    /**
     * ページ読み込み時 (script onload) に呼ばれる関数。
     * - Salesforceのbootstrap.min.js が読み込まれたら実行
     * - embeddedservice_bootstrap が定義済みかチェックしてから init を試みる
     */
    function initEmbeddedMessagingAtPageLoad() {
      console.log('[initEmbeddedMessagingAtPageLoad] START - Script loaded.');

      if (window.embeddedservice_bootstrap) {
        console.log('[initEmbeddedMessagingAtPageLoad] embeddedservice_bootstrap found. Calling doInit()...');
        doInit();
      } else {
        console.warn('[initEmbeddedMessagingAtPageLoad] No embeddedservice_bootstrap yet. Retrying in 1 second...');
        setTimeout(function() {
          if (window.embeddedservice_bootstrap) {
            console.log('[initEmbeddedMessagingAtPageLoad] Retrying doInit() after delay...');
            doInit();
          } else {
            console.error('[initEmbeddedMessagingAtPageLoad] Still no embeddedservice_bootstrap after 1 second. Aborting init.');
          }
        }, 1000);
      }
    }

    /**
     * 実際に Embedded Messaging を初期化する関数。
     * - embeddedservice_bootstrap.settings の設定
     * - embeddedservice_bootstrap.init(...) の呼び出し
     */
    function doInit() {
      console.log('[doInit] START');
      try {
        // 言語設定など
        embeddedservice_bootstrap.settings.language = 'ja';

        // ★ 実際の Org ID / Deployment ID / URL に書き換えてください
        embeddedservice_bootstrap.init(
          '00Dxxxxxxxxxxxx',  // Org ID
          'MIAWxxxxxxxxxxxx', // Deployment ID
          'https://xxx.my.site.com/ESWxxx', // Embedded Service URL
          {
            scrt2URL: 'https://xxx.my.salesforce-scrt.com'
          }
        );

        console.log('[doInit] SUCCESS: Embedded Messaging initialized.');
      } catch (e) {
        console.error('[doInit] ERROR:', e);
      }
      console.log('[doInit] END');
    }

    /**
     * Embedded Messaging を破棄してチャットウィンドウを強制的に閉じる関数。
     * - iframe や script、localStorage、window オブジェクト等を除去
     */
    function destroyEmbeddedMessaging(verbose = true) {
      if (verbose) console.log('[destroyEmbeddedMessaging] START');

      // もし内部の removeIframe() メソッドがあれば、それを先に呼ぶ（任意）
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        if (verbose) console.log('[destroyEmbeddedMessaging] Calling removeIframe()...');
        try {
          window.embeddedservice_bootstrap.core.removeIframe();
        } catch (err) {
          console.warn('[destroyEmbeddedMessaging] removeIframe() threw error:', err);
        }
      }

      // 1. script タグ (bootstrap.min.js) を削除
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        script.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed Embedded Messaging script tag.');
      }

      // 2. iframe を直接削除 (data-embeddedmessaging など属性で検索)
      const chatIframe = document.querySelector('iframe[data-embeddedmessaging], iframe[class*="embeddedMessaging"]');
      if (chatIframe) {
        chatIframe.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed chat iframe.');
      }

      // 3. コンテナ (#embeddedMessaging) を削除
      const container = document.getElementById('embeddedMessaging');
      if (container) {
        container.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed #embeddedMessaging container.');
      }

      // 4. localStorage をクリア
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
        if (verbose) console.log('[destroyEmbeddedMessaging] Cleared localStorage for embeddedMessaging data.');
      } catch (e) {
        console.warn('[destroyEmbeddedMessaging] Error clearing localStorage:', e);
      }

      // 5. window.embeddedservice_bootstrap を削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[destroyEmbeddedMessaging] Deleted embeddedservice_bootstrap from window.');
      }

      if (verbose) console.log('[destroyEmbeddedMessaging] END');
    }

    /**
     * 再初期化 (Reinit) ボタンで呼び出す関数。
     * - まず破棄(destroy) → 少し待って → 新しい script を読み込む → onload で doInit()
     */
    function reinitEmbeddedMessaging() {
      console.log('[reinitEmbeddedMessaging] START');
      // 破棄
      destroyEmbeddedMessaging();

      // 少し待ってから再ロード (setTimeoutは例示的に500ms, 必要なら調整)
      setTimeout(function() {
        console.log('[reinitEmbeddedMessaging] Adding a new script tag...');

        // 新しい <script> タグを追加
        const scriptTag = document.createElement('script');
        scriptTag.type = 'text/javascript';
        // ここも実際の bootstrap.min.js の URL に置き換えてください
        scriptTag.src = 'https://xxx.my.site.com/ESWxxx/assets/js/bootstrap.min.js';

        // script がロードされたら再度 doInit()
        scriptTag.onload = function() {
          console.log('[reinitEmbeddedMessaging] Script loaded. Now calling doInit()...');
          if (window.embeddedservice_bootstrap) {
            doInit();
          } else {
            console.warn('[reinitEmbeddedMessaging] embeddedservice_bootstrap not defined after loading script.');
          }
        };

        scriptTag.onerror = function(e) {
          console.error('[reinitEmbeddedMessaging] ERROR loading new script:', e);
        };

        document.body.appendChild(scriptTag);
      }, 500);

      console.log('[reinitEmbeddedMessaging] END (waiting for setTimeout...)');
    }
  </script>

  <!-- 初回のみ読み込む Script。onload で initEmbeddedMessagingAtPageLoad() を呼ぶ -->
  <script
    type="text/javascript"
    src="https://xxx.my.site.com/ESWxxx/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessagingAtPageLoad()"
  ></script>
</body>
</html>
