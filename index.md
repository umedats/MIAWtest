<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Salesforce Embedded Messaging Debug Sample</title>
</head>
<body>
  <h1>Salesforce Embedded Messaging Debug Sample</h1>
  <p>
    複数 iframe (siteContextFrame, embeddedMessagingFrame, filePreviewFrame など) をまとめて削除できるよう改良したサンプルです。<br>
    再初期化後に <em>自動でチャットウィンドウ</em> を開く <code>openChat()</code> 呼び出しも追加しています。
  </p>

  <!-- ボタン -->
  <button onclick="destroyEmbeddedMessaging()">Force Close</button>
  <button onclick="reinitEmbeddedMessaging()">Reinit</button>

  <hr/>

  <script>
    /******************************************************************
     * 1) Embedded Messaging 初期化 (bootstrap.min.js の onload で呼ばれる)
     ******************************************************************/
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');
      try {
        // 言語設定 (例: 日本語)
        embeddedservice_bootstrap.settings.language = 'ja';

        // (A) 初期化 (OrgID, DeployID, EmbeddedServiceURL, オプション) : 書き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn', // 例
          'MIAW4',           // 例
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // 例
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initEmbeddedMessaging] SUCCESS: Chat initialized.');

        // (B) 再初期化後にチャットを即表示したい場合は openChat() を呼ぶ
        //     ただし全環境で動作保証されるわけではありません（Salesforceのバージョン依存）。
        if (embeddedservice_bootstrap &&
            typeof embeddedservice_bootstrap.openChat === 'function') {
          console.log('[initEmbeddedMessaging] Calling openChat() to auto-display the window...');
          embeddedservice_bootstrap.openChat();
        } else {
          console.log('[initEmbeddedMessaging] openChat() not found. The user may need to click the chat icon manually.');
        }

      } catch (err) {
        console.error('[initEmbeddedMessaging] ERROR loading Embedded Messaging:', err);
      }
      console.log('[initEmbeddedMessaging] END');
    }


    /******************************************************************
     * 2) Destroy: 強制クローズ処理
     *    - 複数の iframe (siteContextFrame, embeddedMessagingFrame, filePreviewFrame等) を全て削除
     *    - script, localStorage, window.embeddedservice_bootstrap も削除
     ******************************************************************/
    function destroyEmbeddedMessaging(verbose = true) {
      if (verbose) console.log('[destroyEmbeddedMessaging] START');

      // 2-1) removeIframe() があれば先に呼ぶ
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

      // 2-2) scriptタグ (bootstrap.min.js) を削除
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        if (verbose) console.log('[destroyEmbeddedMessaging] Removing script tag:', script.outerHTML);
        script.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed script tag.');
      } else {
        if (verbose) console.warn('[destroyEmbeddedMessaging] No script tag found (src*=bootstrap.min.js).');
      }

      // 2-3) すべての Embedded Messaging iframe を削除
      //      (siteContextFrame, embeddedMessagingFrame, filePreviewFrame など)
      const iframeSelector = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');

      const chatIframes = document.querySelectorAll(iframeSelector);
      if (chatIframes.length > 0) {
        chatIframes.forEach((iframe) => {
          if (verbose) console.log('[destroyEmbeddedMessaging] Deleting iframe:', iframe.outerHTML);
          iframe.remove();
        });
        if (verbose) console.log(`[destroyEmbeddedMessaging] Removed ${chatIframes.length} embedded messaging iframes.`);
      } else {
        if (verbose) console.warn('[destroyEmbeddedMessaging] No embedded messaging iframes found with selectors:', iframeSelector);
      }

      // 2-4) #embeddedMessaging コンテナを削除 (無い場合も多い)
      const container = document.getElementById('embeddedMessaging');
      if (container) {
        if (verbose) console.log('[destroyEmbeddedMessaging] Removing #embeddedMessaging:', container.outerHTML);
        container.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed #embeddedMessaging container.');
      } else {
        if (verbose) console.log('[destroyEmbeddedMessaging] #embeddedMessaging not found.');
      }

      // 2-5) localStorage のセッション情報を削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
        if (verbose) console.log('[destroyEmbeddedMessaging] Cleared localStorage for embeddedMessaging.');
      } catch (e) {
        console.warn('[destroyEmbeddedMessaging] Error clearing localStorage:', e);
      }

      // 2-6) window.embeddedservice_bootstrap を削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[destroyEmbeddedMessaging] Deleted window.embeddedservice_bootstrap.');
      } else {
        if (verbose) console.log('[destroyEmbeddedMessaging] No embeddedservice_bootstrap found on window.');
      }

      // 2-7) 削除後に残っている全 iframe をログ出力
      const remainingIframes = document.querySelectorAll('iframe');
      if (verbose) {
        console.log(`[destroyEmbeddedMessaging] Iframes left in DOM: ${remainingIframes.length}`);
        remainingIframes.forEach((frame, idx) => {
          console.log(`- Iframe #${idx}:`, frame, 'outerHTML =', frame.outerHTML);
        });
      }

      if (verbose) console.log('[destroyEmbeddedMessaging] END');
    }


    /******************************************************************
     * 3) Reinit: 再初期化 (スクリプト再挿入 → initEmbeddedMessaging())
     ******************************************************************/
    function reinitEmbeddedMessaging() {
      console.log('[reinitEmbeddedMessaging] START');
      // まず破棄
      destroyEmbeddedMessaging();

      // 少し待ってから script を再挿入
      setTimeout(() => {
        console.log('[reinitEmbeddedMessaging] Adding new script tag...');
        const scriptTag = document.createElement('script');
        scriptTag.type = 'text/javascript';
        // ここを自組織の URL に書き換えてください
        scriptTag.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';
        scriptTag.onload = function() {
          console.log('[reinitEmbeddedMessaging] Script loaded. Calling initEmbeddedMessaging()...');
          if (window.embeddedservice_bootstrap) {
            initEmbeddedMessaging();
          } else {
            console.warn('[reinitEmbeddedMessaging] embeddedservice_bootstrap not defined after script load.');
          }
        };
        scriptTag.onerror = function(e) {
          console.error('[reinitEmbeddedMessaging] ERROR loading new script:', e);
        };

        document.body.appendChild(scriptTag);
      }, 500);

      console.log('[reinitEmbeddedMessaging] END (waiting 500ms to re-load script)');
    }
  </script>

  <!-- 
       4) 初回ロード用の script
          onload で initEmbeddedMessaging() を呼ぶ 
  -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>
</body>
</html>
