<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Salesforce Embedded Messaging Debug Sample</title>
</head>
<body>
  <h1>Salesforce Embedded Messaging Debug Sample</h1>
  <p>
    このページでは、<strong>Salesforce Embedded Messaging</strong> を初期化し、<br>
    「Force Close」ボタンでチャットウィンドウ (iframe) を強制削除し、<br>
    「Reinit」ボタンで再度チャットを初期化するサンプルを示します。<br>
    削除処理後に詳細ログを出力するので、<em>iframe が本当に削除されたか</em>をコンソールでご確認ください。
  </p>

  <!-- ボタン -->
  <button onclick="destroyEmbeddedMessaging()">Force Close</button>
  <button onclick="reinitEmbeddedMessaging()">Reinit</button>

  <hr/>

  <script>
    /******************************************************************
     * 1) Embedded Messaging 初期化 (最初に読み込まれるスクリプトの onload で呼ばれる)
     ******************************************************************/
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');
      try {
        // 言語設定 (例: 日本語)
        embeddedservice_bootstrap.settings.language = 'ja';

        // ここを自組織の値に書き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',        // Org ID (例)
          'MIAW4',                 // Deployment ID (例)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // Embedded Service URL
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initEmbeddedMessaging] SUCCESS: Chat initialized.');
      } catch (err) {
        console.error('[initEmbeddedMessaging] ERROR loading Embedded Messaging:', err);
      }
      console.log('[initEmbeddedMessaging] END');
    }

    /******************************************************************
     * 2) 強制クローズ: destroyEmbeddedMessaging()
     *    - iframe, script, localStorage, embeddedservice_bootstrap を削除
     *    - 削除対象が見つかったらログに詳細を出力
     *    - 削除後に iframe が残っていないか全検索し、ログ出力
     ******************************************************************/
    function destroyEmbeddedMessaging(verbose = true) {
      if (verbose) console.log('[destroyEmbeddedMessaging] START');

      // 2-1) もし removeIframe() があれば先に呼ぶ (内部API)
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

      // 2-2) Embedded Messaging の <script> タグ (bootstrap.min.js) 削除
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        if (verbose) console.log('[destroyEmbeddedMessaging] Removing script tag:', script.outerHTML);
        script.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed script tag.');
      } else {
        if (verbose) console.warn('[destroyEmbeddedMessaging] No script tag found (src*=bootstrap.min.js).');
      }

      // 2-3) チャット iframe を直接削除
      const chatIframe = document.querySelector('iframe[data-embeddedmessaging], iframe[class*="embeddedMessaging"]');
      if (chatIframe) {
        if (verbose) console.log('[destroyEmbeddedMessaging] Deleting iframe:', chatIframe.outerHTML);
        chatIframe.remove();
        if (verbose) console.log('[destroyEmbeddedMessaging] Removed chat iframe.');
      } else {
        if (verbose) console.warn('[destroyEmbeddedMessaging] No chat iframe found with the given selector.');
      }

      // 2-4) チャット用コンテナ (#embeddedMessaging など) を削除
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

      // 2-7) 削除後に残っている iframe を全てログ表示
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
     * 3) Reinit: スクリプトを再読み込み後、initEmbeddedMessaging() を呼ぶ
     *    - destroyEmbeddedMessaging() で一度クリアした後に新しい <script> を挿入
     ******************************************************************/
    function reinitEmbeddedMessaging() {
      console.log('[reinitEmbeddedMessaging] START');
      // まず破棄
      destroyEmbeddedMessaging();

      // 少し待ってから script タグを再挿入
      setTimeout(() => {
        console.log('[reinitEmbeddedMessaging] Adding new script tag...');
        const scriptTag = document.createElement('script');
        scriptTag.type = 'text/javascript';
        // ここを自組織の bootstrap.min.js URL に書き換えてください
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
          onload で initEmbeddedMessaging() を呼び、チャットを初期化 
  -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>
</body>
</html>
