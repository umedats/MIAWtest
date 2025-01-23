<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Force Reopen Chat with openChat()</title>
</head>
<body>
  <h1>Force Reopen Chat with openChat()</h1>
  <p>
    以下のボタンで、終了済みセッションでも強制的にチャットウィンドウを<br>
    新しいセッションとして再表示します。ランチャーが <code>display:none</code> でも、<br>
    <code>openChat()</code> を呼ぶことでウィンドウを直接開くように実装しています。
  </p>

  <!-- ボタン群 -->
  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="reinitAndOpenChat()">Reinit &amp; Open Chat</button>

  <hr/>

  <script>
    /********************************************************************
     * 1) initChat() - Embedded Messaging 初期化 → openChat() で強制表示
     ********************************************************************/
    function initChat() {
      console.log('[initChat] START');
      try {
        // 言語設定 (例: 'ja')
        embeddedservice_bootstrap.settings.language = 'ja';

        // 組織ID / デプロイID / Embedded Service URL を書き換える
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',    // Org ID (例)
          'MIAW4',             // Deployment ID (例)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // ES URL (例)
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initChat] SUCCESS: Chat initialized.');
      } catch (err) {
        console.error('[initChat] ERROR:', err);
      }
      console.log('[initChat] END');

      // ★ init完了後にウィンドウを強制的に開く
      setTimeout(() => {
        if (
          window.embeddedservice_bootstrap &&
          typeof embeddedservice_bootstrap.openChat === 'function'
        ) {
          console.log('[initChat] Calling openChat() to force window display...');
          embeddedservice_bootstrap.openChat();
        } else {
          console.warn('[initChat] openChat() not available; user may need to click an icon.');
        }
      }, 500);
    }

    /********************************************************************
     * 2) removeChat() - 既存のチャット要素をすべて削除
     ********************************************************************/
    function removeChat(verbose = true) {
      if (verbose) console.log('[removeChat] START');

      // removeIframe() があれば先に呼ぶ
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        if (verbose) console.log('[removeChat] removeIframe()...');
        try {
          window.embeddedservice_bootstrap.core.removeIframe();
        } catch(e) {
          console.warn('[removeChat] removeIframe error:', e);
        }
      }

      // scriptタグ (bootstrap.min.js) を削除
      const scriptTag = document.querySelector("script[src*='bootstrap.min.js']");
      if (scriptTag) {
        scriptTag.remove();
        if (verbose) console.log('[removeChat] Removed script tag.');
      }

      // iframe を削除
      const iframeSelectors = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');
      const allIframes = document.querySelectorAll(iframeSelectors);
      allIframes.forEach(ifr => {
        if (verbose) console.log('[removeChat] Deleting iframe:', ifr.outerHTML);
        ifr.remove();
      });

      // localStorage のチャット情報を削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (err) {
        console.warn('[removeChat] localStorage remove error:', err);
      }

      // JSオブジェクト embeddedservice_bootstrap を削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[removeChat] Deleted window.embeddedservice_bootstrap.');
      }

      if (verbose) console.log('[removeChat] END');
    }

    /********************************************************************
     * 3) loadChatScript() - 新しい script を読み込み、onload で initChat()
     ********************************************************************/
    function loadChatScript() {
      console.log('[loadChatScript] START');
      const scriptEl = document.createElement('script');
      scriptEl.type = 'text/javascript';

      // 実際の bootstrap.min.js のURLに書き換える
      scriptEl.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';

      scriptEl.onload = () => {
        console.log('[loadChatScript] Script loaded. Now calling initChat()...');
        if (window.embeddedservice_bootstrap) {
          initChat(); // ← init 後に openChat() を呼ぶ
        } else {
          console.warn('[loadChatScript] embeddedservice_bootstrap not defined after script load.');
        }
      };
      scriptEl.onerror = (err) => {
        console.error('[loadChatScript] Script load error:', err);
      };

      document.body.appendChild(scriptEl);
      console.log('[loadChatScript] END');
    }

    /********************************************************************
     * 4) reinitAndOpenChat() - removeChat() → loadChatScript()
     ********************************************************************/
    function reinitAndOpenChat() {
      console.log('[reinitAndOpenChat] START');
      removeChat(false);

      setTimeout(() => {
        loadChatScript();
      }, 300);

      console.log('[reinitAndOpenChat] END');
    }

    /********************************************************************
     * 5) onInitialLoad()
     *    - ページロード時に何もしない例。最初からチャット開きたい場合は initChat() を呼ぶ
     ********************************************************************/
    function onInitialLoad() {
      console.log('[onInitialLoad] No auto-init. Use buttons to control chat.');
      // initChat(); // ← もしページ読み込み時からウィンドウを表示したいならコメントアウトを解除
    }
  </script>

  <!-- ページ読み込み時に onInitialLoad() を呼ぶだけ -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="onInitialLoad()"
  ></script>
</body>
</html>
