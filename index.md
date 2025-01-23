
<html >
<body>
  <h1>Salesforce Embedded Messaging Test</h1>
  <p>
    初期ロード時に一度だけ init。<br>
    Destroy ボタンで破棄し、Reinit ボタンで再初期化できます。
  </p>

  <!-- 破棄・再初期化用のボタン -->
  <button onclick="destroyEmbeddedMessaging()">Destroy (Force Close)</button>
  <button onclick="reinitEmbeddedMessaging()">Reinit Embedded Messaging</button>

  <script>
    /**
     * ページ読み込み時 (script onload) に呼ばれる関数。
     * もし embeddedservice_bootstrap が既に存在すれば -> 破棄→再初期化
     * 存在しなければ -> doInit() で初期化
     */
    function initEmbeddedMessagingAtPageLoad() {
      console.log('[initEmbeddedMessagingAtPageLoad] START');

      if (window.embeddedservice_bootstrap) {
        console.warn('[initEmbeddedMessagingAtPageLoad] Found existing embeddedservice_bootstrap. Destroying...');

        // 既存のブートストラップを破棄
        destroyEmbeddedMessaging();

        // 少し待ってから、新しい <script> を読み込み & doInit()
        setTimeout(() => {
          console.log('[initEmbeddedMessagingAtPageLoad] Now reinitializing after destroy...');

          const scriptTag = document.createElement('script');
          scriptTag.type = 'text/javascript';
          // 実際の bootstrap.min.js の URL に合わせてください
          scriptTag.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';
          
          scriptTag.onload = function() {
            console.log('[initEmbeddedMessagingAtPageLoad] Script reloaded. Calling doInit()...');
            if (window.embeddedservice_bootstrap) {
              doInit();
            } else {
              console.warn('[initEmbeddedMessagingAtPageLoad] embeddedservice_bootstrap not defined after script load.');
            }
          };

          document.body.appendChild(scriptTag);
        }, 500);

      } else {
        // まだ存在しない → 今回が初回ロードなので直接 init
        doInit();
      }
    }

    /**
     * Embedded Messaging の本来の init 処理
     */
    function doInit() {
      console.log('[doInit] START');
      try {
        // language 設定
        embeddedservice_bootstrap.settings.language = 'ja';

        // 組織ID / デプロイID / URL を実際の値に置き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',  // Org ID の例
          'MIAW4',           // Deployment ID の例
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // Embedded Service URL
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );
        console.log('[doInit] SUCCESS: Embedded Messaging initialized.');
      } catch (err) {
        console.error('[doInit] ERROR loading Embedded Messaging:', err);
      }
      console.log('[doInit] END');
    }

    /**
     * Embedded Messaging の破棄
     *  - iframe / script / localStorage / window.embeddedservice_bootstrap を削除
     */
    function destroyEmbeddedMessaging(verbose = true) {
      if (verbose) console.log('[destroyEmbeddedMessaging] START');

      // removeIframe() が使えるなら先に呼ぶ
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

      // script タグ削除
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        script.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed script tag.');
      }

      // iframe を直接削除
      const chatIframe = document.querySelector('iframe[data-embeddedmessaging], iframe[class*="embeddedMessaging"]');
      if (chatIframe) {
        chatIframe.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed chat iframe.');
      }

      // コンテナ (#embeddedMessaging) があれば削除
      const container = document.getElementById('embeddedMessaging');
      if (container) {
        container.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed #embeddedMessaging container.');
      }

      // localStorage のセッション情報をクリア
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
        if (verbose) console.log('[destroyEmbeddedMessaging] Cleared localStorage for embeddedMessaging.');
      } catch (e) {
        console.warn('[destroyEmbeddedMessaging] Error clearing localStorage:', e);
      }

      // window.embeddedservice_bootstrap を削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[destroyEmbeddedMessaging] Deleted window.embeddedservice_bootstrap.');
      }

      if (verbose) console.log('[destroyEmbeddedMessaging] END');
    }

    /**
     * 再初期化
     *  1) destroyEmbeddedMessaging()
     *  2) setTimeout で少し待ってから新しい script を読み込み
     *  3) onload で doInit()
     */
    function reinitEmbeddedMessaging() {
      console.log('[reinitEmbeddedMessaging] START');
      destroyEmbeddedMessaging();

      setTimeout(() => {
        console.log('[reinitEmbeddedMessaging] Adding new script...');
        const scriptTag = document.createElement('script');
        scriptTag.type = 'text/javascript';
        // 実際のURLに合わせる
        scriptTag.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';

        scriptTag.onload = function() {
          console.log('[reinitEmbeddedMessaging] Script loaded. Now calling doInit()...');
          if (window.embeddedservice_bootstrap) {
            doInit();
          } else {
            console.warn('[reinitEmbeddedMessaging] embeddedservice_bootstrap not defined after script load.');
          }
        };

        document.body.appendChild(scriptTag);
      }, 500);

      console.log('[reinitEmbeddedMessaging] END (waiting 500ms...)');
    }
  </script>

  <!--
    初回ロード時の Script
    onload で initEmbeddedMessagingAtPageLoad() を呼ぶ
  -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessagingAtPageLoad()"
  ></script>
</body>
</html>
