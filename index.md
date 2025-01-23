<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Debugging DOM Elements for Chat</title>
</head>
<body>
  <h1>Debugging DOM Elements for Chat</h1>
  <p>
    「ランチャーアイコン」や「チャット iframe」が正しく DOM に生成されているかをチェックするための
    <strong>ログ出力用関数</strong>を用意しています。
  </p>

  <!-- 操作用ボタン (任意) -->
  <button onclick="removeChat()">Remove Chat</button>
  <button onclick="reopenChat()">Reopen Chat</button>
  <button onclick="logAllChatElements()">Log Chat Elements</button>

  <hr/>

  <script>
    /********************************************************************
     * 1) initChat()
     *    - ここでは例としてチャットを初期化 → ランチャーアイコンを表示する想定
     ********************************************************************/
    function initChat() {
      console.log('[initChat] START');
      try {
        embeddedservice_bootstrap.settings.language = 'ja';
        
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',     // Org ID (例)
          'MIAW4',              // Deployment ID (例)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // ES URL (例)
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
     * 2) removeChat()
     *    - チャット要素 (iframe, script, localStorage, JSオブジェクト) を削除
     ********************************************************************/
    function removeChat(verbose = true) {
      if (verbose) console.log('[removeChat] START');

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

      const scriptTag = document.querySelector("script[src*='bootstrap.min.js']");
      if (scriptTag) {
        if (verbose) console.log('[removeChat] Removing script tag:', scriptTag.outerHTML);
        scriptTag.remove();
      }

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

      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (e) {
        console.warn('[removeChat] localStorage remove error:', e);
      }

      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
        if (verbose) console.log('[removeChat] Deleted embeddedservice_bootstrap.');
      }

      if (verbose) console.log('[removeChat] END');
    }

    /********************************************************************
     * 3) reopenChat()
     *    - removeChat() → 新しい script を挿入 → initChat()
     ********************************************************************/
    function reopenChat() {
      console.log('[reopenChat] START');
      removeChat(false);

      setTimeout(() => {
        console.log('[reopenChat] Adding new script tag...');
        const scriptEl = document.createElement('script');
        scriptEl.type = 'text/javascript';
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

    /********************************************************************
     * 4) logAllChatElements()
     *    - チャット関連と思われる DOM 要素をリストアップ & 状態をログ出力
     *    - ランチャーアイコン, iframe, ボタン などを探索し outerHTML や style を表示
     ********************************************************************/
    function logAllChatElements() {
      console.log('[logAllChatElements] START');

      // 4-1) iframe
      const iframeSelectors = [
        "iframe[data-embeddedmessaging]",
        "iframe[id*='embeddedMessaging']",
        "iframe[class*='embeddedMessaging']"
      ].join(',');
      const chatIframes = document.querySelectorAll(iframeSelectors);
      console.log(`[logAllChatElements] Found ${chatIframes.length} chat iframes using selectors: ${iframeSelectors}`);
      chatIframes.forEach((ifr, idx) => {
        console.log(`- Iframe #${idx}:`, ifr, 'outerHTML:', ifr.outerHTML, 'computedStyle:', getComputedStyle(ifr));
      });

      // 4-2) ランチャーアイコン / ボタン
      //    Embedded Messaging のランチャーボタンには "embeddedMessagingConversationButton" というクラスが付くことが多い
      const launcherButtons = document.querySelectorAll("button.embeddedMessagingConversationButton");
      console.log(`[logAllChatElements] Found ${launcherButtons.length} launcher buttons using selector: button.embeddedMessagingConversationButton`);
      launcherButtons.forEach((btn, idx) => {
        console.log(`- LauncherBtn #${idx}:`, btn, 'outerHTML:', btn.outerHTML, 'computedStyle:', getComputedStyle(btn));
      });

      // 4-3) 全要素の中に "embeddedMessaging" や "embeddedservice" といった文字列がclass/idに含まれるものを検索
      const allEls = document.querySelectorAll('*');
      const suspectEls = [];
      allEls.forEach(el => {
        const idClass = (el.id + ' ' + el.className).toLowerCase();
        if (idClass.includes('embeddedmessaging') || idClass.includes('embeddedservice')) {
          suspectEls.push(el);
        }
      });
      console.log(`[logAllChatElements] Found ${suspectEls.length} suspect elements (id/class containing "embeddedMessaging" or "embeddedservice")`);
      suspectEls.forEach((el, idx) => {
        console.log(`- SuspectEl #${idx}:`, el, 'outerHTML:', el.outerHTML, 'computedStyle:', getComputedStyle(el));
      });

      console.log('[logAllChatElements] END');
    }

    /********************************************************************
     * 5) onInitialLoad()
     *    - ページ読み込み時に最初のチャットを初期化したい場合はこちらから呼ぶ
     ********************************************************************/
    function onInitialLoad() {
      console.log('[onInitialLoad] Do nothing or init Chat here if needed.');
      // initChat();
      // もし最初からランチャーを表示したいなら上のコメントアウトを外す
    }
  </script>

  <!-- ページロード時 -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="onInitialLoad()"
  ></script>
</body>
</html>
