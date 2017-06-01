### atom에서 플러그인 프로젝트 생성하기

Atom은 atom의 package\(plugin\)을 개발하기 위한 package-generator가 내장되어 있다. 따라서 해당 command을 사용하면 atom의 package\(plugin\)프로젝트가 바로 생성되고, 생성과 동시에 해당 프로젝트가 atom의 package\(plugin\)설치 위치에 자동으로 설치된다.

> atom package\(plugin\) project 생성

다음 과정을 통해 손쉽게 atom package\(plugin\) project을 생성 할 수 있다.

* 커멘드 팔렛트 열기: Ctrl + Shift + P 
* package-generator: generator package 커맨드 실행
  * ![](/assets/atom-gen-package-1.png)
* package name 및 package 생성 위치 설정
  * ![](/assets/atom-gen-package-2.png)
* 생성 완료되면, 해당 프로젝트로 atom이 새로 뜬다.
  * ![](/assets/atom-gen-package-3.png)

> atom package\(plugin\) 적용

프로젝트 생성 위치는 아무 위치에나 해도 상관 없지만, 생성한 package\(plugin\) 프로젝트를 atom에 적용하려면, 특정 위치에 해당 프로젝트를 넣어 주어야 한다. 프로젝트 폴더를 복사해서 넣어도 되지만, **링크로 넣어도 된다**.

Window의 경우 아래와 같은 폴더에 atom package\(plugin\)코드를 넣어야 한다.

* C:\Users\&lt;your-account&gt;.atom\packages

생성된 프로젝트를 atom에 적용한 형태로 atom을 실행하려면, atom을 새로 띄우거나, refresh을 한번 해 주어야 한다.

atom에서는 자기자신을 다시 refresh하는 단축키을 제공하고 있다.

* Ctrl + Shift + F5

위와 같이, 특정 폴더에 atom package\(plugin\)을 넣고, atom을 한번 refresh하면, 해당 package\(plugin\)이 적용된 atom이 실행 된다.

> atom package\(plugin\) 구조

```
my-package/
├─ grammars/
├─ keymaps/
├─ lib/
├─ menus/
├─ spec/
├─ snippets/
├─ styles/
├─ index.coffee
└─ package.json
```

* grammars
  * 해당 package\(plugin\) 어떤 언어을 정의하는 package일 경우 구현해야 하는 디렉토리 이고, 이 아래 g
  * 어떤 파일을 open할때 해당 파일이 어떤 언어\(language - e.g.&gt; java, javascript, markdown, etc...\)인제 판단한다.
  * 보통 파일 확장자로 판단한다.
  * grammars자체는 \([http://manual.macromates.com/en/language\_grammars](http://manual.macromates.com/en/language_grammars)\) 형태로 만들어야 한다.
* keymaps
  * 단축키을 설정하기 위한 폴더이다.
  * 해당 폴더 아래 cson 형태로 단축키를 정의하면 된다.
  * 단축키 정의시, 단축키가 눌리면 호출되어야 하는 부분은 command이름을 쓰게 되어 있다.
* lib
  * 실제 package의 구현체 파일들을 넣는 폴더이다.
  * 이름이 lib이지만 일반적인 project의 src폴더로 생각하면 된다.
* menus
  * 메뉴를 설정하기 위한 폴더이다.
  * 해당 폴더 아래 cson 형태로 메뉴를 정의하면 된다.
* spec
  * unit test코드를 넣기 위한 폴더이다. \(어떤 스펙정의를 하라는 폴더 아님!\)
  * atom은 기본적으로 Jasmin v1.3을 사용해서 unit test을 하고 있다.
  * 따라서 해당 폴더에 구현하는 pacakge에 대한 Jasmin 문법의 Unit Test 코드를 작성하고, atom에서 곧바로 이를 테스트 해 볼 수 있다.
    * _Alt + Ctrl + P \| View &gt; Developer &gt; Run Package Specs_
* snippets
  * 코드 snippet기능을 제공하기 위한 폴더이다.
  * 해당 폴더 아래 cson 형태로 snippet을 구현하면 된다.
  * atom에서 snippet은 `key` 을 쓰고, `tab`을 누르면 해당 snippet이 자동으로 editor에 추가되는 형태를 취하고 있다.
* styles
  * package\(plugin\)의 스타일을 설정 하기 위한 폴더이다.
  * package가 activated될 때 해당 폴더 아래 있는 스타일 파일 \(css \| less\)이 로딩되어 적용된다. \(css 추천\)
* index.coffee \| index.js
  * 해당 package\(plugin\)의 default start\(entry\) point\(main\)이다.
  * 만약 package.json에 main항목이 정의되어 있지 않으면 해당 index.coffee \| index.js가 호출되고, 만약 package.json에 main 항목이 정의되어 있으면 해당 항목에 정의된 파일을 호출한다.
* package.json
  * package\(plugin\)에 대한 metadata\(manifest\)을 정의하는 파일 \(필수\) 이다.
  * 해당 package\(plugin\)의 start point\(main\)등을 정의하게 되어 있다.



