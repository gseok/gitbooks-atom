### atom package 구동 flow

atom package\(plugin\)의 구동 flow에 대하여 간략하게 정리한다.

---

> `package loading`

1\) Atom을 실행한다.

2\) Atom이 실행되면서, `package loading`을 시작한다.

* Atom이 적용해야 하는 package는 특정 폴더 아래 있어야 한다.
  * window: `C:\Users\<user_name>\.atom\packages`
* Atom은 특정 폴더 아래 있는 모든 자식 폴더를 읽어서 package인지 확인한다.

3\) Atom은 특정 폴더 아래\(반드시 바로 아래\) `package.json` 파일을 읽어온다.

4\) Atom은 package.json에 정의한 내용을 읽어서 atom에 기여하는 부분을 로드한다.

* package가 atom package\(plugin\)으로 구동될때 필요한 정보를 로드한다.
  * keymaps\(shortcut\), menus, styles
  * main\(entry point\)
* **keymaps, menus, styles가 package.json에 명시되지 않은 경우:** 해당 이름의 폴더에서 정보를 로드한다.
* **main이 명시되지 않은 경우:** 기본으로 index.js \| index.coffee을 entry point로 적용된다.
* **activationCommnads가 명시되지 않은 경우:** Atom의 시작과 동시에 package을 activate한다.
* **activationHooks가 명시된 경우:** 해당 hook이 tirgger되면 그때 package loading을 한다.

5\) Atom은 `package loading` 과정을 완료한다.

---

> `package activate`

1\) package\(package.json\)에서 정의한 `activationCommands`가 호출되면 activate을 진행한다.

* `activationCommands`가 정의되지 않은경우, package loading과정 이후, package을 activate 한다.

2\) package\(package.json\)에서 정의한 main\(entry point\)에서 노출하고 있는 `activate` 함수를, `atom`이 호출한다.

* main entry point는 반드시 `activate` 함수를 구현하고 `export` 하여야 한다.
* activate함수

  * view 생성 및 등록\(atom.workspace.addModalPanel\)

  * command\(atom.commands.add\)등록

  * editor operner\(atom.workspace.addOpener\)등록

  * 등록된 리소스 핸들러를 해지하기위한, disposable 사용 \(CompositeDisposable\)

* activate함수에서 실제 해당 package\(plugin\)에서 해야하는 동작을 구현하고, atom의 command, editor등의 연결을 설정한다.

3\) activationCommands을 호출했을때, 실제 구동되어야 함수는, `activate` 함수가 호출 된 이 후에 호출된다.

* atom은 모든 package와 해당 package에서 기여하는 command list을 가지고 있다.
* 따라서 package가 아직 activation되지 않은 상태에서도, 해당 command 호출까지는 가능
* 해당 command가 호출되었을때, 아직 해당 command을 기여한 package가 activation상태가 아니면, 해당 command을 기여한 package의 `activate`을 먼저 호출 한다.
* 이미 `activate`가 호출되어서, atom이 해당 command에 대한 handler\(function\)을 알고 있는 경우, 해당 함수를 바로 호출한다.

4\) view에 대한 serialize가 필요한 경우 main\(entry point\)에서 구현\(activate구현 위치와 동일위치\)한 serialize를 atom이 호출한다.

* serialize 함수는 object를 리턴해야 한다.
  * 즉 { myStateKey: this.view.reialize\(\) } 형태
* object의 key는 atom이 activate함수를 호출할때 전달하는 state object param에 적용된다.
  * 즉 state.myStateKey 접근 가능

5\) Atom은 `package activate` 과정을 완료한다.

---

> `package deactivate`

1\) main \(entry-point\) 에 정의한 deactivate 함수를, atom이 종료될때 호출 한다.

* 해당 함수에서 dispose, destroy 함수를 호출하여, resource을 release 해주어야 한다.

---

> `package editor 기여 flow`

atom의 editor기여는, editor에 적용되어야 하는 view을 구현하여 리턴하는 형태로 되어 있다. 해당 view가 이미 editor에 붙어 있는 경우 view의 state을 변경하는 형태를 제공한다. 따라서, editor에 view을 제공하는 구현을 할때, serialize, deserializer 함수를 구현해 주어야 한다.

**activate 에서 atom의 addOpener 함수 등록**

* atom은 addOpener에 등록된 **모든 fucntion을 순차적으로 call** 한다.
  * 모든 function이 call되기 때문에 반드시 URI을 비교해서, 자신\(자기 package\(plugin\)\)에서 처리해야하는 부분을 if문으로 만들어 주어야 한다.

```js
    this.subscriptions.add(
        atom.workspace.addOpener(uri => {
            // 아래와 같이 if문에서 uri 비교를 반드시 해 주어야 한다.
            // 만약 그냥 view을 만들어서 return 하면, 다른 package나 default package에서 컨트롤해야 하는 요청에 대하여
            // 내가 만드는 package가 처리해 버리는 경우가 발생 할 수 있다.
            if (uri === 'atom://active-editor-info') {
                return new ActiveEditorInfoView();
            }
        })
    );
```

**atom에서 editor 열기**

* atom에서 editor을 열기
  * unique한 URI을 명시적으로 주어서 editor을 열어야 한다.
  * atom은 내부적으로 addOpener을 호출한다.
    * addOpener에 등록된 함수중 null이 아닌 view을 리턴하면 해당 view을 editor로 보여주는 형태를 가진다.

```js

// editor을 open하는 함수
atom.workspace.open('atom://active-editor-info');

// editor을 toggle하는 함수
atom.workspace.toggle('atom://active-editor-info');
```

**view 구현**

* deserializer에 package.json에서 표현되는, deserializer함수 key을 정의하고, package.json에서는 해당 key와 mapping된 main\(entry point\)에 정의하는 함수를 정의하여야 함.

```js
// package.json에 deserializers가 명시되어야 함.
"deserializers": {
    "active-editor-info/ActiveEditorInfoView": "deserializeActiveEditorInfoView"
}

// main entry 파일에 deserializers에 대한 구현이 있어야 함.
deserializeActiveEditorInfoView(serialized) {
    return new ActiveEditorInfoView();
}

// main entry 파일의 activate에 아래와 같이 addOpner가 있어야 함.
    // Add an opener for our view.
    this.subscriptions.add(
        atom.workspace.addOpener(uri => {
            if (uri === 'atom://active-editor-info') {
                return new ActiveEditorInfoView();
            }
        })
    );
    
// view 구현체의 serialize에 deseializer을 명시 해야 함.
  serialize() {
    return {
      deserializer: 'active-editor-info/ActiveEditorInfoView'
    };
  }
```

아래 코드는 view 구현 예제 코드이다.

```js
'use babel';

export default class ActiveEditorInfoView {

  constructor(serializedState) {
    // Create root element
    this.element = document.createElement('div');
    this.element.classList.add('active-editor-info');

    // Create message element
    const message = document.createElement('div');
    message.textContent = 'The ActiveEditorInfo package is Alive! It\'s ALIVE!';
    message.classList.add('message');
    this.element.appendChild(message);

    this.subscriptions = atom.workspace.getCenter().observeActivePaneItem(item => {
      if (!atom.workspace.isTextEditor(item)) return;
      message.innerHTML = `
        <h2>${item.getFileName() || 'untitled'}</h2>
        <ul>
          <li><b>Soft Wrap:</b> ${item.softWrapped}</li>
          <li><b>Tab Length:</b> ${item.getTabLength()}</li>
          <li><b>Encoding:</b> ${item.getEncoding()}</li>
          <li><b>Line Count:</b> ${item.getLineCount()}</li>
        </ul>
      `;
    });
  }

  getTitle() {
    // Used by Atom for tab text
    return 'Active Editor Info';
  }

  getDefaultLocation() {
    // This location will be used if the user hasn't overridden it by dragging the item elsewhere.
    // Valid values are "left", "right", "bottom", and "center" (the default).
    return 'right';
  }

  getAllowedLocations() {
    // The locations into which the item can be moved.
    return ['left', 'right', 'bottom'];
  }

  getURI() {
    // Used by Atom to identify the view when toggling.
    return 'atom://active-editor-info'
  }

  // Returns an object that can be retrieved when package is activated
  serialize() {
    return {
      deserializer: 'active-editor-info/ActiveEditorInfoView'
    };
  }

  // Tear down any state and detach
  destroy() {
    this.element.remove();
    this.subscriptions.dispose();
  }

  getElement() {
    return this.element;
  }

}

```

* about이나 welcome이 editor에 표시되는 것 역시 atom의 core code에서 about와 welcome에 대한 addOpener가 등록되어 있고, unique한 URI로 이 값이 정의되어 있다.



