<html lang="ja">

<body>
  <h1>Salesforce Embedded Service Chat Test</h1>
  <p>
    こちらは、Salesforce の Embedded Service デプロイメントから
   2
  </p>

<script type='text/javascript'>
	function initEmbeddedMessaging() {
		try {
			embeddedservice_bootstrap.settings.language = 'ja'; // For example, enter 'en' or 'en-US'

			embeddedservice_bootstrap.init(
				'00DIS000002CjVn',
				'test2',
				'https://daihachi20240927.my.site.com/ESWtest21737531745899',
				{
					scrt2URL: 'https://daihachi20240927.my.salesforce-scrt.com'
				}
			);
		} catch (err) {
			console.error('Error loading Embedded Messaging: ', err);
		}
	};
</script>
<script type='text/javascript' src='https://daihachi20240927.my.site.com/ESWtest21737531745899/assets/js/bootstrap.min.js' onload='initEmbeddedMessaging()'></script>


</body>
</html>
