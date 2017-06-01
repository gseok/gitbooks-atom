### atom build 과정 internal 하게 살펴보기

Atom Build는 atom소스의 `~/atom/script/build`을 수행하면서 부터 시작된다. 이때 실제 internal하게 build가 진행되는 과정을 분석해 보았다. atom의 `build`는 node app으로 구성되어 있다. 따라서 node debug을 통해서 build의 과정을 line by line으로 따라가면서 살펴보는게 가능하다. 이 분석은 `build`의 source을 따라가면서 어떤 동작들을 하는지 살펴보는 형태로 구성하였다.

---

#### require\('./bootstrap'\)

build 코드에서 가장 먼저 호출하는 함수이다.

* 코드
  * `script/bootstrap.js`
* 하는일

  * 1\) verifyMachineRequirements\(\) call

  * 2\) cleanDependencies\(\) call

  * 3\) installScriptDependencies\(\) call

  * 4\) installApm\(\) call

  * 5\) runApmInstall\(\) call

##### 1\) verifyMachineRequirements\(\) call

* 코드

  * `script/lib/verify-machine-requirements.js`

* 하는일

  * node, npm, python 버전을 check 한다.

##### 2\) cleanDependencies\(\) call

* 코드

  * `script/lib/clean-dependencies.js`

* 하는일

  * 이전에 build한 파일을 remove한다.
  * fs.removeSync 함수를 사용해서 remove하고 있다.
  * _~/atom/node\_modules, ~/atom/apm/node\_modules, ~/atom/script/node\_modules 을 제거한다._

##### 3\) installScriptDependencies\(\) call

* 코드

  * `script/lib/install-script-dependencies.js`

* 하는일

  * npm install 을 `~/atom/script/` 위치에서 수행한다.
  * `~/atom/script` 폴더에 정의된 `package.json`을 이용해서 node\_module을 설치 한다.
    * build script용 node\_module설치로 보면 된다.

##### 4\) installApm\(\) call

* 코드

  * `script/lib/install-apm.js`

* 하는일

  * npm install 을 `~/atom/apm/` 위치에서 수행한다.
  * apm 모듈을 설치하는 것으로 보면 된다.
    * 설치 이후 `apm` 명령 수행이 가능해 진다.
    * `~/atom/apm/node_modules/atom-package-manager/bin/apm` 이 사용가능하다.

##### 5\) runApmInstall\(\) call

* 코드

  * `script/lib/run-apm-install.js`

* 하는일

  * `apm install` 명령어를 `~/atom` 위치에서 수행한다.
  * `atom` 폴더에 정의된 `package.json`을 이용해서 `node_module`을 설치한다.
    * 실제 atom package들과 atom에서 필요로 하는 package가 설치된다.
    * `package.json`에 dependencies로 정의된 부분은 npm install과 동일하게 설치한다
    * `package.json`에 packageDependencies로 정의된 부분은\(apm 모듈의 src/install.js 참고\) apm이 npm처럼 설치한다.
  * `~/atom/apm/node_modules/atom-package-manager` 의 `install` 동작
  * ```
    1) https://atom.io/api/packages/<<package_name>> 으로 package info(JSON)을 가져옴

    2) 가져온 정보에 있는 download URL & version 정보를 사용

    3) requesetPackage, downloadPackage 함수를 이용해서 git or web에서 실제 package을 가져옴

           위 함수들은 apm의 소스 코드중 install.js을 참고

           실제 node의 request을 이용해서 파일을 가져오고 있음을 볼 수 있다.
    ```

* 결론적으로, `apm install` 이 수행되고 나면 **atom에서 필요로하는 모듈을 build전에 다 local로 가져오게 된다**.

여기까지 진행하면 build을 위한 사전작업\(bootstrap\)이 완료

---

#### checkChromedriverVersion\(\)

atom은 electron기반으로 동작하는데,  electron의 major, minor 버전과, chrom driver 버전과 잘 매칭되는지 확인하는 과정

* 코드
  * `script/lib/check-chromedriver-version.js`
* 하는일
  * `semver` 모듈을 사용해서, version을 체크한다.
  * `semver`: [http://semver.org/](http://semver.org/)
  * `~/atom/package.json` 에 정의되어 있는 `electorn-chromedriver` 와 `electron-mksnapshot` 의 버전 정보를 가지고,  동일한파일의 \(`~/atom/script/config.js` 에서 `appMetaData.electornVersion`을 가져오는데, `appMetaData`는 `~/atom/package.json`임\), 동일 파일의 `electron version`과 비교해서, 사용가능한지 체크한다.

---

#### cleanOutputDirectory\(\)

빌드 과정이기 때문에 본격적인 build가 되기전에 이전 build 결과를 제거하는 동작을 한다.

* 코드
  * `script/lib/clean-output-directory.js`
* 하는일
  * `fs.removeSync`을 이용해서 이전 build output directory을 제거한다.
  * build output directory는 `~/atom/script/config.js` 에 정의되어 있는 `buildOutputPath`을 기준으로 한다.

---

#### copyAssets\(\)

static한 resource을 빌드 output 위치로 복사하는 동작을 한다.

* 코드
  * `script/lib/copy-assets.js`
* 하는일
  * fs.copySync 로 파일을 복사한다.
  * 기본적으로 복사해야 하는 폴더가 위 코드에 하드코딩되어 있다.
    * benchmarks, dot-atom, exports, node\_modules, package.json, static, src, vendor
    * app-icons, atom.png
  * 해당 리소스들이 복사되어지는 destination위치는 `~/atom/out/Atom x64/resources/app` 이다.

---

#### transpilePackagesWithCustomTranspilerPaths\(\)

package중 custom transpiler을 지정한 package의 경우 해당 traanspiler로 소스를 `transpile` 하는 동작을 한다.

* 코드
  * `script/lib/transpile-packages-with-custom-transpiler-paths.js`
* 하는일
  * 1\) atom root에 있는 `package.json`을 읽어온다. \(`~/atom/package.json`\)
  * 2\) `package.json`의 내용중 `packageDependencies` 에 정의된 모듈 list을 가져온다.
  * 3\) `packageDependencies`에 정의된 모듈의 `package.json`을 가져온다.
    * e.g\) `~/atom/package.json`의 `packageDependencies`정의에 welcome 이라는 모듈이정의되어 있다면
    * `~/atom/node_modules/welcome/package.json`을 읽어온다.
  * 4\) 모듈의 `package.json`에서 `atomTranspilers` 이 정의되어 있는지 판단한다.
    * e.g\) `~/atom/node_modules/welcome/package.json`에 `atomTranspilers`가 정의되어 있는지 판다.
  * 5\) `atomTranspileres`가 정의되어 있으면, `CompileCache.addPathToCache()` 함수를 이용해서 `transpile`을 수행한다.
    * e.g\) [https://github.com/atom/github/blob/master/package.json](https://github.com/atom/github/blob/master/package.json)
  * 6\) `CompileCache.addPathToCache()` 함수는 `transpile`된 소스 코드를 리턴한다.
  * 7\) 리턴된 소스코드를 `fs.writeFileSync` 함수로 원본 코드를 다시 `rewrite` 한다.

결론적으로, build시점에 transpile 과정이 일어나고, 각각의 node\_module의 소스는 transpile된 결과로 소스가 저장된다.

##### CompileCache.addPathToCache\(\) call

* 코드

  * `script/lib/compile-cache.js`

* 하는일

  * 실제 주어진 file을 compile한다.\(`transpile`한다\)
  * 매번 compile하면 느리기 때문에 `cache`로 저장하여서, 이미 저장된 `cache`가 있으면 해당 코드를 리턴하게 되어 있다.
    * 따라서, cache을 날려야, 만약 해당 코드를 수정했을때 재컴파일된 내용으로 적용된다.
    * compile 캐쉬는, `C:\Users\사용자\.atom\compile-cache` 에 저장함. \(window의 경우\)
  * 여기서 말하는 compile은, `typescript, coffee, javascript` 파일에 대해서만 동작한다.
  * 해당 파일을 `babel`로 `es6`문법으로 변경한다.

---

#### transpileBabelPaths\(\)

* 코드
  * `script/lib/transpile-babel-paths.js`
* 하는일
  * Transpiling Babel paths in `~\out\app`
  * 즉 `out/app/**` 에 존재하는 `javascript`파일을 모두 array에 path값을 넣어서 저장하고 이를 babel로 `transpile`한다.
  * `transpile` 해야하는 list을 `CompileCache.addPathToCache()`을 호출하여서 `transpile`을 수행한다.

결론적으로, atom package node\_module뿐 아니라 atom source에서 build시 포함되어지는 모든 소스에 대해서, transpile이 한번 수행된다. 위의 custom transpile와 다른점은, atom에서 지정하고 있는 default transpiler을 수행한다는 점이다. default transpiler는 babel이고 es6로 transpile을 한다.

##### CompileCache.addPathToCache\(\) call

* compileCache.addPathToCache\(\) 처음 설명과 동일

---

#### transpileCoffeeScriptPaths\(\)

* 코드
  * `script/lib/transpile-coffee-script-paths.js`
* 하는일
  * Transpiling CoffeeScript paths in `~\atom\out\app`
  * `getPathsToTranspile()` 을 호출해서 `~\atom\out\app`에 있는  `.conffee`파일 list을 가져온다.
  * `CompileCache.addPathToCache()`을 호출한다.

##### CompileCache.addPathToCache\(\) call

* compileCache.addPathToCache\(\) 처음 설명과 동일
* 단 coffee의 경우 coffee script compile과정을 거치게 된다.

---

#### transpileCsonPaths\(\)

* 코드
  * `script/lib/transpile-cson-paths.js`
* 하는일
  * 동일패턴 \(Transpiling CSON paths in `~\atom\out\app`\)
  * `getPathsToTranspile()`을 호출해서 `~\atom\out\app`에 있는 `.cson`파일 list을 가져온다.
  * `CompileCache.addPathToCache()`을 호출한다.

##### CompileCache.addPathToCache\(\) call

* compileCache.addPathToCache\(\) 처음 설명과 동일
* 단 `cson`의 경우 `cson to json` 과정을 거치게 된다.

---

#### transpilePegJsPaths\(\)

* 코드
  * `script/lib/transpile-peg-js-paths.js`
* 하는일

  * 동일패턴 \(Transpiling PEG.js paths in `~\atom\out\app`\)
  * 여기서는 CompileCache.addPathToCache 가 없음

  * `PEG.js`로 `.pegjs`파일 빌드하고, 이를 `.js`파일로 만듬, 만들어지는 `.js`파일은 `PEG.js parser`코드가 됨

문법 체크 가능한 코드 생성하는게 목적인 것으로 보입니다.

##### PEG.js

* [https://pegjs.org/](https://pegjs.org/)
* javascript로 특정 문법에 대한 parser을 만들어준다. 즉 문법을 정의하고, 정의한 문법에 맞는지 확인 가능하다.
* 직접 보는게 이해가 빠르다. [https://pegjs.org/online](https://pegjs.org/online) 에 접속해서 구경하라.

---

#### generateModuleCache\(\)

* 코드
  * `script/lib/generate-module-cache.js`
* 하는일
  * Generating module cache for `~\atom\out\app`
  * atom root에 정의된 `package.json`에서, `packageDependencise` list을 가져와서, `packageName`을 하나씩 얻어옴
  * 각각 하나씩 `ModuleCache.create()`을 호출, `packageName`을 param으로 던짐
    * e.g\) `~\atom\out\app\node_modules\atom-dark-syntax`
  * `~/atom/out/app` 위치에 `package.json`생성.
  * `package.json`에 `_atomModuleCache` 의 하위 `folders` 로 path정보를 붙임
    * `exprots, spec, src, src/main-process, static, vendor` \(고정\)
  * 만들어진 새로운 `package.json`을 build과정에서 사용중인 `CONFIG` 객체에 설정하고, 이를 다시 `package.json`으로 write
    * `~\atom\out\app` 위치

##### ModuleCache.create\(\) call

* 코드
  * ~/atom/src/module-cache.coffee
* 하는일
  * 해당 모듈의\(이함수의 param으로 받은 `packageName`\), \`package.json파일을 읽어옴. 
    * e.g\) `~\atom\out\app\node_modules\atom-dark-syntax\package.json`
  * 해당 모듈의, `package.json`파일에 `_atomModuleCache` 라는 key에 `version, dependency, extensions, folders` 정보를 추가
  * 이후 다시 `package.json`을 write
    * e.g\) `~\atom\out\app\node_modules\atom-dark-syntax\package.json`

---

#### prebuildLessCache\(\)

* 코드
  * `script/lib/prebuild-less-cache.js`
* 하는일
  * less to css
  * `~\atom\out\app\less-compile-cache`에cache함

---

#### generateMetadata\(\)

* 코드
  * `script/lib/generate-metadata.js`
* 하는일
  * `~\atom\out\app\package.json` 파일 생성 \(`기존에 있는거 덮어 쓰고 재생성`\)
  * `package. menu, keymaps, deprecatedpackage`을 추가해서 재생성하고, file write

---

#### generateAPIDocs\(\)

* 코드
  * `script/lib/generate-api-docs.js`
* 하는일
  * Generating API docs at `~\atom\docs\output\atom-api.json` 으로 `api doc` 생성
  * `~/atom/.` 위치의 모든 coffee script와 `~/atom/src/**/*.js`위치의 모든 js파일을 이용해서 `api doc`을 생성한다.
  * `require('donna'), require('tello'), require('joanna')` 3개의 lib을 사용해서 api doc 생성
    * 참고: [https://www.npmjs.com/package/tello](https://www.npmjs.com/package/tello)
    * atom doc을 만드는 lib

---

#### dumpSymbols\(\)

* 코드
  * `script/lib/dump-symbols.js`
* 하는일
  * `minidump`라는 lib을 이용해서 `dump`작업을 한다.
  * Skipping symbol dumping because minidump is not supported on Windows \(윈도우의 경우 skip\)
  * `~/atom/out/app/node_modules/**/*.node`, 즉 `*.node`파일을 Listup 하고 해당 list파일을 하나씩 dump

##### minidump

* 참고: [https://www.npmjs.com/package/minidump](https://www.npmjs.com/package/minidump)

---

#### packageApplication\(\)

* 코드
  * `script/lib/package-application.js`
* 하는일
  * `electron package` 작업을 하는 부분
  * Running electron-packager on `~\atom\out\app`with app name `atom`
  * 1\) 내부적으로 `runPackager(option)`을 호출
    * option에는, `version, arch, name, outputdir, copyright`등을 설정하게 되어 있음
    * runPackager\(option\)은 electornPackager\(\)을 호출한다.
  * 2\) `copyNonASARResources()` 을 호출

##### electronPackager\(\) call

* 코드
  * node module이다.
  * [https://www.npmjs.com/package/electron-packager](https://www.npmjs.com/package/electron-packager)
* 하는일
  * `electronPackager`는 `electron` 기반 app을 package해서, `(.app, .exe)`와 같은 **실행파일로 만들어 주는 유틸**이다.
  * 즉 여기서는 electron기반 app을 package해서, 시작점인 `atom.exe` \(window의 경우\)을 만드는 작업
  * 해당 함수가 정상 동작하고 나면, `~/atom/out/Atom x64` 디렉토리가 생성되고, 해당 디렉토리 아래 `atom.exe`파일이 생성된다.

##### copyNonASARResources\(\) call

* 코드
  * `script/lib/package-application.js`
    * 이 함수는 packageApplication에 구현되어 있다.
* 하는일
  * 이 함수는 runPackage\(option\) 함수 호출 성공시 다음 step에서 바로 호출된다.
  * `~\atom\out\Atom x64\resources` 위치에 `non-ASAR resource`을 `copy`함.
  * 일단 APM을 copy
    * `~\atom\apm\node_modules\atom-package-manager` 을 copy해서 `~\atom\out\Atom x64\resources\app\apm`에 넣음
    * 따라서 atom이 build된 결과물에는 apm이 잘 존재하게 됨
  * \[ 'atom.cmd', 'atom.sh', 'atom.js', 'apm.cmd', 'apm.sh', 'file.ico', 'folder.ico' \] 파일도 copy함
  * LICENSE.md 파일도 만들어서 copy
    * script/lib/get-license-text.js을 이용해서 생성

`electorn`에서 `electorn-package`을 하면, `resource`가 복사됨 이때 빠진 부분을 수동으로 copy하는 역할

###### ASAR 설명

* `electron`에서 만든 `tar` 비슷한 `archive format`
  * 참고: [https://github.com/electron/asar](https://github.com/electron/asar)
  * 참고: [https://github.com/electron/electron/blob/master/docs/tutorial/application-packaging.md](https://github.com/electron/electron/blob/master/docs/tutorial/application-packaging.md)

---

#### generateStartupSnapshot\(\)

* 코드
  * `script/lib/generate-startup-snapshot.js`
* 하는일
  * `~\atom\out\startup.js` 을 만듬.
    * coreModules: new Set\(\['electron', 'atom', 'shell', 'WNdb', 'lapack', 'remote'\]\)
  * `electron-link` 모듈 사용해서 함수 콜
    * 참고: [https://github.com/atom/electron-link](https://github.com/atom/electron-link)
  * 생성한 snapshot 파일 move
    * `~\atom\out\snapshot_blob.bin 파일을`  `~\atom\out\Atom x64\snapshot_blob.bin` 이동

##### electron-link\(\) call

* 코드
  * [https://github.com/atom/electron-link](https://github.com/atom/electron-link)
  * [https://www.npmjs.com/package/electron-link](https://www.npmjs.com/package/electron-link)
* 하는일
  * _시작점부터 필요한 모든 _`module`_\(require한거\)을 모아서 취합_하는 것으로 보임
  * `snapshot_blob.bin` 파일을 생성하고, mksnapshot을 이용해서 검증\(verifying\)\(childProcess.execFileSync이용해서\)까지 함

---

#### build내 promise chain에서 generateStartupSnapshot\(\) 이후 첫번째 then

* 코드
  * `~/atom/script/build`
* 하는일
  * 1\) 인스톨러 생성
    * 인스톨러 생성 옵션이 없는 경우, 인스톨러 생성안함
  * 2\) codeSign 과정
    * codeSign 옵션에 따라 동작 or 미동작

##### 1\) 인스톨러 생성

build 옵션에서, installer생성하는 옵션을 준 경우 이 코드에서 installer을 생성한다.

* 인스톨러 생성 옵션이 있으면, `createWindowsInstaller`\(윈도우의 경우\)함수를 호출해서 installer을 생성함.
  * linux: createDebianPackage
  * max: code sign on mac만 호출하고 별도로 묶는 코드 없음...
* 인스톨러 생성시 패키징 되어야 하는 `atom`\(소스\) 위치
  * `~/atom/out/Atom x64` 이 됨

###### createWindowsInstaller \(윈도우의 경우\) call

* 코드
  * `script/lib/create-windows-installer.js`
* 하는일
  * `electorn window installer` 모듈을 사용해서 `window installer`을 생성함.
  * 참고: [https://github.com/electron/windows-installer](https://github.com/electron/windows-installer)

**커스텀 인스톨러**

```
커스텀 인스톨러를 생성하려면, 위 "script/lib/create-windows-installer.js" 코드를 수정해야 한다.

* setup.exe파일을 만들었다고 가정하면, 해당 파일의 아이콘, 해당파을 실행햇을때 progress시 화면 등을 여기서 다 설정 가능
* 참고: https://github.com/electron/windows-installer 를 살펴보면, 인스톨러 생성용 옵션들이 존재함.
```

##### 2\) codeSign옵션에 따라 codeSign과정 수행

* codeSign은 p12파일로 code를 signing하는 과정임
* build결과물 코드를 signing할 필요가 있는 경우 사용

---

#### build내 promise chain에서 generateStartupSnapshot 이후 두번째 then \(마지막 then\)임

* 코드
  * `~/atom/script/build`
* 하는일
  * 1\) build명령어에 compressArtifacts 옵션에 따른 동작
    * 옵션이 true면 compress 과정 `compressArtifacts()` 호출
  * 2\) build명령어에 install 옵션에 따른 동작
    * 옵션이 true면 install 과정 `installApplication()` 호출



##### compressArtifacts\(\) call

* 코드
  * `script/lib/compress-artifacts.js`
* 하는일
  * win, linux, mac에 따라서 **소스를 압축**하는 동작을 함
    * 'atom-mac.zip', \`atom-windows.zip\`, \`atom-${getLinuxArchiveArch\(\)}.tar.gz\`
  * 압축시 사용하는 util도 다름 \(압축시 node의 spawnSync사용\)
    * zip\(linux\), 7z.exe\(window\), tar\(mac\)
    * 따라서 window의 경우 `7z.exe`가 환경변수 설정되어 있어서 바로 실행 가능해야함.



##### installApplication\(\) call

* 코드
  * `script/lib/install-application.js`
*  하는일
  * build의 결과는 default로 `out/Atom x64` 위치에 저장됨. **이걸 install 위치에 copy해서 install하는 형태**임
  * 이미 설치되어 있으면 지우고 설치함



