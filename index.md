<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Force Chat with Custom OnClick</title>
</head>
<body>
  <h1>Force Chat with Custom OnClick</h1>
  <p>
    ランチャーアイコンを強制表示したあと、<br>
    <strong>クリック時に <code>embeddedservice_bootstrap.openChat()</code> を呼ぶ</strong> ようにして、  
    内部状態が「終了」扱いでもウィンドウを無理やり開きます。
  </p>

  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="reAddChat()">Re-Add Chat</button>

  <hr/>

  <script type="text/javascript">
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');
      try {
        embeddedservice_bootstrap.settings.language = 'ja';

        // 1) onEmbeddedMessagingReady: UI 準備完了時にランチャーボタンを強制表示 & onclick 設定
        window.addEventListener('onEmbeddedMessagingReady', () => {
          console.log('[onEmbeddedMessagingReady] triggered. Forcing launcher display & custom onclick...');

          // ランチャーボタンを探す
          const launcherBtn = document.getElementById('embeddedMessagingConversationButton');
          if (launcherBtn) {
            console.log('[onEmbeddedMessagingReady] Found button. Overriding display:none, setting onclick...');
            launcherBtn.style.display = 'block';            // 表示を強制
            launcherBtn.removeAttribute('disabled');        // 念のため有効化
            launcherBtn.removeAttribute('tabindex');        // -1 なら削除
            launcherBtn.style.pointerEvents = 'auto';       // クリック可能に

            // ボタンをクリックしたら openChat() を呼ぶ
            launcherBtn.onclick = () => {
              console.log('[launcherBtn.onclick] Forced openChat()');
              if (embeddedservice_bootstrap && typeof embeddedservice_bootstrap.openChat === 'function') {
                embeddedservice_bootstrap.openChat();
              } else {
                console.warn('[launcherBtn.onclick] openChat() not available');
              }
            };
          } else {
            console.warn('[onEmbeddedMessagingReady] No #embeddedMessagingConversationButton found');
          }
        });

        // 2) init
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

    // 既存チャット削除
    function removeChat() {
      console.log('[removeChat] START');
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        console.log('[removeChat] removeIframe()...');
        try {
          window.embeddedservice_bootstrap.core.removeIframe();
        } catch (e) {
          console.warn('[removeChat] removeIframe error:', e);
        }
      }
      const scriptTag = document.querySelector("script[src*='bootstrap.min.js']");
      if (scriptTag) {
        console.log('[removeChat] Removing script:', scriptTag.outerHTML);
        scriptTag.remove();
      }
      const iframeSel = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');
      document.querySelectorAll(iframeSel).forEach(ifr => {
        console.log('[removeChat] Removing iframe:', ifr.outerHTML);
        ifr.remove();
      });
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch(e) {
        console.warn('[removeChat] localStorage remove error:', e);
      }
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        console.log('[removeChat] Deleted window.embeddedservice_bootstrap');
      }
      console.log('[removeChat] END');
    }

    // 新たにチャットを再初期化
    function reAddChat() {
      console.log('[reAddChat] START');
      removeChat();
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

  <!-- 初回ロード時にinit -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>

</body>
</html>
