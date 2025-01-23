<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Salesforce Embedded Messaging - Remove & Reopen Example</title>
</head>
<body>
  <h1>Salesforce Embedded Messaging - Remove &amp; Reopen Example</h1>
  <p>
    このサンプルでは、以下の2つのボタンで Embedded Messaging を制御します。<br>
    1. <strong>Remove Chat</strong>: チャット (iframe, script, localStorageなど) を削除<br>
    2. <strong>Reopen Chat</strong>: 再度 script を挿入し、初期化してチャットウィンドウを再描画
  </p>

  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="reopenChat()">Reopen Chat</button>

  <hr>

  <script>
    /********************************************************************
     * 1) initChat()
     *    - チャットを初期化 (iframeを描画) する関数
     *    - Salesforceのbootstrap.min.js がロードされたら呼ぶ想定
     ********************************************************************/
    function initChat() {
      console.log('[initChat] START');
      try {
        // 言語設定
        embeddedservice_bootstrap.settings.language = 'ja';

        // 実際の組織ID / デプロイID / Embedded Service URLに書き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',  // Org ID
          'MIAW4',            // Deployment ID
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // Embedded Service URL
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );
        console.log('[initChat] SUCCESS: Chat initialized.');
      } catch (err) {
        console.error('[initChat] ERROR:', err);
      }
      console.log('[initChat] END');
    }

    /********************************************************************
     * 2) removeChat():  
     *    - チャット(iframe, script, localStorage, window object) を削除
     ********************************************************************/
    function removeChat(verbose = true) {
      if (verbose) console.log('[removeChat] START');

      // 2-1) removeIframe() があれば呼ぶ
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

      // 2-2) scriptタグ (bootstrap.min.js) を削除
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        if (verbose) console.log('[removeChat] Removing script tag:', script.outerHTML);
        script.remove();
      } else {
        if (verbose) console.warn('[removeChat] No script tag found for bootstrap.min.js');
      }

      // 2-3) チャット用 iframe を削除
      const iframeSelectors = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');
      const chatIframes = document.querySelectorAll(iframeSelectors);
      chatIframes.forEach((iframe) => {
        if (verbose) console.log('[removeChat] Deleting iframe:', iframe.outerHTML);
        iframe.remove();
      });

      // 2-4) localStorage 削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (e) {
        console.warn('[removeChat] Error removing localStorage keys:', e);
      }

      // 2-5) window.embeddedservice_bootstrap 削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[removeChat] Deleted embeddedservice_bootstrap from window.');
      }

      // 2-6) ログに残っているiframe数を表示
      const remaining = document.querySelectorAll('iframe');
      if (verbose) {
        console.log('[removeChat] Iframes left in DOM:', remaining.length);
        remaining.forEach((frm, idx) => {
          console.log(`- Iframe #${idx}:`, frm.outerHTML);
        });
      }

      if (verbose) console.log('[removeChat] END');
    }

    /********************************************************************
     * 3) reopenChat():  
     *    - scriptタグを再挿入 → onload で initChat() を呼び出し → iframe再描画
     ********************************************************************/
    function reopenChat() {
      console.log('[reopenChat] START');

      // まず removeChat() で前のチャットを破棄 (念のため)
      removeChat(false);

      // script タグを再挿入
      setTimeout(() => {
        console.log('[reopenChat] Adding bootstrap.min.js script tag again...');
        const scriptEl = document.createElement('script');
        scriptEl.type = 'text/javascript';
        // 実際の bootstrap.min.js のURLを設定
        scriptEl.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';
        // ロード後に initChat() を呼ぶ
        scriptEl.onload = () => {
          console.log('[reopenChat] Script loaded. Calling initChat()...');
          if (window.embeddedservice_bootstrap) {
            initChat();
          } else {
            console.warn('[reopenChat] embeddedservice_bootstrap not defined after script load.');
          }
        };
        scriptEl.onerror = (err) => {
          console.error('[reopenChat] Failed to load script:', err);
        };
        document.body.appendChild(scriptEl);
      }, 300);

      console.log('[reopenChat] END');
    }
  </script>

  <!-- 
       4) 初回ロード時: script を読み込み、onload で initChat() を呼ぶ
       src= のURLは実際の Salesfore Embedded Service のbootstrap.min.js に書き換える
  -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initChat()"
  ></script>
</body>
</html>
