<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Salesforce Embedded Messaging - Auto Reinit on Chat End</title>
</head>
<body>
  <h1>Salesforce Embedded Messaging - Auto Reinit on Chat End</h1>
  <p>
    こちらは、<strong>チャット終了をフックして自動再起動</strong> するサンプルです。<br>
    - <em>onEvent コールバック</em> と <em>DOM 監視</em> を併用して会話終了を検知し、<br>
    - <code>destroyEmbeddedMessaging()</code> → <code>reinitEmbeddedMessaging()</code> を自動的に呼び出します。
  </p>

  <!-- 手動ボタン (デバッグ用) -->
  <button onclick="destroyEmbeddedMessaging()">Force Close (手動)</button>
  <button onclick="reinitEmbeddedMessaging()">Reinit (手動)</button>

  <hr/>

  <script>
    /********************************************************************
     * A. 初期化関数: Salesforce の Embedded Messaging を初期化
     *    - onEvent(ConversationEnded) コールバック設定
     *    - DOM 監視 (フォールバック) 開始
     ********************************************************************/
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');
      try {
        // 1) onEvent コールバック設定
        if (window.embeddedservice_bootstrap && embeddedservice_bootstrap.settings) {
          const originalOnEvent = embeddedservice_bootstrap.settings.onEvent;
          embeddedservice_bootstrap.settings.onEvent = function(eventData) {
            if (originalOnEvent) {
              originalOnEvent(eventData);
            }
            console.log('[initEmbeddedMessaging] onEvent:', eventData);

            // もし "ConversationEnded" イベント名で終了を通知してくれるなら、ここで検知
            if (eventData && eventData.type === 'ConversationEnded') {
              console.warn('[initEmbeddedMessaging] Detected ConversationEnded event. Auto reinit...');
              autoReinitAfterChatEnd();
            }
          };
        }

        // 2) 言語設定 (例: 'ja')
        embeddedservice_bootstrap.settings.language = 'ja';

        // 3) init (Org ID / Deploy ID / Embedded Service URL を書き換えてください)
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn', // Org ID (例)
          'MIAW4',           // Deployment ID (例)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136',
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initEmbeddedMessaging] SUCCESS: Chat initialized.');
      } catch (err) {
        console.error('[initEmbeddedMessaging] ERROR:', err);
      }

      console.log('[initEmbeddedMessaging] END');

      // 4) フォールバック用の DOM 監視開始 (onEvent が働かない場合用)
      startDomObserverForChatEnd();
    }

    /********************************************************************
     * B. フォールバック: DOM 監視
     *    - 「会話が終了しました」などのテキストやUIが出現したら自動再起動
     *    - 実際の文言やクラス名は環境によって異なるので調整してください
     ********************************************************************/
    function startDomObserverForChatEnd() {
      console.log('[startDomObserverForChatEnd] START');
      const targetNode = document.body;

      const observer = new MutationObserver((mutationsList) => {
        for (const mutation of mutationsList) {
          if (mutation.addedNodes) {
            mutation.addedNodes.forEach((node) => {
              if (node.nodeType === Node.ELEMENT_NODE) {
                // 例: innerText に「会話が終了しました」が含まれていれば終了と判断
                if (node.innerText && node.innerText.includes('会話が終了しました')) {
                  console.warn('[startDomObserverForChatEnd] Detected "会話が終了しました" text. Auto reinit...');
                  autoReinitAfterChatEnd();
                }
              }
            });
          }
        }
      });

      observer.observe(targetNode, { childList: true, subtree: true });
    }

    /********************************************************************
     * C. 会話終了を検知した際の処理 (destroy → reinit)
     *    - 連続で呼ばれないようフラグ制御 (一度実行したら再度実行しない)
     ********************************************************************/
    let isAlreadyReinitializing = false;

    function autoReinitAfterChatEnd() {
      if (isAlreadyReinitializing) {
        console.log('[autoReinitAfterChatEnd] Already in progress. Skip.');
        return;
      }
      isAlreadyReinitializing = true;

      console.log('[autoReinitAfterChatEnd] Attempting to auto reinit...');
      destroyEmbeddedMessaging(); // まず強制クローズ

      // 1秒後に再初期化
      setTimeout(() => {
        reinitEmbeddedMessaging();
      }, 1000);
    }

    /********************************************************************
     * D. destroy: 強制クローズ (iframe, script, localStorage など削除)
     ********************************************************************/
    function destroyEmbeddedMessaging(verbose = true) {
      if (verbose) console.log('[destroyEmbeddedMessaging] START');

      // removeIframe() があれば使う
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

      // script タグ削除
      const script = document.querySelector("script[src*='bootstrap.min.js']");
      if (script) {
        if (verbose) console.log('[destroyEmbeddedMessaging] Removing script tag:', script.outerHTML);
        script.remove();
      }

      // 複数 iframe 削除 (siteContextFrame, embeddedMessagingFrame, filePreviewFrame 等)
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

      // localStorage 削除
      try {
        localStorage.removeItem('embeddedMessaging:conversationData');
        localStorage.removeItem('embeddedMessaging:isLoggedIn');
        localStorage.removeItem('embeddedMessaging:settings');
      } catch (e) {
        console.warn('[destroyEmbeddedMessaging] Error clearing localStorage:', e);
      }

      // window.embeddedservice_bootstrap 削除
      if (window.embeddedservice_bootstrap) {
        delete window.embeddedservice_bootstrap;
      }

      // iframe残数をログ表示
      if (verbose) {
        const remainingIframes = document.querySelectorAll('iframe');
        console.log(`[destroyEmbeddedMessaging] Iframes left in DOM: ${remainingIframes.length}`);
        remainingIframes.forEach((frame, idx) => {
          console.log(`- Iframe #${idx}:`, frame, 'outerHTML=', frame.outerHTML);
        });
      }

      if (verbose) console.log('[destroyEmbeddedMessaging] END');
    }

    /********************************************************************
     * E. Reinit: スクリプトを再読み込み → initEmbeddedMessaging()
     ********************************************************************/
    function reinitEmbeddedMessaging() {
      console.log('[reinitEmbeddedMessaging] START');
      // script タグ再挿入
      setTimeout(() => {
        console.log('[reinitEmbeddedMessaging] Adding new script tag...');
        const scriptTag = document.createElement('script');
        scriptTag.type = 'text/javascript';
        // ここは各自の URL に書き換えてください
        scriptTag.src = 'https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js';
        scriptTag.onload = function() {
          console.log('[reinitEmbeddedMessaging] Script loaded. Calling initEmbeddedMessaging()...');
          if (window.embeddedservice_bootstrap) {
            initEmbeddedMessaging();
          } else {
            console.warn('[reinitEmbeddedMessaging] embeddedservice_bootstrap not defined after script load.');
          }
        };
        document.body.appendChild(scriptTag);
      }, 500);

      console.log('[reinitEmbeddedMessaging] END (will load script in 500ms)');
    }
  </script>

  <!-- 初回ロード時に initEmbeddedMessaging() を呼ぶスクリプト -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>
</body>
</html>
