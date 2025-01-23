<html>

<body>
  <h1>Salesforce Embedded Messaging Test</h1>
  <p>初期ロード時に一度だけ init。Destroy ボタンで破棄、Reinit ボタンで再初期化します。</p>

  <button onclick="destroyEmbeddedMessaging()">Destroy (Force Close)</button>
  <button onclick="reinitEmbeddedMessaging()">Reinit Embedded Messaging</button>

  <script>
    /**
     * ページ読み込み時 (script onload) に呼ばれる関数。
     * embeddedservice_bootstrap が既に存在すれば -> 破棄して終了
     * 存在しなければ -> doInit() で初期化
     */
    function initEmbeddedMessagingAtPageLoad() {
      console.log('[initEmbeddedMessagingAtPageLoad] START');

      if (window.embeddedservice_bootstrap) {
        console.warn('[initEmbeddedMessagingAtPageLoad] Found existing embeddedservice_bootstrap. Destroying...');
        destroyEmbeddedMessaging();
        // destroy で scriptタグやブートストラップオブジェクトを消すので、
        // ここではすぐに init を呼ばず、いったん終了。
        // 必要であれば setTimeout で再init してもOK。
        return;
      }

      // まだ embeddedservice_bootstrap がなければ init
      doInit();
    }

    /**
     * Embedded Messaging の本来の init 処理
     */
    function doInit() {
      console.log('[doInit] START');
      try {
        // language設定
        embeddedservice_bootstrap.settings.language = 'ja';

        // 実際の init(OrgID, DeployID, URL, {options...}) を書き換える
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
      console.log('[doInit] END');
    }

    /**
     * Embedded Messaging の破棄
     */
    function destroyEmbeddedMessaging(verbose = true) {
      if (verbose) console.log('[destroyEmbeddedMessaging] START');

      // removeIframe() が使えるなら先に呼ぶ (任意)
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

      // iframe (data-embeddedmessaging) を削除
      const chatIframe = document.querySelector('iframe[data-embeddedmessaging], iframe[class*="embeddedMessaging"]');
      if (chatIframe) {
        chatIframe.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed chat iframe.');
      }

      // コンテナID(#embeddedMessaging) があれば削除
      const container = document.getElementById('embeddedMessaging');
      if (container) {
        container.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed #embeddedMessaging container.');
      }

      // localStorage削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
        if (verbose) console.log('[destroyEmbeddedMessaging] Cleared localStorage.');
      } catch (err) {
        console.warn('[destroyEmbeddedMessaging] Error clearing localStorage:', err);
      }

      // windowオブジェクト上の embeddedservice_bootstrap を削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[destroyEmbeddedMessaging] Deleted window.embeddedservice_bootstrap.');
      }

      if (verbose) console.log('[destroyEmbeddedMessaging] END');
    }

    /**
     * 再初期化
     * - いったん destroy
     * - 少し待って新しい <script> を読み込み -> onload で doInit()
     */
    function reinitEmbeddedMessaging() {
      console.log('[reinitEmbeddedMessaging] START');
      destroyEmbeddedMessaging();

      // 破棄完了後にセットタイムアウトなどで init する
      setTimeout(() => {
        console.log('[reinitEmbeddedMessaging] Adding new script...');
        const scriptTag = document.createElement('script');
        scriptTag.type = 'text/javascript';
        scriptTag.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js'; // 実際のURLに合わせる
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

      console.log('[reinitEmbeddedMessaging] END (waiting 500ms before loading new script)');
    }
  </script>

  <!-- 
    初回のみ読み込むScript。
    onload で initEmbeddedMessagingAtPageLoad() を呼ぶことで
    ページロード時に一度だけ init を実行
  -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessagingAtPageLoad()"
  ></script>
</body>
</html>
