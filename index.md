<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Salesforce Embedded Messaging Example</title>
</head>
<body>
  <h1>Salesforce Embedded Messaging Example</h1>
  <p>
    このページは、<strong>Salesforce Embedded Messaging</strong> を読み込み、  
    会話終了イベントをフックして「ページリロード」または「チャットの再初期化」を行うサンプルです。<br>
    <em>（※ カスタム実装のため公式サポート外です）</em>
  </p>

  <!-- 操作用のボタン -->
  <button onclick="endChatAndReload()">End Chat & Reload (Demo)</button>
  <button onclick="endChatAndReinit()">End Chat & Reinit (Demo)</button>

  <hr />

  <script>
    /***********************************************************************
     * 1. Embedded Messaging 初期化関数
     *    - Salesforce が提供する bootstrap.min.js の onload で呼び出される想定
     ***********************************************************************/
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');

      try {
        // 1. イベントハンドラが使えるか (非公式API)
        if (window.embeddedservice_bootstrap && embeddedservice_bootstrap.settings) {
          // onEventコールバックを上書き
          const originalOnEvent = embeddedservice_bootstrap.settings.onEvent;
          embeddedservice_bootstrap.settings.onEvent = function(eventData) {
            if (originalOnEvent) {
              originalOnEvent(eventData); // 元の onEvent があれば呼ぶ
            }
            console.log('[initEmbeddedMessaging] onEvent received:', eventData);

            // もし "ConversationEnded" 的なイベントが飛んでくればここで検知
            if (eventData && eventData.type === 'ConversationEnded') {
              console.warn('[initEmbeddedMessaging] Detected conversation ended event (unofficial).');
              // 下記いずれかをコメントアウト解除して使う:
              // window.location.reload();                // ページリロード
              // reinitEmbeddedMessaging();               // チャット再初期化
            }
          };
        }

        // 2. 言語設定 (例: 'ja')
        embeddedservice_bootstrap.settings.language = 'ja';

        // 3. 初期化 (OrgID, デプロイID, Embedded Service URL, オプション) を実際の値に書き換えてください
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn',  // Org ID (例)
          'MIAW4',           // Deployment ID (例)
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136', // Embedded Service URL
          {
            scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
          }
        );

        console.log('[initEmbeddedMessaging] SUCCESS: Embedded Messaging initialized.');
      } catch (err) {
        console.error('[initEmbeddedMessaging] ERROR:', err);
      }

      console.log('[initEmbeddedMessaging] END');

      // 4. フォールバックとしてDOM監視を開始 (イベントが来ない場合用)
      startDOMObserverForChatEnd();
    }

    /***********************************************************************
     * 2. フォールバック: MutationObserver を使って "会話終了" UI を検知
     *    - もし onEvent コールバックが機能しない場合、DOM変化を監視して
     *      "会話終了" メッセージやボタン等が出現したら反応する。
     *    - 実際にどういうテキストやクラス名が使われているか、要確認。
     ***********************************************************************/
    function startDOMObserverForChatEnd() {
      console.log('[startDOMObserverForChatEnd] Starting MutationObserver...');

      const targetNode = document.body;
      const observer = new MutationObserver((mutationsList) => {
        for (const mutation of mutationsList) {
          // 追加されたノードをチェック
          if (mutation.addedNodes) {
            mutation.addedNodes.forEach((node) => {
              if (node.nodeType === Node.ELEMENT_NODE) {
                const el = node;
                // 例: "会話が終了しました" と書かれた要素を見つけたら
                if (el.innerText && el.innerText.includes('会話が終了しました')) {
                  console.warn('[DOMObserver] Detected "会話が終了しました" text. Forcing reload or reinit...');
                  // window.location.reload();  // リロードする場合
                  // reinitEmbeddedMessaging(); // 再初期化する場合
                }
              }
            });
          }
        }
      });

      observer.observe(targetNode, { childList: true, subtree: true });
    }

    /***********************************************************************
     * 3. チャットを手動で終了 (デモ用)
     *    - 実際にはユーザやエージェントが終了ボタンを押す
     *      もしくは onEvent/DOM監視で終了を検知して対処する
     ***********************************************************************/
    function endChatAndReload() {
      // ここではデモ用に "removeIframe" + リロード
      forceCloseChatIframe();
      setTimeout(() => {
        window.location.reload();
      }, 1000);
    }

    function endChatAndReinit() {
      // ここではデモ用に "removeIframe" + 再初期化
      forceCloseChatIframe();
      setTimeout(() => {
        reinitEmbeddedMessaging();
      }, 1000);
    }

    /***********************************************************************
     * 4. チャットウィンドウを強制的に閉じる (iframe削除) 関数
     ***********************************************************************/
    function forceCloseChatIframe() {
      console.log('[forceCloseChatIframe] Attempting to close chat iframe via removeIframe or DOM removal...');

      // 1. もし embeddedservice_bootstrap.core.removeIframe が使えるなら呼ぶ
      if (
        window.embeddedservice_bootstrap &&
        window.embeddedservice_bootstrap.core &&
        typeof window.embeddedservice_bootstrap.core.removeIframe === 'function'
      ) {
        try {
          window.embeddedservice_bootstrap.core.removeIframe();
          console.log('[forceCloseChatIframe] Called removeIframe()');
        } catch (err) {
          console.warn('[forceCloseChatIframe] removeIframe() error:', err);
        }
      } else {
        console.log('[forceCloseChatIframe] removeIframe() not found, removing iframe from DOM...');

        // 2. iframe を直接削除
        const chatIframe = document.querySelector('iframe[data-embeddedmessaging], iframe[class*="embeddedMessaging"]');
        if (chatIframe) {
          chatIframe.remove();
          console.log('[forceCloseChatIframe] Removed chat iframe directly.');
        } else {
          console.log('[forceCloseChatIframe] No chat iframe found.');
        }
      }
    }

    /***********************************************************************
     * 5. 再初期化関数
     *   - destroy → 新しいscriptタグ読み込み → initEmbeddedMessaging()
     *   - ここでは簡易的に "既存iframeを削除" + "再度 init" だけ
     ***********************************************************************/
    function reinitEmbeddedMessaging() {
      console.log('[reinitEmbeddedMessaging] START');

      // iframeやlocalStorageを削除して完全な再初期化をしたい場合は、
      // destroyEmbeddedMessaging() の実装を流用してください。
      // ここでは簡略版として "forceCloseChatIframe" だけ呼ぶ。
      forceCloseChatIframe();

      // もし bootstrap.min.js のscriptタグ自体を抜き差しするなら下記のように書く:
      //   1) scriptタグを remove()
      //   2) setTimeoutで再insert()
      //   3) onload で initEmbeddedMessaging()

      // ここでは手短に "initEmbeddedMessaging()" を直接呼ぶ (既にスクリプトは読み込まれている前提)
      setTimeout(() => {
        if (window.embeddedservice_bootstrap) {
          console.log('[reinitEmbeddedMessaging] calling initEmbeddedMessaging()...');
          initEmbeddedMessaging();
        } else {
          console.warn('[reinitEmbeddedMessaging] embeddedservice_bootstrap not defined. Possibly script was removed?');
        }
      }, 500);

      console.log('[reinitEmbeddedMessaging] END');
    }
  </script>

  <!-- 
       6. Salesforce が提供する bootstrap.min.js の読み込み
          onload で initEmbeddedMessaging() を呼ぶ 
  -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>
</body>
</html>
