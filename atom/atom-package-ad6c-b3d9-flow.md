### atom package 구동 flow

atom package\(plugin\)의 구동 flow에 대하여 간략하게 정리한다.

---

`package loading`

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

`package activate`

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



