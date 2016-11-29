PhoneGap IntelliJ Plugin Embedded Example
==========================================

This is a simple example of how to embed Cordova in a view.  Right now the plugin just
sets up the libraries, the Node JS plugin system and the config.xml so that the project is a 
cordova project.  There is still some Java setup required.

1. Add a view to your layout.  In this example, we add it to res/layouts/content_mail.xml

```
    <org.apache.cordova.engine.SystemWebView
        android:id="@+id/WebViewComponent"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
```

2. In the MainActivity, add the following code to the class:
```
    public CordovaWebView webInterface;
    private CordovaInterfaceImpl cordovaInterface = new CordovaInterfaceImpl(this);
    private String TAG = "ConFooActivity";
```

3. Then add the following code to onCreate
```
    //Set up the webview
    ConfigXmlParser parser = new ConfigXmlParser();
    parser.parse(this);

    SystemWebView webView = (SystemWebView) findViewById(R.id.WebViewComponent);
    webInterface = new CordovaWebViewImpl(new SystemWebViewEngine(webView));
    webInterface.init(cordovaInterface, parser.getPluginEntries(), parser.getPreferences());

    webView.loadUrl(parser.getLaunchUrl());
```

4.  You will need to implement onDestroy if it hasn't already been implemented.  If it has, just 
insert the contents of this method into your existing onDestroy
```
    // This is still required by Cordova
    @Override
    public void onDestroy()
    {
        super.onDestroy();
        PluginManager pluginManager = webInterface.getPluginManager();
        if(pluginManager != null)
        {
            pluginManager.onDestroy();
        }

    }
```

5. The same goes for onActivityResult and onPluginPermissionResult.  If you are using plugins that require the use of intents
please make sure that you are familiar with the Android Life Cycle and how to use Cordova in this way.

```
    /**
     * Called when an activity you launched exits, giving you the requestCode you started it with,
     * the resultCode it returned, and any additional data from it.
     *
     * @param requestCode       The request code originally supplied to startActivityForResult(),
     *                          allowing you to identify who this result came from.
     * @param resultCode        The integer result code returned by the child activity through its setResult().
     * @param intent            An Intent, which can return result data to the caller (various data can be attached to Intent "extras").
     */
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent intent) {
        super.onActivityResult(requestCode, resultCode, intent);
        cordovaInterface.onActivityResult(requestCode, resultCode, intent);
    }

    /**
     * Called by the system when the user grants permissions!
     *
     * Note: The fragment gets priority over the activity, since the activity doesn't call
     * into the parent onRequestPermissionResult, which is why there's no override.
     *
     * @param requestCode
     * @param permissions
     * @param grantResults
     */
    public void onRequestPermissionsResult(int requestCode, String permissions[],
                                           int[] grantResults) {
        try
        {
            cordovaInterface.onRequestPermissionResult(requestCode, permissions, grantResults);
        }
        catch (JSONException e)
        {
            Log.d(TAG, "JSONException: Parameters fed into the method are not valid");
            e.printStackTrace();
        }

    }

```

Once this is done, the WebView should be wired up and should work like an existing WebView.  We are working
on making this process easier, and this is a finished project created using the following process.

