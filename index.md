<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Remove Chat & Recreate Launcher Example</title>
</head>
<body>
  <h1>Remove Chat & Recreate Launcher Example</h1>
  <p>
    以下の 2 つのボタンを用意しています。<br>
    1. <strong>Remove Chat</strong>: チャット (iframe, script, localStorage, など) を削除<br>
    2. <strong>Recreate Chat Launcher</strong>: 再度チャットのスクリプトを読み込み、<br>
       ランチャーアイコンを表示する初期化のみ行う (<code>openChat()</code> は呼ばない)
  </p>

  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="recreateChatLauncher()">Recreate Chat Launcher</button>

  <hr/>

  <script>
    /********************************************************************
     * 1) initChatForLauncherIcon()
     *    - Chatを初期化し、ランチャーアイコンだけ表示する。
     *    - ここでは openChat() を呼ばず、ユーザがアイコンを押して開く想定。
     ********************************************************************/
    function initChatForLauncherIcon() {
      console.log('[initChatForLauncherIcon] START');
      try {
        // 言語設定 (例: 'ja')
        embeddedservice_bootstrap.settings.language = 'ja';

        // 組織ID / デプロイID / Embedded Service URL を自環境に合わせて設定
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',  // Org ID (例)
          'MIAW4',            // Deployment ID (例)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // ES URL (例)
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initChatForLauncherIcon] SUCCESS: Chat initialized (launcher mode).');
      } catch (err) {
        console.error('[initChatForLauncherIcon] ERROR:', err);
      }
      console.log('[initChatForLauncherIcon] END');
    }

    /********************************************************************
     * 2) removeChat()
     *    - チャット要素 (iframe, script, localStorage, JS object) を削除
     ********************************************************************/
    function removeChat(verbose = true) {
      if (verbose) console.log('[removeChat] START');

      // removeIframe() があれば先に呼ぶ
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        if (verbose) console.log('[removeChat] Calling removeIframe()...');
        try {
          window.embeddedservice_bootstrap.core.removeIframe();
        } catch (err) {
          console.warn('[removeChat] removeIframe error:', err);
        }
      }

      // scriptタグ (bootstrap.min.js) を削除
      const scriptTag = document.querySelector("script[src*='bootstrap.min.js']");
      if (scriptTag) {
        if (verbose) console.log('[removeChat] Removing script tag:', scriptTag.outerHTML);
        scriptTag.remove();
      } else if (verbose) {
        console.warn('[removeChat] No existing script tag (bootstrap.min.js) found.');
      }

      // iframe削除
      const iframeSelectors = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');
      const chatIframes = document.querySelectorAll(iframeSelectors);
      chatIframes.forEach(ifr => {
        if (verbose) console.log('[removeChat] Deleting iframe:', ifr.outerHTML);
        ifr.remove();
      });

      // localStorage 上のチャット情報を削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (e) {
        console.warn('[removeChat] localStorage remove error:', e);
      }

      // window.embeddedservice_bootstrap を削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[removeChat] Deleted window.embeddedservice_bootstrap.');
      }

      if (verbose) console.log('[removeChat] END');
    }

    /********************************************************************
     * 3) recreateChatLauncher()
     *    - removeChat() → 新しい script を挿入 → onload で initChatForLauncherIcon()
     ********************************************************************/
    function recreateChatLauncher() {
      console.log('[recreateChatLauncher] START');
      removeChat(false);

      // 少し待ってから script を再挿入
      setTimeout(() => {
        console.log('[recreateChatLauncher] Adding new script tag for bootstrap.min.js...');
        const scriptEl = document.createElement('script');
        scriptEl.type = 'text/javascript';

        // 組織の bootstrap.min.js のURLに合わせる
        scriptEl.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';

        scriptEl.onload = () => {
          console.log('[recreateChatLauncher] Script loaded. Now calling initChatForLauncherIcon()...');
          if (window.embeddedservice_bootstrap) {
            initChatForLauncherIcon();
          } else {
            console.warn('[recreateChatLauncher] embeddedservice_bootstrap not defined after script load.');
          }
        };
        scriptEl.onerror = (err) => {
          console.error('[recreateChatLauncher] Script load error:', err);
        };

        document.body.appendChild(scriptEl);
      }, 500);

      console.log('[recreateChatLauncher] END');
    }

    /********************************************************************
     * 4) onInitialLoad()
     *    - ページ読み込み時に一度だけ実行する (必要なら)
     *    - ここでは最初にチャットを表示したくない場合は何も呼ばない or
     *      initChatForLauncherIcon() を呼んで最初からアイコンを出しても可
     ********************************************************************/
    function onInitialLoad() {
      console.log('[onInitialLoad] Do nothing or init Chat here if needed.');
      // 例: initChatForLauncherIcon(); // ←初回からランチャーを出したい場合
    }
  </script>

  <!-- 初期ロードで特にチャットを表示しない場合、何もせず -->
  <script
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="onInitialLoad()"
  ></script>
</body>
</html>
