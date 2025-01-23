<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Auto Close Chat on "会話を終了した時刻"</title>
</head>
<body>
  <h1>Auto Close Chat on "会話を終了した時刻"</h1>
  <p>
    このサンプルでは、**DOM 監視 (MutationObserver)** を使い、
    <strong>「会話を終了した時刻」</strong> という文字列がページに出現した瞬間、
    <code>removeChat()</code> を呼び出してチャットを自動で閉じる例を示します。<br>
    ※もし Shadow DOM 内に文字列が表示される場合は検知できません。
  </p>

  <!-- チャット削除/再描画用のボタン (任意) -->
  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="reopenChat()">Reopen Chat</button>

  <hr/>

  <script>
    /*****************************************
     * A) initChat() - チャット初期化関数
     *****************************************/
    function initChat() {
      console.log('[initChat] START');
      try {
        embeddedservice_bootstrap.settings.language = 'ja';

        // 組織ID / デプロイID / Embedded Service URL を書き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',   // Org ID
          'MIAW4',            // Deployment ID
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // ES URL
          { scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com' }
        );
        console.log('[initChat] SUCCESS: Chat initialized.');
      } catch (err) {
        console.error('[initChat] ERROR:', err);
      }
      console.log('[initChat] END');
    }

    /*****************************************
     * B) removeChat() - チャット(iframe/script等)を削除
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

      // scriptタグ削除
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
      const allIframes = document.querySelectorAll(iframeSelectors);
      allIframes.forEach(ifr => {
        if (verbose) console.log('[removeChat] Deleting iframe:', ifr.outerHTML);
        ifr.remove();
      });

      // localStorage
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (err) {
        console.warn('[removeChat] localStorage remove error:', err);
      }

      // embeddedservice_bootstrap オブジェクト
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[removeChat] Deleted embeddedservice_bootstrap.');
      }

      if (verbose) console.log('[removeChat] END');
    }

    /*****************************************
     * C) reopenChat() - 再度チャットを表示
     *****************************************/
    function reopenChat() {
      console.log('[reopenChat] START');
      removeChat(false);

      setTimeout(() => {
        console.log('[reopenChat] Adding new script tag...');
        const scriptEl = document.createElement('script');
        scriptEl.type = 'text/javascript';
        scriptEl.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';
        scriptEl.onload = () => {
          console.log('[reopenChat] Script loaded. Calling initChat()...');
          if (window.embeddedservice_bootstrap) {
            initChat();
          } else {
            console.warn('[reopenChat] embeddedservice_bootstrap not defined');
          }
        };
        document.body.appendChild(scriptEl);
      }, 300);

      console.log('[reopenChat] END');
    }

    /*****************************************
     * D) 自動検知: MutationObserver
     *    - DOM変化を監視し、新たに追加された要素に
     *      「会話を終了した時刻」 が含まれていれば removeChat()
     *****************************************/
    function autoCloseOnEndTime() {
      console.log('[autoCloseOnEndTime] START MutationObserver...');
      const observer = new MutationObserver((mutationsList) => {
        for (const mutation of mutationsList) {
          if (mutation.addedNodes) {
            mutation.addedNodes.forEach((node) => {
              // nodeType = 1 (ELEMENT_NODE) の場合
              if (node.nodeType === Node.ELEMENT_NODE) {
                // innerTextに "会話を終了した時刻" が含まれるか？
                if (node.innerText && node.innerText.includes('会話を終了した時刻')) {
                  console.warn('[autoCloseOnEndTime] Detected end time text. Auto removing chat...');
                  removeChat();
                  // 必要なら observer.disconnect() する
                  observer.disconnect();
                }
              }
            });
          }
        }
      });

      // body以下のDOM変化を監視
      observer.observe(document.body, { childList: true, subtree: true });
    }

    /*****************************************
     * E) 初回ロード時の Script: initChat + autoCloseOnEndTime
     *****************************************/
    function onScriptLoaded() {
      initChat();
      autoCloseOnEndTime();
    }
  </script>

  <!-- 初回ロード時に onScriptLoaded() を呼ぶ -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="onScriptLoaded()"
  ></script>
</body>
</html>
