---
title: Eclipse Jython 사용 ( + Pydev )
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Project
tags:
  - Web_Project
---

<br><br>

# Jython
* Jython이란 ?
  - Java 에서 Python을 사용할 수 있도록 해주는 라이브러리


## 01. 설치
* [Jython 설치 링크](https://www.jython.org/download)
* 자바와 파이썬이 모두 설치되었다는 가정하에 위의 링크에서 Jython을 설치한다 `(jar파일)`
  - `jython-installer-2.7.x.jar` : 나는 2.7.2버전
* jar파일을 실행한다. ( 따로 설정한 것 없이 동의후 계속 `Next` )
  - 단 jython을 설치한 폴더 경로는 기억해둘 것.

* **환경설정**
  1. 아무파일 열어서 `내 PC` 에서 우클릭 후 `속성`클릭
  2. `고급시스템설정` 클릭 후 `환경변수(N)...`클릭
  3. Main에 대한 사용자 변수(U)에서 `Path`클릭
  4. `새로만들기(N)`을 클릭한 후 jython이 설치된 경로를 입력
  5. cmd 창에서 `jython --version` 명령어 입력시 버전이 나와야 설치에 성공한 것!
    + ex) `Jython 2.7.2`

<br><br>

## 02. Spring ( Java )에서 실행하기

### 02-1. pom.xml
* 우선 pom.xml에 버전에 맞는 maven repository를 등록한다.

```xml
<!-- https://mvnrepository.com/artifact/org.python/jython-standalone -->
<dependency>
    <groupId>org.python</groupId>
    <artifactId>jython-standalone</artifactId>
    <version>2.7.2</version>
</dependency>
```

<br>

### 02-2. python파일
* test.py 파일 준비
  - python파일을 java에서 실행할때에는 반드시 encoding이 필요하다.
  - 안해주면 500에러 등장
  - `# -*- coding: utf-8 -*-` python 파일의 첫줄에 이 명령어를 입력하면 문제없이 사용할 수 있다.

```python
# -*- coding: utf-8 -*-
def testFunc(a,b):
	print("테스트 하자! Test!")
	c = a+b
	return c
```

### 02-3. java파일
* java파일과 매핑된 jsp에 요청을 보내면 python파일을 실행하는 코드가 실행된다.


#### 실행방법 1

```java

private static PythonInterpreter intPre;


@RequestMapping("/pyInfo.do")
public String pyInfo() {

  intPre = new PythonInterpreter();
  intPre.execfile("python파일이 있는 경로");
  //intPre.exec("print(testFunc(5,10))"); // testFunc(a,b) 실행
  
  intPre.exec("a = testFunc(5,10)");
  PyObject result = intPre.get("a");  //변수를 받기
  System.out.println(result.toString());
  
  
  
  return "pythonpj/pyInfo";
}

```

<br>

* 결과

```
테스트 하자! Test!
15
```

<br>

#### 실행방법 2

```java
private static PythonInterpreter intPre;


@RequestMapping("/pyInfo.do")
public String pyInfo() {

  intPre = new PythonInterpreter();
  PyFunction pyFunction = (PyFunction)intPre.get("testFunc", PyFunction.class);
  int a = 10;
  int b = 20;
  PyObject pyobj = pyFunction.__call__(new PyInteger(a), new PyInteger(b));
  System.out.println(pyobj.toString());
  return "pythonpj/pyInfo";
}
```
<br>

* 결과

```
테스트 하자! Test!
30
```

<br><br>

# PyDev
* 그런데 jython을 쓰다보면 py파일 만질때 메모장으로 나와서 상당히 불편하다.
* PyDev는 Eclipse에서 설치된 Python을 사용할 수 있도록 해주는 플러그인이다.

## 01. 설치
1. 상단 메뉴의 `Help` - `install New Software` 클릭
2. Add버튼을 누르고 아래의 정보를 입력한다.
  - Name : `PyDev`
  - Location : `http://pydev.org/updates`
  - <span style="color:red">만약 에러를 만났다면?</span>
    + dependencies are not satisfiable 어쩌구 하는 에러를 만났다면?
    + Location에 `http://update-production-pydev.s3.amazonaws.com/pydev/updayes/site.xml`를 입력한다.

3. 설치 후, Restart하라는 메시지가 뜨면 성공한 것!

<br><br>

## 02. 약간의 환경설정
* 상단의 `Windows` - `Preference` 에 들어가면 `PyDev`가 생긴 것을 볼 수 있다.
* `Python Interpreters`를 클릭
* `Quick Auto-Config`클릭하면 자신의 컴퓨터에 설치된 Python이 알아서 세팅된다.



---


<br><br><br><br>



