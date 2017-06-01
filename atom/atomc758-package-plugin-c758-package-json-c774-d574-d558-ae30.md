### atom의 package\(plugin\)의 package.json이해하기

atom의 package\(plugin\)프로젝트는 반드시 `package.json`이 필요하다. 해당 파일은 package\(plugin\)의 metadata\(manifest\)을 기술하도록 되어 있다. 기본적으로 Node 의 npm 에서 사용하는 `package.json`의 구조를 그대로 사용하고 있다. 거기에 더하여 atom에서 사용하는 `keys`가 추가되어 있다. 하나씩 살펴보도록 한다.

> package.json 살펴보기

```
{
  "name": "my-package",
  "main": "./lib/my-package",
  "version": "0.0.0",
  "description": "A short description of your package",
  "styles": [
  ],
  "
  "keymaps": [
  ],
  "menus": [
  ],
  "snippets": [
  ],
  "activationCommands": {
    "atom-workspace": "my-package:toggle"
  },
  "activationHooks": [
    "language-javascript:grammar-used",
    "language-coffee-script:grammar-used"
  ],
  "repository": "https://github.com/atom/my-package",
  "license": "MIT",
  "engines": {
    "atom": ">=1.0.0 <2.0.0"
  },
  "dependencies": {
  }
}
```

* name
  * package name을 기술한다. npm에서 사용하는것과 동일하다.
* main
  * package의 entry point, 기술되어 있지 않으면 root의 index.coffee나 index.js을 사용한다.
* version
  * package version을 기술한다. npm에서 사용하는것과 동일하다.
* description
  * package desc을 기술한다. npm에서 사용하는 것과 동일하다.
* style
  * package에서 필요로 하는 스타일 정의.
  * 배열안의 요소는 string으로 정의한다.
  * 만약 정의하지 않으면, `styles` 디렉토리에 정의한 cson파일이 알파벳 순서로 등록된다.

* keymaps
  * package에서 필요로 하는 key정의.
  * 배열안의 요소는 string으로 정의한다.
  * 만약 정의하지 않으면, `keymaps` 디렉토리에 정의한 cson파일이 알파벳 순서로 등록된다.
* menus
  * package에서 기여하는 메뉴를 정의한다.
  * 배열안에 요소는 string으로 정의한다.
  * 만약 정의하지 않으면, `menus` 디렉토리에 정의한 cson파일이 알파벳 순서로 등록된다.
* snippets
  * package에서 기여하는 snippets을 정의한다.
  * 배열안의 요소는 string으로 정의한다.
  * 만약 정의하지 않으면, `snippets` 디렉토리에 정의한 cson파일이 알파벳 순서로 등록된다.

* activationCommands
  * package가 action되는 trigger인 commands을 정의한다.
  * 배열안의 요소는 string으로 정의한다.
  * 배열안의 요소 값은 command 이다.
  * activationCommands을 정의하지 않으면, package가 load될때 entry point\(main\)에서 export 하는 `activate()` 함수가 호울 된다.
  * **주의: package load와 package active는 서로 다르다.**
    * load: 특정 폴더\(window의 경우, ~/사용자/.atom/package/\) 아래 존재하는 폴더들은 모두 package\(plugin\)으로 판단하고, package.json 을 읽어서, package\(plugin\)을 atom이 알게 한다. load 타임에, atom의 메뉴, short cut등은 바로 기여됨
    * active: 실제 command을 수행 하거나, 연관된 파일을 open하는 등의 동작이 있을때, 해당 동작에 따라 package가 어떤 일을 수행해야 하는 경우, 해당 package의 entry point을 호출하고, package의 동작을 수행한다.
* activationHooks
  * package의 activation을 dependency을 주어서, 어떤 동작이 다 된 다음에 package가 activation되도록 하고 싶을때 사용할 수 있는 방법이다.
  * 즉 해당 hooks가 trigger될때 까지 package의 load가 지연된다. 
  * 예를 들어 어떤 package가 `language-javascript:grammar-used` 을 hook 으로 기술 하고 있다면, 해당 package는 `language-javascript:grammar-used`라는 command가 실행되기 전까지는 load되지 않는다. 즉 해당 package는 `language-javascript:grammar-used`가 실행 된 이후에 실행 가능한 package라는 소리이다.



