<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Manual Remove/Reopen Chat (No MutationObserver)</title>
</head>
<body>
  <h1>Manual Remove/Reopen Chat (No MutationObserver)</h1>
  <p>
    このサンプルは、<strong>チャットを自動で閉じる仕組み (MutationObserver)</strong> を削除し、<br>
    「Remove Chat」「Reopen Chat」ボタンを手動で押すだけでチャットを制御します。<br>
    ランチャーアイコンがない環境でも <code>openChat()</code> を呼び出してウィンドウを自動表示します。
  </p>

  <!-- 手動操作ボタン -->
  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="reopenChat()">Reopen Chat</button>

  <hr/>

  <script>
    /*****************************************
     * 1) initChat() - チャット初期化関数
     *    - init 後に openChat() を呼んで自動でウィンドウを展開
     *****************************************/
    function initChat() {
      console.log('[initChat] START');
      try {
        // 言語設定（例：日本語）
        embeddedservice_bootstrap.settings.language = 'ja';

        // 組織ID / デプロイID / Embedded Service URL を実際の環境に合わせる
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',   // Org ID
          'MIAW4',            // Deployment ID
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136',
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initChat] SUCCESS: Chat initialized.');
      } catch (err) {
        console.error('[initChat] ERROR:', err);
      }
      console.log('[initChat] END');

      // ここで強制的にチャットウィンドウを開く
      setTimeout(() => {
        if (
          window.embeddedservice_bootstrap &&
          typeof embeddedservice_bootstrap.openChat === 'function'
        ) {
          console.log('[initChat] Calling embeddedservice_bootstrap.openChat() to auto-show window...');
          embeddedservice_bootstrap.openChat();
        } else {
          console.warn('[initChat] openChat() not found. The user may need to click a chat icon (if any).');
        }
      }, 500);
    }

    /*****************************************
     * 2) removeChat() - チャット要素を削除
     *****************************************/
    function removeChat(verbose = true) {
      if (verbose) console.log('[removeChat] START');

      // removeIframe() があれば呼ぶ
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

      // scriptタグ削除 (bootstrap.min.js)
      const scriptTag = document.querySelector("script[src*='bootstrap.min.js']");
      if (scriptTag) {
        scriptTag.remove();
        if (verbose) console.log('[removeChat] Removed script tag.');
      }

      // iframe削除
      const iframeSelectors = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');

      const chatIframes = document.querySelectorAll(iframeSelectors);
      chatIframes.forEach((ifr) => {
        if (verbose) console.log('[removeChat] Deleting iframe:', ifr.outerHTML);
        ifr.remove();
      });

      // localStorage のチャットセッション情報を削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (err) {
        console.warn('[removeChat] localStorage remove error:', err);
      }

      // embeddedservice_bootstrap オブジェクト削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[removeChat] Deleted embeddedservice_bootstrap.');
      }

      if (verbose) console.log('[removeChat] END');
    }

    /*****************************************
     * 3) reopenChat() - 新たに script をロードし initChat() → openChat()
     *****************************************/
    function reopenChat() {
      console.log('[reopenChat] START');
      removeChat(false);

      setTimeout(() => {
        console.log('[reopenChat] Adding new script tag...');
        const scriptEl = document.createElement('script');
        scriptEl.type = 'text/javascript';
        // 実際の bootstrap.min.js のURLを指定
        scriptEl.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';
        scriptEl.onload = () => {
          console.log('[reopenChat] Script loaded. Now calling initChat()...');
          if (window.embeddedservice_bootstrap) {
            initChat();
          } else {
            console.warn('[reopenChat] embeddedservice_bootstrap not defined after script load.');
          }
        };
        document.body.appendChild(scriptEl);
      }, 300);

      console.log('[reopenChat] END');
    }

    /*****************************************
     * 4) ページ読み込み時にチャットを初期化
     *****************************************/
    function onScriptLoaded() {
      initChat();
      // ※ MutationObserver は削除しているので何も呼ばない
    }
  </script>

  <!-- 最初のスクリプト読み込みで initChat() 呼び出し -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="onScriptLoaded()"
  ></script>
</body>
</html>
