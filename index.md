<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Test End Time Text via querySelector</title>
</head>
<body>
  <h1>Salesforce Embedded Messaging - Test "会話を終了した時刻" Retrieval</h1>
  <p>
    このサンプルでは、以下のボタンを用意しています:<br>
    - <strong>Remove Chat</strong>: Embedded Messaging (iframe, script など) を削除<br>
    - <strong>Reopen Chat</strong>: 再度 <code>bootstrap.min.js</code> を読み込み、チャットを初期化 (iframe再描画)<br>
    - <strong>Check End Time</strong>: ページ上の要素を走査し、「会話を終了した時刻」テキストを含む要素があるかを検索
  </p>

  <!-- 操作ボタン -->
  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="reopenChat()">Reopen Chat</button>
  <button onclick="testEndTime()">Check End Time</button>

  <hr>

  <script>
    /********************************************************************
     * 1) initChat()
     *    - Embedded Messaging を初期化 (iframeを生成) する関数
     ********************************************************************/
    function initChat() {
      console.log('[initChat] START');
      try {
        // 言語設定 (例: 'ja')
        embeddedservice_bootstrap.settings.language = 'ja';

        // 実際の OrgID / DeploymentID / EmbeddedServiceURL に書き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',  // Org ID
          'MIAW4',            // Deployment ID
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // ES URL
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
     *    - すでに表示中のチャットを削除 (iframe, script, localStorage, など)
     ********************************************************************/
    function removeChat(verbose = true) {
      if (verbose) console.log('[removeChat] START');

      // removeIframe() があれば呼ぶ
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        try {
          if (verbose) console.log('[removeChat] Calling removeIframe()...');
          window.embeddedservice_bootstrap.core.removeIframe();
        } catch (err) {
          console.warn('[removeChat] removeIframe error:', err);
        }
      }

      // scriptタグ削除
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        if (verbose) console.log('[removeChat] Removing script tag:', script.outerHTML);
        script.remove();
      }

      // iframe削除
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

      // localStorage 削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (e) {
        console.warn('[removeChat] Error clearing localStorage:', e);
      }

      // window.embeddedservice_bootstrap 削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[removeChat] Deleted embeddedservice_bootstrap.');
      }

      // ログ iframe残数
      const remaining = document.querySelectorAll('iframe');
      if (verbose) {
        console.log('[removeChat] Iframes left:', remaining.length);
        remaining.forEach((frm, idx) => {
          console.log(`- Iframe #${idx}:`, frm.outerHTML);
        });
      }

      if (verbose) console.log('[removeChat] END');
    }

    /********************************************************************
     * 3) reopenChat():
     *    - scriptタグを再挿入 → onload で initChat() → iframe再描画
     ********************************************************************/
    function reopenChat() {
      console.log('[reopenChat] START');

      // 念のため既存チャット削除
      removeChat(false);

      // 新しく script を挿入し、onload で initChat() を呼ぶ
      setTimeout(() => {
        console.log('[reopenChat] Adding new script tag for bootstrap.min.js...');
        const scriptEl = document.createElement('script');
        scriptEl.type = 'text/javascript';
        scriptEl.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';
        scriptEl.onload = () => {
          console.log('[reopenChat] Script loaded. Calling initChat()...');
          if (window.embeddedservice_bootstrap) {
            initChat();
          } else {
            console.warn('[reopenChat] embeddedservice_bootstrap not defined after load.');
          }
        };
        scriptEl.onerror = (err) => {
          console.error('[reopenChat] script load error:', err);
        };
        document.body.appendChild(scriptEl);
      }, 300);

      console.log('[reopenChat] END');
    }

    /********************************************************************
     * 4) testEndTime():
     *    - ページ中のすべての要素を走査し、
     *      innerText に「会話を終了した時刻」という文字列が含まれるか調査
     ********************************************************************/
    function testEndTime() {
      console.log('[testEndTime] START');
      // 1) 全要素を取得
      const allEls = document.querySelectorAll('*');

      let found = false;
      allEls.forEach((el) => {
        if (el.innerText && el.innerText.includes('会話を終了した時刻')) {
          found = true;
          console.warn('[testEndTime] Found element with end time text:', el, 'innerText=', el.innerText);
        }
      });

      if (!found) {
        console.log('[testEndTime] No element containing "会話を終了した時刻" found in the normal DOM');
      }
      console.log('[testEndTime] END');
    }
  </script>

  <!-- 
       初回ロード用スクリプト 
       onload="initChat()" でチャット初期化
  -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initChat()"
  ></script>
</body>
</html>
