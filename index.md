<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>Embedded Messaging: Log ConversationEnded Event</title>
</head>
<body>
  <h1>Salesforce Embedded Messaging - Log ConversationEnded Event</h1>
  <p>
    チャット終了イベント (<code>ConversationEnded</code>) が発生したら、<br>
    そのイベントオブジェクト (<code>eventData</code>) をコンソールに詳細ログを出すサンプルです。
  </p>

  <script>
    function initEmbeddedMessaging() {
      console.log('[initEmbeddedMessaging] START');
      try {
        // 1) onEvent コールバックを設定
        if (window.embeddedservice_bootstrap && embeddedservice_bootstrap.settings) {
          // もともと登録されている onEvent (ある場合) を退避
          const originalOnEvent = embeddedservice_bootstrap.settings.onEvent;

          embeddedservice_bootstrap.settings.onEvent = function(eventData) {
            // もともとのコールバックがあれば呼ぶ
            if (originalOnEvent) {
              originalOnEvent(eventData);
            }
            // すべてのイベントをログに出す
            console.log('[onEvent] Received eventData:', eventData);

            // イベントタイプが "ConversationEnded" かどうか判定
            if (eventData && eventData.type === 'ConversationEnded') {
              console.warn('[onEvent] ConversationEnded event fired!', eventData);
              // TODO: ここで再初期化、強制クローズ、ページリロードなどを行いたい場合は実装する
            }
          };
        }

        // 2) 言語設定
        embeddedservice_bootstrap.settings.language = 'ja';

        // 3) メイン初期化 (実際の組織ID, デプロイID, URLに書き換えてください)
        embeddedservice_bootstrap.init(
          '00DIS000002CjVn', // Org ID
          'MIAW4',           // Deployment ID
          'https://daihachi20240927.my.site.com/ESWMIAW41737545576136',
          { scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com' }
        );

        console.log('[initEmbeddedMessaging] SUCCESS: Chat initialized.');
      } catch (err) {
        console.error('[initEmbeddedMessaging] ERROR:', err);
      }

      console.log('[initEmbeddedMessaging] END');
    }
  </script>

  <!-- 
       Salesforce Embedded Messaging bootstrap 
       onload="initEmbeddedMessaging()" で初期化を呼び出す
  -->
  <script
    type="text/javascript"
    src="https://daihachi20240927.my.site.com/ESWMIAW41737545576136/assets/js/bootstrap.min.js"
    onload="initEmbeddedMessaging()"
  ></script>
</body>
</html>
