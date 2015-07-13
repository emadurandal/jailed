Jailed — flexible JS sandbox
============================

Jailed is a small JavaScript library for running untrusted code in a
sandbox.

Jailedはサンドボックス下で信用できないコードを実行するための小さなJavaScriptライブラリです。

With Jailed you can:

Jailedを使うことで、あなたは以下のことができます：

- Load an untrusted code into a secure sandbox;

　安全なサンドボックスの中で信用できないコードをロードできます。

- Export a set of external functions into the sandbox.

　外部関数のセットをサンドボックスの中にエクスポートできます。

The untrusted code may then interract with the main application by
directly calling those functions, but the application owner decides
which functions to export, and therefore what will be allowed for the
untrusted code to perform.

信用できないコードはそれから、これらの関数を直接呼ぶことでメインアプリケーションとインタラクションすることができます。しかしアプリケーションの所有者はどの関数をエクスポートするか、つまり信用できないコードを実行するために何が許されるのか決めることができます。

The code is executed as a *plugin*, a special instance running in a
web-worker inside a sandboxed frame (in case of web-browser
environment), or as a restricted subprocess (in Node.js).

その信用できないコードは*プラグイン*（特別なインスタンス）として、サンドボックスフレームの中のweb-worker（Webブラウザ環境の場合）内、または宣言されたサブプロセスとして（Nodejsの場合）、実行されます。
　
You can use Jailed to:

あなたは、Jailedを使って以下のことができます。

- Setup a safe environment for executing untrusted code, without a
  need to create a sandboxed worker / subprocess manually;
  
　サンドボックスなWorkerやサブプロセスを手動で作ることなしに、信用できないコードを実行するための安全な環境をセットアップすることができます。

- Do that in an isomorphic way: the syntax is same both for Node.js
  and web-browser, the code works unchanged;
  
　isomorphicな方法で利用できます。つまり、Node.jsとWebブラウザで同じシンタックスが使えます。コードを変更する必要はありません。

- Execute a code from a string or from a file;

　コードを、文字列またはファイルから実行できます。

- Initiate and interrupt the execution anytime;

　いつでも実行の開始と中断ができます。

- Control the execution against a hangup or too long calculation
  times;
  
　ハングアップや長すぎる計算時間に対して、実行をきちんとコントロールできます。

- Perform heavy calculations in a separate thread
  *[Demo](http://asvd.github.io/jailed/demos/web/circle/)*
  
　別のスレッドで重い計算をさせることができます。

- Delegate to a 3rd-party code the precise set of functions to
  harmlessly operate on the part of your application
  *[Demo](http://asvd.github.io/jailed/demos/web/banner/)*
  
　あなたのアプリケーションの部分を害なく実行するように、サードパーティ製のコードに関数の精密なセットを移譲することができます。

- Safely execute user-submitted code
  *[Demo](http://asvd.github.io/jailed/demos/web/console/)*
  
　ユーザーがサブミットしたコードを安全に実行することができます。

- Export the particular set of application functions into the sandbox
  (or in the opposite direction), and let those functions be invoked
  from the other site (without a need for manual messaging) thus
  building any custom API and set of permissions.

　アプリケーションの特定の関数セットをサンドボックスに対してエクスポートすることができます（あるいは、その逆方向もできます）。そして、（手動のメッセージングをすることなしに）それらの関数をもう一方の側から呼び出すことができます。これにより、カスタムのAPIと権限のセットを構築することができます。

For instance:


```js
var path = 'http://path.to/the/plugin.js';

// exported methods, will be available to the plugin
var api = {
    alert: alert
};

var plugin = new jailed.Plugin(path, api);
```


*plugin.js:*

```js
// runs in a sandboxed worker, cannot access the main application,
// with except for the explicitly exported alert() method

// exported methods are stored in the application.remote object
application.remote.alert('Hello from the plugin!');
```

*(exporting the `alert()` method is not that good idea actually)*
(`alert()`メソッドのエクスポートは、実際には良いアイデアとはいえません)

Under the hood, an application may only communicate to a plugin
(sandboxed worker / jailed subprocess) through a messaging mechanism,
which is reused by Jailed in order to simulate the exporting of
particular functions.

内部を見てみると、アプリケーションはただ、メッセージングメカニズムを通して、プラグイン（サンドボックスworker/jailedサブプロセス）と通信しているだけです。これは、特定の関数のエクスポートをシミュレートするためにJailedによって再利用されます。

Each exported function is duplicated on the
opposite site with a special wrapper method with the same name. Upon
the wrapper method is called, arguments are serialized, and the
corresponding message is sent, which leads to the actual function
invocation on the other site.

それぞれのエクスポートされた関数は同じ名前の特別なラッパーメソッドによって、反対側で複製されます。ラッパーメソッドが呼ばれると、引数がシリアライズされ、そして対応するメッセージが送られ、結果、もう一方の側で実際の関数が呼ばれます。

If the executed function then issues a
callback, the responce message will be sent back and handled by the
opposite site, which will, in turn, execute the actual callback
previously stored upon the initial wrapper method invocation. A
callback is in fact a short-term exported function and behaves in the
same way, particularly it may invoke a newer callback in reply.

もし、実行された関数がコールバックを発行すると、応答メッセージが送り返され、そして反対側でそれがハンドリングされます。そしてそれは、最初のラッパーメソッドの呼び出しで前もって保存されていた実際のコールバックを順番に実行します。コールバックは実際は、短期間にエクスポートされた関数で、同じように振る舞います。特に、応答の際に新しいコールバックを呼び出すかもしれません。

### Installation

For the web-browser environment — download and unpack the
[distribution](https://github.com/asvd/jailed/releases/download/v0.2.0/jailed-0.2.0.tar.gz), or install it using [Bower](http://bower.io/):

Webブラウザ環境では、ここからからダウンロードして解凍してください。または、Bowerを使ってインストールしてください。

```sh
$ bower install jailed
```

Load the `jailed.js` in a preferrable way. That is an UMD module, thus
for instance it may simply be loaded as a plain JavaScript file using
the `<script>` tag:

jailed.jsを好きな方法で読み込んでください。これはUMDモジュールです。なので、たとえば`<script>`タグを使ってプレインJavaScriptファイルを単純にロードします。

```html
<script src="jailed/jailed.js"></script>
```

For Node.js — install Jailed with npm:

```sh
$ npm install jailed
```

and then in your code:

```js
var jailed = require('jailed');
```

Optionally you may load the script from the
[distribution](https://github.com/asvd/jailed/releases/download/v0.2.0/jailed-0.2.0.tar.gz):

オプションとして、ここからスクリプトをロードすることもできます。

```js
var jailed = require('path/to/jailed.js');
```

After the module is loaded, the two plugin constructors are available:
`jailed.Plugin` and `jailed.DynamicPlugin`.

モジュールをロードしたら、２つのプラグインコンストラクター`jailed.Plugin` と `jailed.DynamicPlugin`が利用可能になります。

### Usage

The messaging mechanism reused beyond the remote method invocation
introduces some natural limitations for the exported functions and
their usage (nevertheless the most common use-cases are still
straightforward):

リモートメソッド呼び出しを超えて再利用されるメッセージングメカニズムは、エクスポートされた関数やそれらの利用（それでもやはり、ほとんどの共通のユースケースは未だに直接的です）にいくつかの自然な制限をもたらします：

- Exported function arguments may only be either simple objects (which
  are then serialized and sent within a message), or callbacks (which
  are preserved and replaced with special identifiers before
  sending). Custom object instance may not be used as an argument.
  
エクスポートされた関数の引数はシンプルなオブジェクト（それらはシリアライズされてメッセージの中に送られます）なだけか、コールバック（それらは保護され、送られる前に特別な識別値で置き換えられます）です。カスタムなオブジェクトインスタンスは引数として使われることはありません。

- A callback can not be executed several times, it will be destroyed
  upon the first invocation.
  
コールバックを複数回実行することはできません。最初の呼び出しのあと、コールバックは破棄されます。

- If several callbacks are provided, only one of them may be called.

複数のコールバックが供給されると、それらのうちの一つだけが呼び出されます。

- Returned value of an exported function is ignored, result should be
  provided to a callback instead.

エクスポートされた関数の返り値が無視されるなら、結果は代わりにコールバックに供給されるべきです。

*In Node.js the
 [send()](http://nodejs.org/api/child_process.html#child_process_child_send_message_sendhandle)
 method of a child process object is used for transfering messages,
 which serializes an object into a JSON-string. In a web-browser
 environment, the messages are transfered via
 [postMessage()](https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage)
 method, which implements [the structured clone
 algorithm](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/The_structured_clone_algorithm)
 for the serialization. That algorithm is more capable than JSON (for
 instance, in a web-browser you may send a RegExp object, which is not
 possible in Node.js). [More details about structured clone algorithm
 and its comparsion to
 JSON](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/The_structured_clone_algorithm).*

Node.jsでは、子プロセスオブジェクトのsend()メソッドがメッセージの転送に使われます。このsend()メソッドはオブジェクトをJSON文字列にシリアライズします。ブラウザ環境では、メッセージはpostMessage()メソッドを通して転送されます。（postMessage()メソッドは、シリアライゼーションのための構造化されたクローンアルゴリズムを実装しています）。このアルゴリズムはJSONよりも多くの機能性があります（例えば、ブラウザ環境ではあなたはRegExpオブジェクトを送ることができます。Node.jsではそれができません）。より詳細な構造化されたクローンアルゴリズムと、それのJSONとの比較。

A plugin object may be created either from a string containing a
source code to be executed, or with a path to the script. To load a
plugin code from a file, create the plugin using `jailed.Plugin`
constructor and provide the path:

プラグインオブジェクトはソースコードを含む文字列から、またはスクリプトのパスから、それを実行されるために作られます。ファイルからプラグインコードをロードするには、`jailed.Plugin`コンストラクタを使ってパスを渡すことで、プラグインを作成します。

```js
var path = 'http://path.to/some/plugin.js';

// set of methods to be exported into the plugin
var api = {
    alert: alert
}

var plugin = new jailed.Plugin(path, api);
```


*plugin.js:*

```js
application.remote.alert('Hello from the plugin!');
```


Creating a plugin from a string containing a code is very similar,
this is performed using `jailed.DynamicPlugin` constructor:


```js
var code = "application.remote.alert('Hello from the plugin!');";

var api = {
    alert: alert
}

var plugin = new jailed.DynamicPlugin(code, api);
```


The second `api` argument provided to the `jailed.Plugin` and
`jailed.DynamicPlugin` constructors is an interface object with a set
of functions to be exported into the plugin. It is also possible to
export functions in the opposite direction — from a plugin to the main
application. It may be used for instance if a plugin provides a method
to perform a calculation. In this case the second argument of a plugin
constructor may be omitted. To export some plugin functions, use
`application.setInterface()` method in the plugin code:

`jailed.Plugin`と`jailed.DynamicPlugin`に渡される２つ目の引数`api`はプラグイン内部に公開される関数のセットのインターフェースオブジェクトです。この仕組みでは、反対の方向（プラグインからメインアプリケーション）に関数をエクスポートすることも可能です。これはたとえば、プラグインが計算を実行するためのメソッドを提供する場合などです。このケースでは、プラグインのコンストラクタへの２つ目の引数は省略します。いくつかのプラグインの関数をエクスポートする場合には、プラグインのコードで`application.setInterface()`メソッドを使います。


```js
// create a plugin
var path = "http://path.to/some/plugin.js";
var plugin = new jailed.Plugin(path);

// called after the plugin is loaded
var start = function() {
    // exported method is available at this point
    plugin.remote.square(2, reportResult);
}

var reportResult = function(result) {
    window.alert("Result is: " + result);
}

// execute start() upon the plugin is loaded
plugin.whenConnected(start);
```

*plugin.js:*

```js
// provides the method to square a number
var api = {
    square: function(num, cb) {
        // result reported to the callback
        cb(num*num);
    }
}

// exports the api to the application environment
application.setInterface(api);
```

In this example the `whenConnected()` plugin method is used at the
application site: that method subscribes the given function to the
plugin connection event, after which the functions exported by the
plugin become accessible at the `remote` property of a plugin.

このサンプルでは、`whenConnected()`プラグインのメソッドがアプリケーションサイトで使われている：そのメソッドは与えられた関数をプラグイン接続イベントにサブスクライブする。そうすると、そのプラグインでエクスポートされた関数はプラグインの`remote`プロパティからアクセス可能になる。

The `whenConnected()` method may be used as many times as needed and
thus subscribe several handlers for a single connection event. For
additional convenience, it is also possible to set a connection
handler even after the plugin has already been connected — in this
case the handler is issued immediately (yet asynchronously).

`whenConnected()`メソッドは、必要に応じていくらでも使うことになり、またこのような、単一の接続イベントのための複数のハンドラーをサブスクライブする。追加的な利便性のため、プラグインがすでに接続された後になってでも、接続ハンドラーをセットすることが可能である。このケースでは、ハンドラーはすぐに発行されている（まだ非同期ではない）。

When a plugin code is executed, a set of functions exported by the
application is already prepared. But if one of those functions is
invoked, it will actually be called on the application site. If in
this case the code of that function will try to use a function
exported by the plugin, it may not be prepared yet. To solve this, the
similar `application.whenConnected()` method is available on the
plugin site. The method works same as the one of the plugin object:
the subscribed handler function will be executed after the connection
is initialized, and a set of functions exported by each site is
available on the opposite site.

プラグインコードが実行されるとき、アプリケーションによってエクスポートされた関数のセットはすでに準備が整っている。しかし、それらの関数の１つが呼び出されると、それは実際はアプリケーションサイト上で呼ばれることになる。もし、この場合にその関数のコードがプラグインによってエクスポートされた関数を使おうと試みた場合、その関数はまだ準備ができていないかもしれない。これを解決するために、`application.whenConnected()`と類似したメソッドがプラグインサイト上で利用可能である。そのメソッドはプラグインオブジェクトの１つと同じように動作する：接続が初期化されたあとにサブスクライブされたハンドラー関数が実行される。そして、それぞれのサイトでエクスポートされた関数のセットは両方のサイトから利用可能になる。

Therefore:

それゆえ：

- If you need to load a plugin and supply it with a set of exported
  functions, simply provide those functions into the plugin
  constructor, and then access those at `applictaion.remote` property
  on the plugin site — the exported functions are already prepared
  when the plugin code is exectued.
  
もしあなたがプラグインをロードし、エクスポートされた関数のセットをそのプラグインに供給する必要があるなら、単純にそれらの関数をプラグインのコンストラクタに渡してください。そして、プラグインサイトの`applictaion.remote`プロパティを通してそれらにアクセスしてください。エクスポートされた関数はすでにプラグインコードが実行された辞典で準備ができています。

- If you need to load a plugin and use the functions it provides
  through exporting, set up a handler using `plugin.whenConnected()`
  method on the application site. After the event is fired, the
  functions exported by the plugin are available at its `remote`
  property of the plugin object;.
  
　もし、あなたがプラグインをロードし、そのプラグインがエクスポートを通して供給している関数を使いたいなら、アプリケーションサイトで`plugin.whenConnected()`メソッドを使ってハンドラーをセットアップしてください。そのイベントが発火した後で、プラグインによってエクスポートされた関数はプラグインオブジェクトの`remote`プロパティを通して利用可能になります。

- If both application and a plugin use the exported functions of each
  other, *and* the communication is initiated by the plugin, you will
  most likely need to use the `application.whenConnected()` method on
  the plugin site before initiating the communication, in order to
  make sure that the functions exported by the plugin are already
  available to the application.

　もし、アプリケーションとプラグインがそれぞれのエクスポートされた関数を使いたいなら、そしてその通信がプラグインによって初期化されるなら、あなたは通信を始める前にプラグインサイトの`application.whenConnected()`メソッドを使う必要があるでしょう。これは、プラグインによってエクスポートされた関数がすでにアプリケーションで利用可能になっていることを保証するために必要なことです。

To disconnect a plugin, use the `disconnect()` method: it kills a
worker / subprocess immediately without any chance for its code to
react.

プラグインを接続解除するためには、`disconnect()`メソッドを使います。これは、プラグインのコードが反応するチャンスを与えずに、worker / subprocessをすぐに殺します。

A plugin may also disconnect itself by calling the
`application.disconnect()` method.

プラグイン自身も`application.disconnect()`メソッドを呼ぶことで接続を解除できます。

In addition to `whenConnected()` method, the plugin object also
provides similar `whenFailed()` and `whenDisconnected()` methods:

`whenConnected()`メソッドに加え、プラグインオブジェクトは`whenFailed()`と`whenDisconnected()`メソッドも提供しています。

- `whenFailed()` subscribes a handler function to the connection
  failure event, which happens if there have been some error during
  the plugin initialization, like a network problem or a syntax error
  in the plugin initialization code.
  
　`whenFailed()`はハンドラー関数を接続失敗イベントにサブスクライブします。この接続失敗イベントは、プラグインの初期化処理で何らかのエラーが発生した場合に起こります。たとえばネットワーク問題やプラグインの初期化コードのシンタックスエラーなどです。

- `whenDisconnected()` subscribes a function to the disconnect event,
  which happens if a plugin was disconnected by calling the
  `disconnect()` method, or a plugin has disconnected itself by
  calling `application.disconnect()`, or if a plugin failed to
  initialize (along with the failure event mentioned above). After the
  event is fired, the plugin is not usable anymore.
  
　`whenDisconnected()` は関数を接続解除イベントにサブスクライブします。この接続解除イベントは`disconnect()`メソッドやプラグイン自身による`application.disconnect()`メソッドが呼ばれることで、プラグインの接続が解除されたときに起こります。あるいは、プラグインが初期化に失敗した時（前述の失敗イベントで説明したような）にも呼ばれます。このイベントが発火した後は、プラグインはもはや使えなくなります。

Just like as for `whenConnected()` method, those two methods may also
be used several times or even after the event has actually been fired.

ちょうど`whenConnected()`メソッドのように、これらの２つのメソッドは、複数回またはイベントが実際に発火した後にも呼ばれることがあります。

### Security

This is how the sandbox is built:

サンドボックスがどのように実現されているか：

##### In Node.js:

- A Node.js subprocess is created by the Jailed library;

　Node.jsのサブプロセスがJailedライブラリによって作成されます。

- the subprocess (down)loads the file containing an untrusted code as
  a string (or, in case of `DynamicPlugin`, simply uses the provided
  string with code)
  
　このサブプロセスは文字列として信用できないコードを含んだファイルをダウンロードします。（あるいは、`DynamicPlugin`の場合、提供されたコードを含む文字列を単に使います）。

- then `"use strict";` is appended to the head of that code (in order
  to prevent breaking the sandbox using `arguments.callee.caller`);
  
　そして`"use strict";`がコードの戦闘に追加されます（これは、`arguments.callee.caller`が使われてサンドボックスが壊されることを防ぐためです）。

- finally the code is executed using `vm.runInNewContext()` method,
  where the provided sandbox only exposes some basic methods like
  `setTimeout()`, and the `application` object for messaging with the
  application site.

　最後に、`vm.runInNewContext()`メソッドによってコードが実行されます。これは、`setTimeout()`などのいくつかの基本的なメソッドと、アプリケーションサイトとのメッセージングのための`application`オブジェクトを公開するサンドボックスを提供するものです。

##### In a web-browser:

- a [sandboxed
iframe](http://www.html5rocks.com/en/tutorials/security/sandboxed-iframes/)
is created with its `sandbox` attribute only set to `"allow-scripts"`
(to prevent the content of the frame from accessing anything of the
main application origin);

（"allow-scripts"だけがセットされたsandboxアトリビュートを持つ）サンドボックス化されたiframeが生成されます。（これは、フレームのコンテンツがメインアプリケーションオリジンの何らかにアクセスするようなことを防ぐためです。）

- then a web-worker is started inside that frame;

　そして、web-workerがiframeの中でスタートします。

- finaly the code is loaded by the worker and executed.

　最後に、コードがworkerによってロードされて実行されます。

*Note: when Jailed library is loaded from the local source (its path
 starts with `file://`), the `"allow-same-origin"` permission is added
 to the `sandbox` attribute of the iframe. Local installations are
 mostly used for testing, and without that permission it would not be
 possible to load the plugin code from a local file. This means that
 the plugin code has an access to the local filesystem, and to some
 origin-shared things like IndexedDB (though the main application page
 is still not accessible from the worker). Therefore if you need to
 safely execute untrusted code on a local system, reuse the Jailed
 library in Node.js.*

注意：Jailedライブラリがローカルソース（`file://`から始まるパス）からロードされた場合、`"allow-same-arigin"`パーミッションがiframeの`sandbox`属性に加えられます。ローカルインストールはほとんどテストのために使われます。そして、そのパーミッションなしでは、ローカルファイルからプラグインコードをロードすることができません。このことは、そのプラグインコードはローカルファイルシステムにアクセスできることを意味します。そして、IndexedDBのような、いくつかのorigin-sharedのモノにもアクセスできることを意味します。（たとえ、メインアプリケーションページがworkerからアクセスできる状態でなくても）。それゆえ、もしあなたが安全に信用できないコードをローカルシステムで実行させたいなら、Node.jsの中でJailedライブラリを利用してください。

follow me on twitter: https://twitter.com/asvd0

