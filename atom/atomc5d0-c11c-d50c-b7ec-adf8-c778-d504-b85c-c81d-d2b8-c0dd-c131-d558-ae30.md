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





생성된 프로젝트를 atom에 적용한 형태로 atom을 실행하려면, atom을 새로 띄우거나, refresh을 한번 해 주어야 한다.

