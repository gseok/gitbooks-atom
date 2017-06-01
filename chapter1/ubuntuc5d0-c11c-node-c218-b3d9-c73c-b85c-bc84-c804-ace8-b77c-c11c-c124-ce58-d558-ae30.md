### ubuntu에서 node.js 수동으로 버전 골라서 설치 하기

`ubuntu`에서 `node.js` 설치시 보통 package manager인 `apt-get`을 사용해서 설치한다.

\(`apt-get`에 `repository`을 추가한뒤, `apt-get`을 사용하여 설치\)

하지만 경우에 따라서, `node.js`을 버전에 따라서, 수동으로 설치해야 하는 경우가 있다.

수동으로 설치하는 방법 역시 어렵지 않기 때문에 간단히 설명한다.

#### ubuntu에서 node.js 수동 설치 방법

##### 1. 원하는 버전 다운로드 받기

node.js 에서는 node.js의 모든 버전별 package을 아래 주소에서 관리하고 있다.

**node.js packages: **[**http://nodejs.org/dist/**](http://nodejs.org/dist/)

해당 위치에가면, 처음 node.js 부터 현재 가장 최신의 node.js 까지 모두 다운로드 가능하다.

현재 자신의 os와 os architecture\(32bit or 64bit\)에 맞는 버전을 다운로드 받는다.

```bash
$ wget http://nodejs.org/dist/v7.9.0/node-v7.9.0-linux-x64.tar.gz
```

##### **2. 압축풀고 설치하기**

> 압축 풀기

```
$ tar -xvf node-v7.9.0-linux-x64.tar.gz
```

> 사용하기

다운로드 받은 node.js는 사실 압축을 풀고나면 그대로 사용가능하다.

```
# 압축 풀고 바로 사용하는 예제

$ /home/gseok/node-v7.9.0-linux-x64/bin/node --version
```

하지만, 보통 ubuntu에서 어느 위치에서나 사용가능하게 만들어 두고 쓰는게 일반적이다.

따라서, 압축해제한 node폴더를 `/usr/local/` 로 복사해서 사용한다.

\(또는 ln 명령어로 link을 걸어서 사용해도 된다.\)

> /usr/local/ 에 복사해서 사용하기

```bash
# 압축 해제한 디렉토리로 이동
$ cd node-v7.9.0-linux-x64

# 복사 명령 수행 (sudo)
$ sudo cp -r * /usr/local
```

> 확인 및 사용

```bash
$ node --version
```

정상적으로 설치되었다면, 아무 위치에서나, node --version 명령 수행이 잘 동작할 것이다.

위 방법으로 설치하면 npm도 같이 잘 설치된다.

#### 참고

[https://github.com/nodesource/distributions](https://github.com/nodesource/distributions)

* apt-get을 사용한 설치

```bash
# Using Ubuntu

$ curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
$ sudo apt-get install -y nodejs
```

* 수동 설치를 한번에 처리하는 명령어 \(node-v7.9.0에 64bit를 예로 들었음\)

```bash
$ wget http://nodejs.org/dist/v7.9.0/node-v7.9.0-linux-x64.tar.gz && \
    tar zxvf node-v7.9.0-linux-x64.tar.gz && \
    cd node-v7.9.0-linux-x64 && \
    cp -r * /usr/local/ && \
    cd .. && \
    rm -rf node*
```



