<html>

<body>
  <h1>Salesforce Embedded Service Chat Test</h1>
  <p>
    こちらは、Salesforce の Embedded Service デプロイメントから取得したコードスニペットを埋め込み、強制的に閉じたり再初期化する機能を追加したサンプルです。
  </p>

  <!-- ボタン群 -->
  <button onclick="destroyEmbeddedMessaging()">Destroy / Force Close</button>
  <button onclick="reinitEmbeddedMessaging()">Reinit Embedded Messaging</button>
  <button onclick="forceCloseChatIframeOnly()">Close Iframe (removeIframe)</button>

  <script type="text/javascript">
    /**
     * 1) 初期ロード時に呼ばれる関数。
     *    Salesforce が提供する bootstrap.min.js が onload したら呼び出される。
     */
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');

      // 念のため、既に存在する場合は破棄
      if (window.embeddedservice_bootstrap) {
        console.warn('[initEmbeddedMessaging] Found existing embeddedservice_bootstrap. Destroying first...');
        destroyEmbeddedMessaging(false); // verbose=false でログ控えめ
      }

      try {
        // このオブジェクトが存在しない場合、bootstrap.min.js がまだ読み込まれていない
        if (!window.embeddedservice_bootstrap) {
          console.log('[initEmbeddedMessaging] No embeddedservice_bootstrap found yet, proceeding with normal init...');
        }

        // 言語設定 (例: 'ja' や 'en-US' など)
        embeddedservice_bootstrap.settings.language = 'ja';

        // initメソッド呼び出し
        // ここは実際の組織ID / デプロイID / リリースURLに合わせて書き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn', // Org ID (ダミー)
          'MIAW4',           // Deployment ID (ダミー)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // Embedded Service URL (例)
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initEmbeddedMessaging] SUCCESS: Embedded Messaging initialized.');
      } catch (err) {
        console.error('[initEmbeddedMessaging] ERROR:', err);
      }

      console.log('[initEmbeddedMessaging] END');
    }

    /**
     * 2) 強制クローズ用の関数。
     *    - Embedded Messaging の iframe やスクリプト、localStorage のセッション情報を削除
     *    - 後から reinit すると「新しいセッション」として扱われる
     * @param {boolean} verbose - 詳細ログを出すかどうか
     */
    function destroyEmbeddedMessaging(verbose = true) {
      if (verbose) console.log('[destroyEmbeddedMessaging] START');

      // 試しに内部の removeIframe() メソッドがあれば使う
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        if (verbose) console.log('[destroyEmbeddedMessaging] Calling embeddedservice_bootstrap.core.removeIframe()...');
        try {
          window.embeddedservice_bootstrap.core.removeIframe();
        } catch (err) {
          if (verbose) console.warn('[destroyEmbeddedMessaging] removeIframe() threw error:', err);
        }
      } else {
        if (verbose) console.log('[destroyEmbeddedMessaging] removeIframe() method not found, will remove DOM manually.');
      }

      // 1. <script> タグの削除 (src に 'bootstrap.min.js' を含むものを想定)
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        script.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed Embedded Messaging script tag.');
      } else {
        if (verbose) console.warn('[destroyEmbeddedMessaging] No script tag found.');
      }

      // 2. iFrameを直接探して削除 (data-embeddedmessaging など属性を確認)
      const chatIframe = document.querySelector('iframe[data-embeddedmessaging], iframe[class*="embeddedMessaging"]');
      if (chatIframe) {
        chatIframe.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed chat iframe.');
      } else {
        if (verbose) console.log('[destroyEmbeddedMessaging] No chat iframe found.');
      }

      // 3. Embedded Messaging コンテナ div (#embeddedMessaging) 削除
      const container = document.getElementById('embeddedMessaging');
      if (container) {
        container.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed #embeddedMessaging container.');
      }

      // 4. localStorage に保存されているセッション情報を削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
        if (verbose) console.log('[destroyEmbeddedMessaging] Cleared localStorage for embeddedMessaging data.');
      } catch (e) {
        console.warn('[destroyEmbeddedMessaging] Error clearing localStorage:', e);
      }

      // 5. window オブジェクト上の embeddedservice_bootstrap を削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[destroyEmbeddedMessaging] Deleted window.embeddedservice_bootstrap reference.');
      } else {
        if (verbose) console.log('[destroyEmbeddedMessaging] No embeddedservice_bootstrap found on window.');
      }

      if (verbose) console.log('[destroyEmbeddedMessaging] END');
    }

    /**
     * 3) 再初期化の関数。
     *    destroyEmbeddedMessaging() で完全に破棄したあと、新しい <script> を埋め込んで init する。
     */
    function reinitEmbeddedMessaging() {
      console.log('[reinitEmbeddedMessaging] START');

      // 一応前のインスタンスを破棄
      console.log('[reinitEmbeddedMessaging] Destroying existing instance if any...');
      destroyEmbeddedMessaging();

      // 新たな <script> タグを作成し、読み込みが完了したら init を実行
      const scriptTag = document.createElement('script');
      scriptTag.type = 'text/javascript';
      scriptTag.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';

      scriptTag.onload = function() {
        console.log('[reinitEmbeddedMessaging] Script loaded. Now calling initEmbeddedMessaging()...');
        initEmbeddedMessaging();
      };

      scriptTag.onerror = function(err) {
        console.error('[reinitEmbeddedMessaging] ERROR loading new script:', err);
      };

      // body の末尾に追加
      document.body.appendChild(scriptTag);

      console.log('[reinitEmbeddedMessaging] END (script appended, waiting for onload...).');
    }

    /**
     * 4) 「iframe を閉じる」だけ試す関数。
     *    removeIframe() があれば利用し、なければ DOM 上の iframe を削除する。
     *    destroyEmbeddedMessaging() ほどセッション情報を消さないので、次に開くと前の会話が継続する可能性あり。
     */
    function forceCloseChatIframeOnly() {
      console.log('[forceCloseChatIframeOnly] Attempting to close chat iframe.');

      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        console.log('[forceCloseChatIframeOnly] Using removeIframe().');
        window.embeddedservice_bootstrap.core.removeIframe();
      } else {
        console.warn('[forceCloseChatIframeOnly] removeIframe() not found, removing iframe manually.');
        const chatIframe = document.querySelector('iframe[data-embeddedmessaging], iframe[class*="embeddedMessaging"]');
        if (chatIframe) {
          chatIframe.remove();
          console.log('[forceCloseChatIframeOnly] Removed chat iframe directly.');
        } else {
          console.log('[forceCloseChatIframeOnly] No chat iframe found.');
        }
      }
    }
  </script>

  <!-- 初回読み込み用のスクリプト: ページ読み込み時に自動 init -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>
</body>
</html>
