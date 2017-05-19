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
* keymaps
* lib
* menus
* spec
* snippets
* styles
* index.coffee 
* package.json



