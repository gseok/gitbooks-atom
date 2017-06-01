#### ubuntu 에서 32bit, 64bit 확인 방법

ubuntu 사용중 자기가 사용하고 있는 os architecture 확인 방법

ubunut가 32bit인지 64bit인지 확인해야 하는경우가 가끔 있다. 이를 확인 하는 방법을 간단히 소개한다.



##### GUI로 확인

ubunut에서 info 로 찾아보면 \(영문: system info, 한글: 자세히 보기\)

![](/assets/ubuntu-version-check-01.png)

현재 자신이 사용하고 있는 ubuntu에 대한 정보\(system info\)를 볼 수 있다.![](/assets/ubuntu-version-check-02.png)

##### 터미널에서 확인

```bash
$ uname -a

or

$ uname -i
```

![](/assets/ubuntu-version-check-03.png)

x86\_64는 64bit, i386은 32bit이다.



참고: [https://askubuntu.com/questions/41332/how-do-i-check-if-i-have-a-32-bit-or-a-64-bit-os](https://askubuntu.com/questions/41332/how-do-i-check-if-i-have-a-32-bit-or-a-64-bit-os)



