# 在Tuanjie1.7.1及以上使用自定义Webview
## 1. Tuanjie项目Assets/Plugins/OpenHarmony中增加以下文件
#### CustomWebviewUtils.ets
```
import { WebviewInfo } from './pages/components/CustomWebview';
import display from '@ohos.display';

export class CustomWebviewUtils {
  static _instance: CustomWebviewUtils;

  static getInstance(): CustomWebviewUtils {
    if (!CustomWebviewUtils._instance) {
      CustomWebviewUtils._instance = new CustomWebviewUtils();
    }

    return CustomWebviewUtils._instance;
  }

  public static get webviewInfos(): WebviewInfo {
    return AppStorage.get("customWebviewInfo") as WebviewInfo
  }

  public static set webviewInfos(value: WebviewInfo) {
    AppStorage.setOrCreate("customWebviewInfo", value);
  }

  public static get controller(): WebviewController {
    return AppStorage.get("customWebviewController") as WebviewController
  }

  static CreateWebView() {
    CustomWebviewUtils.webviewInfos.occupyble = true;
  }

  static RemoveWebview() {
    CustomWebviewUtils.webviewInfos.reset();
  }

  static LoadURL(url: string) {
    CustomWebviewUtils.webviewInfos.url = url;
    CustomWebviewUtils.controller.loadUrl(url);
  }

  static LoadHTMLString(contents: string, baseUrl: string) {
    CustomWebviewUtils.controller.loadData(contents, "text/html", "UTF-8", baseUrl);
  }

  static LoadData(contents: string, baseUrl: string) {
    CustomWebviewUtils.controller.loadData(contents, "text/html", "UTF-8", baseUrl);
  }

  static EvaluateJS(jsContents: string) {
    try {
      CustomWebviewUtils.controller.runJavaScript(jsContents).then((result: ESObject) => {
        if (result) {
          console.info(`The return value is: ${result}`)
        }
      })
    } catch (error) {
      console.error('Failed to evaluate JavaScript. Cause: ' + JSON.stringify(error));
    }
  }

  static SetVisibility(visible: boolean) {
    CustomWebviewUtils.webviewInfos.visible = visible;
  }

  static SetMargins(ml: number, mt: number, mr: number, mb: number) {
    let viewInfo: WebviewInfo = CustomWebviewUtils.webviewInfos;
    viewInfo.x = ml;
    viewInfo.y = mt;
    let w = display.getDefaultDisplaySync().width - ml - mr;
    let h = display.getDefaultDisplaySync().height - mt - mb;
    viewInfo.w = w;
    viewInfo.h = h;
  }

  static Reload() {
    CustomWebviewUtils.controller.refresh();
  }

  static StopLoading() {
    CustomWebviewUtils.controller.stop();
  }

  static GoForward() {
    if (CustomWebviewUtils.controller.accessForward()) {
      CustomWebviewUtils.controller.forward()
    }
  }

  static GoBack() {
    if (CustomWebviewUtils.controller.accessBackward()) {
      CustomWebviewUtils.controller.backward()
    }
  }
}

```

#### WebviewControlBridge.etslib
```
import { POST_MESSAGE_TO_HOST } from 'classesLib';

export class WebviewControlBridge {
  private messageCallback: Function | null = null;

  private constructor() {
    globalThis.workerPort.addEventListener("message", async (e: ESObject) => {
      let msg: ESObject = JSON.parse(JSON.stringify(e));
      if (msg.data.type == "WebControllerAttached") {
        console.info(">>>> receive WebControllerAttached message.")
        if (this.messageCallback != null) {
          this.messageCallback();
        }
      }
    });
  }

  public CreateWebview(ml: number, mt: number, mr: number, mb: number, visible: boolean, callback: Function) {
    console.info('>>>> CreateWebview method enter');
    POST_MESSAGE_TO_HOST({ type: "RUN_ON_UI_THREAD_USER_EVENT", moduleName: "tuanjieLib",funcName: "CustomWebviewUtils.CreateWebView", args: [] });
    this.SetMargins(ml, mt, mr, mb);
    this.SetVisibility(visible);
    this.messageCallback = callback;
  }

  public RemoveWebview() {
    console.info('>>>> RemoveWebview method enter');
    POST_MESSAGE_TO_HOST({ type: "RUN_ON_UI_THREAD_USER_EVENT", moduleName: "tuanjieLib",funcName: "CustomWebviewUtils.RemoveWebview", args: [] });
  }

  public LoadURL(url: string): void {
    console.info('>>>> LoadURL method enter url is ' + url);
    POST_MESSAGE_TO_HOST({ type: "RUN_ON_UI_THREAD_USER_EVENT", moduleName: "tuanjieLib",funcName: "CustomWebviewUtils.LoadURL", args: [url] });
  }

  public LoadHTMLString(contents: string, baseUrl: string): void {
    console.info('>>>> LoadHTMLString method enter');
    POST_MESSAGE_TO_HOST({
      type: "RUN_ON_UI_THREAD_USER_EVENT",moduleName: "tuanjieLib",
      funcName: "CustomWebviewUtils.LoadHTMLString",
      args: [contents, baseUrl]
    });
  }

  public LoadData(contents: string, baseUrl: string): void {
    console.info('>>>> LoadData method enter');
    POST_MESSAGE_TO_HOST({
      type: "RUN_ON_UI_THREAD_USER_EVENT",moduleName: "tuanjieLib",
      funcName: "CustomWebviewUtils.LoadData",
      args: [contents, baseUrl]
    });
  }

  public EvaluateJS(jsContents: string): void {
    console.info('>>>> EvaluateJS method enter ' + jsContents);
    POST_MESSAGE_TO_HOST({
      type: "RUN_ON_UI_THREAD_USER_EVENT",moduleName: "tuanjieLib",
      funcName: "CustomWebviewUtils.EvaluateJS",
      args: [jsContents]
    });
  }

  public Reload(): void {
    console.info('>>>> Reload method enter');
    POST_MESSAGE_TO_HOST({
      type: "RUN_ON_UI_THREAD_USER_EVENT",moduleName: "tuanjieLib",
      funcName: "CustomWebviewUtils.Reload",
      args: []
    });
  }

  public StopLoading(): void {
    console.info('>>>> StopLoading method enter');
    POST_MESSAGE_TO_HOST({
      type: "RUN_ON_UI_THREAD_USER_EVENT",moduleName: "tuanjieLib",
      funcName: "CustomWebviewUtils.StopLoading",
      args: []
    });
  }

  public GoForward(): void {
    console.info('>>>> GoForward method enter');
    POST_MESSAGE_TO_HOST({
      type: "RUN_ON_UI_THREAD_USER_EVENT",moduleName: "tuanjieLib",
      funcName: "CustomWebviewUtils.GoForward",
      args: []
    });
  }

  public GoBack(): void {
    console.info('>>>> GoBack method enter');
    POST_MESSAGE_TO_HOST({
      type: "RUN_ON_UI_THREAD_USER_EVENT",moduleName: "tuanjieLib",
      funcName: "CustomWebviewUtils.GoBack",
      args: []
    });
  }

  public SetVisibility(visible: boolean): void {
    console.info('>>>> SetVisibility method enter, visible is ' + visible);
    POST_MESSAGE_TO_HOST({
      type: "RUN_ON_UI_THREAD_USER_EVENT",moduleName: "tuanjieLib",
      funcName: "CustomWebviewUtils.SetVisibility",
      args: [visible]
    });
  }

  public SetMargins(ml: number, mt: number, mr: number, mb: number): void {
    console.info('>>>> SetMargins method enter');
    POST_MESSAGE_TO_HOST({
      type: "RUN_ON_UI_THREAD_USER_EVENT",moduleName: "tuanjieLib",
      funcName: "CustomWebviewUtils.SetMargins",
      args: [ml, mt, mr, mb]
    });
  }

  public static getInstance() {
    return new WebviewControlBridge();
  }
}

export function RegisterWebviewControlBridge() {
  let register: Record<string, Object> = {};
  register["WebviewControlBridge"] = WebviewControlBridge;
  return register;
}
```

## 2. 导出DevEco工程，新增CustomWebview组件，放置于**tuanjieLib**\src\main\ets\pages\components\CustomWebview.ets
```
import web_webview from '@ohos.web.webview'

import { POST_MESSAGE_TO_WORKER } from 'classesLib';

@Observed
export class WebviewInfo {
  // position
  public x: number = 0;
  public y: number = 0;

  // size
  public w: number = 0;
  public h: number = 0;

  // url
  public url: string = '';

  public visible: boolean = false;
  public occupyble: boolean = false;

  constructor(x: number, y: number, w: number, h: number) {
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
  }

  public reset() {
    this.x = 0;
    this.y = 0;
    this.w = 0;
    this.h = 0;
    this.url = '';
    this.visible = false;
    this.occupyble = false;
  }
}

@Component
export struct CustomWebview {
  @StorageLink("customWebviewInfo") viewInfo: WebviewInfo = new WebviewInfo(0,0,0,0);
  @StorageLink("customWebviewController") webviewController: WebviewController = new web_webview.WebviewController();
  build() {
    if(this.viewInfo.visible) {
      Web({
        src: this.viewInfo.url,
        controller: this.webviewController
      })
        .position({ x: px2vp(this.viewInfo.x), y: px2vp(this.viewInfo.y) })
        .width(px2vp(this.viewInfo.w))
        .height((px2vp(this.viewInfo.h)))
        .border({ width: 1 })
        .domStorageAccess(true)
        .databaseAccess(true)
        .imageAccess(true)
        .javaScriptAccess(true)
        .visibility(this.viewInfo.occupyble ? (this.viewInfo.visible ? Visibility.Visible : Visibility.Hidden) : Visibility.None)
        .onControllerAttached(() => {
          POST_MESSAGE_TO_WORKER({
            'type': 'WebControllerAttached'
          },"")
        })
    }
  }
}
```
## 3. tuanjieLib\exported.ets文件中新增导出声明
```
export { CustomWebview, WebviewInfo } from './src/main/ets/pages/components/CustomWebview'
```

## 4. entry\src\main\ets\pages\Index.ets中加入CustomWebview组件
```
……
import { CustomWebview } from 'tuanjieLib';
……
@Entry
@Component
struct TuanjiePlayer {
  ……
  build() {
    Column() {
      Row() {
        ……
      }
        ……
      CustomWebview();
    }
    ……
  }
    ……
}


```

## 5. 使用以下C#脚本来调用Webview能力
```
using System;
using UnityEngine;

public class CustomeWebview
{
    private OpenHarmonyJSClass webviewControlBridgeClass;
    private OpenHarmonyJSObject webviewControlBridge;

    public CustomeWebview()
    {
        webviewControlBridgeClass = new OpenHarmonyJSClass("WebviewControlBridge");
        webviewControlBridge = webviewControlBridgeClass.CallStatic<OpenHarmonyJSObject>("getInstance");
    }

    public void CreateWebview(int ml, int mt, int mr, int mb, Boolean visible)
    {
        OpenHarmonyJSCallback webviewCallBack = new OpenHarmonyJSCallback(WebviewCallBack);
        webviewControlBridge.Call("CreateWebview", ml, mt, mr, mb, visible, webviewCallBack);
    }
    
    public object WebviewCallBack(params OpenHarmonyJSObject[] args)
    {
        webviewControlBridge.Call("LoadURL", "https://www.baidu.com/");
        return null;
    }

    public void RemoveWebview()
    {
        webviewControlBridge.Call("RemoveWebview");
    }

    public void LoadURL(string url)
    {
        webviewControlBridge.Call("LoadURL", url);
    }

    public void LoadHTMLString(string contents, string baseUrl)
    {
        webviewControlBridge.Call("LoadHTMLString", contents, baseUrl);
    }

    public void LoadData(string contents, string baseUrl)
    {
        webviewControlBridge.Call("LoadData", contents, baseUrl);
    }

    public void EvaluateJS(string jsContents)
    {
        webviewControlBridge.Call("EvaluateJS", jsContents);
    }

    public void SetVisibility(Boolean visible)
    {
        webviewControlBridge.Call("SetVisibility", visible);
    }

    public void SetMargins(int ml, int mt, int mr, int mb)
    {
        webviewControlBridge.Call("SetMargins", ml, mt, mr, mb);
    }

    public void Reload()
    {
        webviewControlBridge.Call("Reload");
    }

    public void StopLoading()
    {
        webviewControlBridge.Call("StopLoading");
    }

    public void GoForward()
    {
        webviewControlBridge.Call("GoForward");
    }

    public void GoBack()
    {
        webviewControlBridge.Call("GoBack");
    }
}

```
