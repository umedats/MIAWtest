<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Restart Chat Session after End2</title>
</head>
<body>
  <h1>Restart Chat Session after End</h1>
  <p>
    このサンプルは、終了したセッションを再利用するのではなく、<br>
    <strong>まったく新しいセッションとしてチャットを開始</strong> する流れを示します。<br>
    事前チャットフォームの設定で「すべてのセッション」または「すべての会話」を選択しておけば、<br>
    新しいセッション開始時に事前チャットフォームが表示されます。
  </p>

  <!-- 会話終了時に呼ばれる想定のボタン (デモ用) -->
  <button onclick="endChatAndRestart()">End Chat & Restart</button>

  <hr />

  <script>
    /************************************************************
     * 1) initChat()
     *    - Embedded Messaging の初期化 (＝新規セッション開始)
     ************************************************************/
    function initChat() {
      console.log('[initChat] START');
      try {
        // 言語設定 (例: 'ja')
        embeddedservice_bootstrap.settings.language = 'ja';

        // 組織ID / デプロイID / ES URL は環境に合わせて書き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',  // Org ID 例
          'MIAW4',            // デプロイID 例
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // ES URL 例
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initChat] SUCCESS: Chat initialized.');

      } catch (err) {
        console.error('[initChat] ERROR:', err);
      }
      console.log('[initChat] END');

      // ★ もし自動でウィンドウを開きたい場合は以下を活用
      setTimeout(() => {
        if (window.embeddedservice_bootstrap && typeof embeddedservice_bootstrap.openChat === 'function') {
          console.log('[initChat] Calling openChat() to display chat window...');
          embeddedservice_bootstrap.openChat();
        } else {
          console.warn('[initChat] openChat() not found. The user may need to click the chat icon, if it exists.');
        }
      }, 500);
    }

    /************************************************************
     * 2) removeChat()
     *    - 古いチャット要素(iframe, script, localStorage, JSオブジェクト)を削除
     ************************************************************/
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

      // script タグ (bootstrap.min.js) を削除
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

      const chatIframes = document.querySelectorAll(iframeSelectors);
      chatIframes.forEach((ifr) => {
        if (verbose) console.log('[removeChat] Deleting iframe:', ifr.outerHTML);
        ifr.remove();
      });

      // localStorage 上のチャット情報を削除
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
        if (verbose) console.log('[removeChat] Deleted embeddedservice_bootstrap.');
      }

      if (verbose) console.log('[removeChat] END');
    }

    /************************************************************
     * 3) loadChatScriptAndInit()
     *    - 新しい <script> を挿入 → onload で initChat() を実行
     ************************************************************/
    function loadChatScriptAndInit() {
      console.log('[loadChatScriptAndInit] START');
      // 新しい script タグを作成
      const scriptEl = document.createElement('script');
      scriptEl.type = 'text/javascript';

      // 自組織の bootstrap.min.js のURLに書き換えてください
      scriptEl.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';

      // scriptロード完了後に initChat() を呼ぶ
      scriptEl.onload = () => {
        console.log('[loadChatScriptAndInit] Script loaded. Now calling initChat()...');
        if (window.embeddedservice_bootstrap) {
          initChat();
        } else {
          console.warn('[loadChatScriptAndInit] embeddedservice_bootstrap not defined after script load.');
        }
      };

      // script タグを追加
      document.body.appendChild(scriptEl);
      console.log('[loadChatScriptAndInit] END');
    }

    /************************************************************
     * 4) endChatAndRestart()
     *    - (1) removeChat() で既存セッションを破棄
     *    - (2) 新しい script を挿入 → initChat() を呼び出し
     *    - ＝ 新しいセッションとしてチャットを開始
     ************************************************************/
    function endChatAndRestart() {
      console.log('[endChatAndRestart] START');
      removeChat(); // まずチャット要素を完全削除

      // 少し待ってから新たな script を挿入して init
      setTimeout(() => {
        loadChatScriptAndInit();
      }, 500);
      console.log('[endChatAndRestart] END');
    }

    /************************************************************
     * 5) ページ初回ロード時
     *    - すでに会話を開始したいなら、下記で script を読み込み & initChat()
     ************************************************************/
    function onScriptLoad() {
      console.log('[onScriptLoad] -> loadChatScriptAndInit');
      loadChatScriptAndInit();
    }
  </script>

  <!-- ページロード時に onScriptLoad() を呼ぶ -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="onScriptLoad()"
  ></script>
</body>
</html>
