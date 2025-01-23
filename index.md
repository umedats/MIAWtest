<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Embedded Service Chat Test</title>
</head>
<body>
  <h1>Salesforce Embedded Service Chat Test</h1>
  <p>
    こちらは、Salesforce の Embedded Service デプロイメントから
    取得したコードスニペットを埋め込んだテスト用のページです。
  </p>

  <!-- ボタン群: Chat を削除/再表示 -->
  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="reAddChat()">Re-Init Chat</button>

  <hr />

  <script type="text/javascript">
    /**
     * initEmbeddedMessaging()
     * - チャットを初期化するメソッド
     */
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');
      try {
        // 言語設定を日本語に
        embeddedservice_bootstrap.settings.language = 'ja';

        // 実際の組織 ID / デプロイ ID / URL に書き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',                    // Org ID (例)
          'MIAW4',                              // Deployment ID (例)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // ES URL
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

    /**
     * removeChat()
     * - 既存チャット要素 (iframe, script, localStorage, JSオブジェクト) を削除
     */
    function removeChat() {
      console.log('[removeChat] START');

      // removeIframe() があれば呼ぶ (既にロード済みの場合)
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        console.log('[removeChat] Calling removeIframe() to remove chat iframe...');
        try {
          window.embeddedservice_bootstrap.core.removeIframe();
        } catch (e) {
          console.warn('[removeChat] removeIframe() threw error:', e);
        }
      }

      // scriptタグ (bootstrap.min.js) を削除
      const scriptTag = document.querySelector("script[src*='bootstrap.min.js']");
      if (scriptTag) {
        console.log('[removeChat] Removing script tag:', scriptTag.outerHTML);
        scriptTag.remove();
      } else {
        console.warn('[removeChat] No existing Chat script tag found');
      }

      // もしまだ iframe が残っていれば削除 (念のため)
      const iframeSelectors = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');
      const chatIframes = document.querySelectorAll(iframeSelectors);
      chatIframes.forEach(ifr => {
        console.log('[removeChat] Removing iframe:', ifr.outerHTML);
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
        console.log('[removeChat] Deleted window.embeddedservice_bootstrap.');
      }

      console.log('[removeChat] END');
    }

    /**
     * reAddChat()
     * - 新しく <script> を挿入して再度 initEmbeddedMessaging() を呼ぶ
     */
    function reAddChat() {
      console.log('[reAddChat] START');
      // 念のため既存チャットを削除
      removeChat();

      // 新しい <script> タグを作成
      const scriptEl = document.createElement('script');
      scriptEl.type = 'text/javascript';

      // 実際の bootstrap.min.js のURLに書き換えてください
      scriptEl.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';

      // ロード完了後に initEmbeddedMessaging() を呼び出す
      scriptEl.onload = function() {
        console.log('[reAddChat] Script loaded, now calling initEmbeddedMessaging()...');
        if (window.embeddedservice_bootstrap) {
          initEmbeddedMessaging();
        } else {
          console.warn('[reAddChat] embeddedservice_bootstrap not defined after script load');
        }
      };

      // Scriptタグを body の末尾に追加
      document.body.appendChild(scriptEl);

      console.log('[reAddChat] END (script insertion done)');
    }
  </script>

  <!-- 初回ロードでチャットを表示 (下記スクリプト) -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>

</body>
</html>
