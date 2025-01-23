<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Force Show Launcher after Reinit</title>
</head>
<body>
  <h1>Force Show Launcher after Reinit</h1>

  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="reAddChat()">Re-Init Chat</button>

  <hr />

  <script type="text/javascript">
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');
      try {
        embeddedservice_bootstrap.settings.language = 'ja';

        // ここで "onEmbeddedMessagingReady" のリスナーを仕込む
        window.addEventListener('onEmbeddedMessagingReady', () => {
          console.log('[onEmbeddedMessagingReady] triggered. Forcing launcher to display.');

          // ランチャーボタンを強制的に表示
          const launcherBtn = document.getElementById('embeddedMessagingConversationButton');
          if (launcherBtn) {
            console.log('[onEmbeddedMessagingReady] Found button. Overriding display:none...');
            launcherBtn.style.display = 'block'; // 強制的に表示
            launcherBtn.removeAttribute('tabindex'); // もし tabindex=-1 なら削除
          } else {
            console.warn('[onEmbeddedMessagingReady] No #embeddedMessagingConversationButton found');
          }
        });

        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',
          'MIAW4',
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136',
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

    function removeChat() {
      console.log('[removeChat] START');
      // removeIframe などで既存要素を削除
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        console.log('[removeChat] removeIframe()...');
        try {
          window.embeddedservice_bootstrap.core.removeIframe();
        } catch(e) { console.warn('[removeChat] removeIframe error:', e); }
      }
      const scriptTag = document.querySelector("script[src*='bootstrap.min.js']");
      if (scriptTag) {
        console.log('[removeChat] Removing script:', scriptTag.outerHTML);
        scriptTag.remove();
      }
      // iframe など削除
      const iframeSel = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');
      document.querySelectorAll(iframeSel).forEach(ifr => {
        console.log('[removeChat] Removing iframe:', ifr.outerHTML);
        ifr.remove();
      });
      // localStorage
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch(e) { console.warn('[removeChat] localStorage remove error:', e); }
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        console.log('[removeChat] Deleted window.embeddedservice_bootstrap');
      }
      console.log('[removeChat] END');
    }

    function reAddChat() {
      console.log('[reAddChat] START');
      removeChat();
      // 新たな script を挿入
      setTimeout(() => {
        console.log('[reAddChat] Inserting new script for Chat...');
        const scriptEl = document.createElement('script');
        scriptEl.type = 'text/javascript';
        scriptEl.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';
        scriptEl.onload = () => {
          console.log('[reAddChat] Script loaded, now calling initEmbeddedMessaging()...');
          if (window.embeddedservice_bootstrap) {
            initEmbeddedMessaging();
          } else {
            console.warn('[reAddChat] embeddedservice_bootstrap not defined after load.');
          }
        };
        document.body.appendChild(scriptEl);
      }, 300);
      console.log('[reAddChat] END');
    }
  </script>

  <!-- 初回ロード: 何もしない (or onload="initEmbeddedMessaging()") -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>
</body>
</html>
