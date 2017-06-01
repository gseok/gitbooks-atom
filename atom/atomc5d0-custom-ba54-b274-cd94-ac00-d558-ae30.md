### Atom에 custom 메뉴 추가하는 방법

Atom은 Atom editor의 Top Level 메뉴 및, Top Level 메뉴의 sub메뉴를 사용자가 직접 기여하여, 사용자 자신만의 Custom 메뉴를 구성하고, 해당 메뉴를 통해 사용자가 작성한 기능을 구동 할 수 있도록, Custom 메뉴 기여 방법을 제공하고 있다. 여기서는 사용자가 자신만의 Custom 메뉴를 Atom에 추가하는 방법을 설명해 보도록 한다.

#### Atom custom package\(plugin\)을 통한 방법

Atom은 사용자가 Atom을 사용하면서, 추가적인 자신만의 기능을 제공 할 수 있도록, package기능을 제공한다. 따라서 사용자는 custom package구현을 통해서, 자신만의 추가적인 기능 구현이 가능하다. 이때, custom package의 entry point\(시작점\)으로 Top Level 메뉴 및, Top Level 메뉴의 sub메뉴를 생성 할 수 있는 방법을 제공한다.

##### custom package menu 기여 방법

구현하는 custom package에 menus 폴더 하위에, 기여하려는 custom package name와 동일한 이름으로 json 파일을 작성한다.

> 예제

* custom package name: study-atom-1
* menu정의 파일: study-atom-1.json
  * 해당 파일을 custom package의 menus 폴더 아래 위치 시킨다.
* 폴더구조
  * ![](/assets/atom-custom-menu-1.png)
* menu정의 파일의 내용을 아래 예제 코드처럼 작성한다. \(study-atom-1.json\)

  * ```js
    {
      "context-menu": {
        "atom-text-editor": [
          {
            "label": "Toggle study-atom-1",
            "command": "study-atom-1:toggle"
          }
        ]
      },
      "menu": [
        {
          "label": "TestPackages",
          "submenu": [
            {
              "label": "study-atom-1",
              "submenu": [
                {
                  "label": "Toggle",
                  "command": "study-atom-1:toggle"
                }
              ]
            }
          ]
        }
      ]
    }
    ```

위와 같이 menu정의 파일을 작성하면,  atom editor상에서 열 수 있는 context menu와 atom의 Top Level 메뉴에, 메뉴가 추가된다.

해당 메뉴를 사용자가 눌렀을때 \(사용할때\), 연결되어야 하는 동작은 `command`로 기술한다. 따라서, custom package 개발시 해당 메뉴를 사용자가 실행할때의 동작을 동일한 `command`이름으로 구현해 주어야 한다.

`command`가 실제 구현되어 있지 않아도, 메뉴 자체는 잘 기여되어서 atom에 나타나게 된다. 위 예제 코드를 사용해서 기여하면 아래 그림과 같이 메뉴가 기여된 모습을 atom에서 확인 할 수 있다. 물론 해당 package\(plugin\)이 atom에 load되어야 하는 것은 기본이다.

![](/assets/atom-custom-menu-2.png)

#### Atom original source을 통한 방법

Atom에서 custom package\(plugin\)을 사용해서 메뉴를 간단하게 기여 할 수 있다. 하지만, custom package\(plugin\)을 사용하여 메뉴를 추가할때,  **Top Level 메뉴의 경우, 항상 Help 다음 위치에 메뉴가 추가**되고, **Top Level 메뉴의 Sub Menu로 추가할때는 항상 맨 마지막 위치에 메뉴가 추가**된다. 만약 사용자가 원하는 위치\(예를 들어 File 메뉴보다 앞에 위치\)에 메뉴를 추가하고 싶은 경우, atom 원본 소스를 받아서 메뉴 구성을 수정한 다음, 수정된 코드를 빌드해서 atom을 생성해야 한다.

##### atom original source을 사용하여 custom menu 기여

atom original source을 다운받아서, menus 항목을 수정하여야 한다.

> 예제

* 메뉴 정의 파일: darwin.cson, linux.cson, win32.cson
  * os 별로 존재한다. 모든 os별로 기여하려면, 동일하게 모두 수정해 주어야 한다.
* 폴더구조

  * ![](/assets/atom-custom-menu-3.png)

* 메뉴 정의 파일

  * ```js
    'menu': [
      {
        label: '&Gseok'
        submenu: [
          { label: '&Toggle', command: 'study-atom-1:toggle'}
        ]
      }

      {
        label: '&File'
        submenu: [
          { label: 'New &Window', command: 'application:new-window' }
          { label: '&New File', command: 'application:new-file' }
          { label: '&Open File…', command: 'application:open-file' }
          { label: 'Open Folder…', command: 'application:open-folder' }
          { label: 'Add Project Folder…', command: 'application:add-project-folder' }
          {
            label: 'Reopen Project',
            submenu: [
              { label: 'Clear Project History', command: 'application:clear-project-history' }
              { type: 'separator' }
            ]
          }
          { label: 'Reopen Last &Item', command: 'pane:reopen-closed-item' }
          { type: 'separator' }
          { label: 'Se&ttings', command: 'application:show-settings' }
          { type: 'separator' }
          { label: 'Config…', command: 'application:open-your-config' }
          { label: 'Init Script…', command: 'application:open-your-init-script' }
          { label: 'Keymap…', command: 'application:open-your-keymap' }
          { label: 'Snippets…', command: 'application:open-your-snippets' }
          { label: 'Stylesheet…', command: 'application:open-your-stylesheet' }
          { type: 'separator' }
          { label: '&Save', command: 'core:save' }
          { label: 'Save &As…', command: 'core:save-as' }
          { label: 'Save A&ll', command: 'window:save-all' }
          { type: 'separator' }
          { label: '&Close Tab', command: 'core:close' }
          { label: 'Close &Pane', command: 'pane:close' }
          { label: 'Clos&e Window', command: 'window:close' }
          { type: 'separator' }
          { label: 'E&xit', command: 'application:quit' }
        ]
      }
    ....
    ]
    ```

위 예제 코드처럼,  atom의 original source의 메뉴 구성을 직접 변경하는 작업이기 때문에, 해당 파일에는 custom package에서 기여하는 메뉴를 제외한 모든 메뉴구성이 나온다. 따라서 사용자는 전체 구성을 변경 할 수 있다. 위 예제에서는 `File` 메뉴 앞에 `Gseok`라는 메뉴와 `Gseok`의 `SubMenu`로 `Toggle` 메뉴를 추가해 보았다.

**주의할점**

* atom의 original source을 수정하는 것이기 때문에 **연결될 command을 잘 결정해서 꼭 여기에 기술**해 주어야 한다.
* 해당 파일의 변경이 실제 atom에 반영되게 하기 위해서는, 해당 파일 값을 반영한 atom을 만들어야 한다. 즉, **atom original source을 빌드해서, atom을 생성 하여야 한다**.

![](/assets/atom-custom-menu-5.png)

#### command 연결

atom의 메뉴 자체는 cson이나, json을 통해서 기여가 가능하다. 메뉴를 기여할때, 해당 메뉴에 대한 동작은 command을 통해 정의하게 되어 있다. 따라서, 메뉴에서 정의한 command을 실제 등록하고, 해당 command가 요청되었을때, 수행해야하는 함수를 만들어야 한다. **command을 등록하고,  해당 command가 수행되었을대 동작하는 함수는 package\(plugin\)코드에서 개발하면 된다**. 위 메뉴 기여 방법 설명 중 original source을 build 하는 방법의 경우, original source 메뉴의 command는 해당 command에 대한 동작 구현이 각각의 atom default\(core\) package에 되어 있다.

> 예제

```js
'use babel';

import { CompositeDisposable } from 'atom';

export default {

  activate(state) {

    // Events subscribed to in atom's system can be easily cleaned up with a CompositeDisposable
    this.subscriptions = new CompositeDisposable();


    // Register command that toggles this view
    this.subscriptions.add(atom.commands.add('atom-workspace', {
      'study-atom-1:toggle': () => this.testFunction()
    }));
  }
  
  testFunction() {
    console.log('test function called');
  }
};

```

위 예제 코드는 `study-atom-1:toggle` 이라는 `command`을 등록하고 해당 `command`가 발생하면, `testFucntion`을 호출하는 package\(plugin\) 구현이다. `command`등록은, package\(plugin\)구현의 `activate` 함수에서 하여야 하고, 등록한 `commnad`을 해제 할 수 있도록, `compositeDisposable`에 `param`으로 전달해 주는 형태가 `atom`의 기본적인 package\(plugin\)구현 형태이다.

