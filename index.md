<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Salesforce Embedded Messaging - Simple Manual End Chat</title>
</head>
<body>
  <h1>Salesforce Embedded Messaging - Manual End Chat Example</h1>
  <p>
    こちらのページでは、<strong>独自の「End Chat」ボタン</strong> を用意し、  
    ボタンを押すと <code>destroyEmbeddedMessaging()</code> を呼び出して、  
    すべてのチャット関連要素 (iframe, script, localStorage など) を削除します。
  </p>

  <!-- 1) 手動終了ボタン -->
  <button onclick="endChat()">End Chat</button>

  <hr/>

  <script>
    /********************************************************************
     * (A) Embedded Messaging 初期化関数
     *     - Salesforce が提供する bootstrap.min.js の onload で呼ばれる
     ********************************************************************/
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');
      try {
        // 言語設定
        embeddedservice_bootstrap.settings.language = 'ja';

        // 組織ID / デプロイID / Embedded Service URL を書き換えてください
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


    /********************************************************************
     * (B) 独自の「End Chat」ボタンを押したときの処理
     *     - destroyEmbeddedMessaging() を呼ぶだけ
     ********************************************************************/
    function endChat() {
      console.log('[endChat] User triggered custom End Chat button. Destroying chat...');
      destroyEmbeddedMessaging();
    }


    /********************************************************************
     * (C) destroyEmbeddedMessaging(): 
     *     チャット用の iframe, script, localStorage, embeddedservice_bootstrap を削除
     ********************************************************************/
    function destroyEmbeddedMessaging(verbose = true) {
      if (verbose) console.log('[destroyEmbeddedMessaging] START');

      // removeIframe() が使えるなら先に呼ぶ
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        if (verbose) console.log('[destroyEmbeddedMessaging] Calling removeIframe()...');
        try {
          window.embeddedservice_bootstrap.core.removeIframe();
        } catch (err) {
          console.warn('[destroyEmbeddedMessaging] removeIframe error:', err);
        }
      }

      // script タグ削除 (bootstrap.min.js)
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        if (verbose) console.log('[destroyEmbeddedMessaging] Removing script tag:', script.outerHTML);
        script.remove();
      }

      // すべての Embedded Messaging iframe を削除
      const iframeSelector = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');

      const chatIframes = document.querySelectorAll(iframeSelector);
      chatIframes.forEach((iframe) => {
        if (verbose) console.log('[destroyEmbeddedMessaging] Deleting iframe:', iframe.outerHTML);
        iframe.remove();
      });

      // localStorage のセッション情報を削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (e) {
        console.warn('[destroyEmbeddedMessaging] Error clearing localStorage:', e);
      }

      // embeddedservice_bootstrap オブジェクトを削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
      }

      if (verbose) {
        const remainingIframes = document.querySelectorAll('iframe');
        console.log(`[destroyEmbeddedMessaging] Iframes left in DOM: ${remainingIframes.length}`);
        remainingIframes.forEach((frame, idx) => {
          console.log(`- Iframe #${idx}:`, frame.outerHTML);
        });
      }

      if (verbose) console.log('[destroyEmbeddedMessaging] END');
    }
  </script>

  <!-- 
       (D) 初回ロード時に initEmbeddedMessaging() を呼ぶスクリプト 
       src= のURLは各自の組織に合わせて書き換えてください
  -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>
</body>
</html>
