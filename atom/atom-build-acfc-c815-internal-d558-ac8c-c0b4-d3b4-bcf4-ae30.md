### atom build 과정 internal 하게 살펴보기

Atom Build는 atom소스의 script/build을 수행하면서 부터 시작된다. 이때 실제 internal하게 build가 진행되는 과정을 분석해 보았다.



-----------------------------------------------------------------------------------------------------------------

&gt; require\('./bootstrap'\)

 \* script/bootstrap.js

 \* 1\) verifyMachineRequirements\(\) call

  \*\* script/lib/verify-machine-requirements.js

  \*\* node, npm, python 버전을 check 한다.

 \* 2\) cleanDependencies\(\) call

  \*\* script/lib/clean-dependencies.js

  \*\* 이전에 build한 파일을 remove한다.

  \*\* fs.removeSync

 \* 3\) installScriptDependencies\(\) call

  \*\* script/lib/install-script-dependencies.js

  \*\* npm install 을 "~/atom/sciprt/" 위치에서 수행한다.

  \*\* atom/script 폴더에 정의된 package.json을 이용해서 node\_module을 설치 ==&gt; build script용 node\_module설치로 보면 된다.

 \* 4\) installApm\(\) call

  \*\* script/lib/install-apm.js

  \*\* npm install 을 "~/atom/apm/" 위치에서 수행한다.

  \*\* apm 모듈을 설치하는 것으로 보면 된다.

  \*\* 설치 이후 "apm" 명령 수행이 가능해 진다.

 \* 5\) runApmInstall\(\) call

  \*\* script/lib/run-apm-install.js

  \*\* apm install 명령어를 "~/atom" 위치에서 수행한다.

  \*\* atom 폴더에 정의된 packagd.json을 이용해서 node\_moudle을 설치한다. ==&gt; 실체 atom package들과 atom에서 필요로 하는 package가 설치된다.

  \*\* package.json에 dependencies로 정의된 부분은 npm install과 동일하게 설치한다.

  \*\* package.json에 packageDependencies로 정의된 부분은\(apm 모듈의 src/install.js 참고\) apm이 npm처럼 설치한다.

    \*\*\* 1\) https://atom.io/api/packages/&lt;&lt;package\_name&gt;&gt; 으로 package info\(JSON\)을 가져옴

    \*\*\* 2\) 가져온 정보에 있는 download URL & version 정보를 사용

    \*\*\* 3\) requesetPackage, downloadPackage 함수를 이용해서 git or web에서 실제 package을 가져옴

           위 함수들은 apm의 소스 코드중 install.js을 참고

           실제 node의 request을 이용해서 파일을 가져오고 있음을 볼 수 있다.

  \*\* 결론적으로, \(apm install\) 이 수행되고 나면 atom에서 필요로하는 모듈을 build전에 다 local로 가져오게 된다.



여기까지 진행하면 build을 위한 사전작업\(bootstrap\)이 완료







-----------------------------------------------------------------------------------------------------------------

&gt; checkChromedriverVersion\(\)







-----------------------------------------------------------------------------------------------------------------

&gt; cleanOutputDirectory\(\)







-----------------------------------------------------------------------------------------------------------------

&gt; copyAssets\(\)







-----------------------------------------------------------------------------------------------------------------

&gt; transpilePackagesWithCustomTranspilerPaths\(\)

 \* script/lib/transpile-packages-with-custom-transpiler-paths.js



&gt;&gt; 함수\(transpilePackagesWithCustomTranspilerPaths\) call시 step

 \* atom root에 있는 package.json에 packageDependencies 에 정의된 모듈만 확인

 \* packageDependencies에 정의된 모듈의 package.json을 다시 확인

 \* packageDependencies에 정의된 모듈의 package.json의 내용중 atomTranspilers 가 정의된 파일에 대해서만 동작

  \*\* e.g.\) https://github.com/atom/github/blob/master/package.json

 \* packageDependencies에 정의된 모듈의 package.json의 내용중 atomTranspilers 가 정의된 파일을 이용해서 transpiler 해야 하는 path\(source\)을 모

두 array에 저장

 \* 이후 CompileCache.addPathToCache\(\) 함수 호출

 \* CompileCache.addPathToCache\(\) 함수는 transpile된 소스 코드를 리턴

 \* 리턴된 코드를 기존 file에 fs.writeFileSync을 통해 rewrite!

 \* 즉 build시 out/app/node\_module/&lt;&lt;package dependency 모듈중 atomTranspilers을 정의한 모듈&gt;&gt;/lib/ 소스코드는,

   transpile된 코드로 변경되어서 저장됨



&gt;&gt;&gt;&gt; 함수\(CompileCache.addPathToCache\) call시 step

 \* script/lib/compile-cache.js

 \* 실제 주어진 file을 compile함

 \* 매번 compile하면 느리기 때문에 cache로 저장하여서, 이미 저장된 cache가 있으면 해당 코드를 리턴하게 되어 있음

  \*\* 따라서, cache을 날려야, 만약 해당 코드를 수정했을때 재컴파일된 내용으로 적용된다.

  \*\* out은 지우지만 컴파일 캐쉬는, "C:\Users\사용자\.atom\compile-cache" 에 저장함.

 \* 여기서 말하는 compile은, ts, coffee, js 파일에 대해서만 되고.

 \* 해당 파일을 babel로 es6문법으로 변경한다.







-----------------------------------------------------------------------------------------------------------------

&gt; transpileBabelPaths\(\)

 \* script/lib/transpile-babel-paths.js

 \* Transpiling Babel paths in ~\out\app

 \* 즉 out/app/\*\* 에 존재하는 js파일을 모두 array에 path값을 넣어서 저장하고 이를 babel로 transpile한다.

 \* CompileCache.addPathToCache\(\) 함수 호출을 해서 리턴된 코드를 기존 file에 fs.writeFileSync을 통해 rewrite!



&gt;&gt;&gt;&gt; 함수\(CompileCache.addPathToCache\) call시 step

 \* 위의 transpilePackagesWithCustomTranspilerPaths에서의 함수\(CompileCache.addPathToCache\) call시 step과 동일함







-----------------------------------------------------------------------------------------------------------------

&gt; transpileCoffeeScriptPaths\(\)

 \* script/lib/transpile-coffee-script-paths.js

 \* Transpiling CoffeeScript paths in ~\atom\out\app

 \* getPathsToTranspile\(\)을 호출해서 "~\atom\out\app"에 있는  ".conffee"파일 list을 가져온다.

 \* CompileCache.addPathToCache\(\)을 호출한다.



&gt;&gt;&gt;&gt; 함수\(CompileCache.addPathToCache\) call시 step

 \* 위의 transpilePackagesWithCustomTranspilerPaths에서의 함수\(CompileCache.addPathToCache\) call시 step과 동일함

 \* 단 coffee의 경우 coffee script compile과정을 거치게 된다.







-----------------------------------------------------------------------------------------------------------------

&gt; transpileCsonPaths\(\)

 \* script/lib/transpile-cson-paths.js

 \* 동일패턴 \(Transpiling CSON paths in "~\atom\out\app"\)

 \* getPathsToTranspile\(\)을 호출해서 "~\atom\out\app"에 있는 ".cson"파일 list을 가져온다.

 \* CompileCache.addPathToCache\(\)을 호출한다.



&gt;&gt;&gt;&gt; 함수\(CompileCache.addPathToCache\) call시 step

 \* 위의 transpilePackagesWithCustomTranspilerPaths에서의 함수\(CompileCache.addPathToCache\) call시 step과 동일함

 \* 단 cson의 경우 cson to json 과정을 거치게 된다.







-----------------------------------------------------------------------------------------------------------------

&gt; transpilePegJsPaths\(\)

 \* script/lib/transpile-peg-js-paths.js

 \* 동일패턴 \(Transpiling PEG.js paths in "~\atom\out\app"\)

 \* PEG.js ==&gt; https://pegjs.org/

  \*\* javascript로 특정 문법에 대한 parser을 만들어준다. ==&gt; 문법을 정의하고, 정의한 문법에 맞는지 확인 가능하다.

  \*\* https://pegjs.org/online

 \* 여기서는 CompileCache.addPathToCache 가 없음

 \* PEG.js로 ".pegjs"파일 빌드하고, 이를 ".js"파일로 만듬, 만들어지는 ".js"파일은 PEG.js parser코드가 됨

  \*\* 즉 문법 체크 가능한 코드 만들어서 이른 .js로 저장







-----------------------------------------------------------------------------------------------------------------

&gt; generateModuleCache\(\)

 \* script/lib/generate-module-cache.js

 \* Generating module cache for "~\atom\out\app"

 \* atom root에 정의된 package.json에서, packageDependencise list을 가져와서, packageName을 하나씩 얻어옴

  \*\* 각각 하나씩 ModuleCache.create\(\)을 호출

  \*\* e.g\) "~\atom\out\app\node\_modules\atom-dark-syntax"

 \* "~\atom\out\app" 위치에도 package.json생성.

 \* \(package.json\)에 "\_atomModuleCache" 의 하위 "folders" 로 path정보를 붙임

  \*\* '', exprots, spec, src, src/main-process, static, vendor \(고정\)

 \* 만들어진 새로운 package.json을 build과정에서 사용중인 CONFIG 객체에 설정하고, 이를 다시 package.json으로 write

  \*\* "~\atom\out\app" 위치



&gt;&gt;&gt;&gt; 함수\(ModuleCache.create\) call시 step

 \* 해당 모듈의, package.json파일을 읽어옴. \("~\atom\out\app\node\_modules\atom-dark-syntax\package.json"\)

 \* 해당 모듈의, package.json파일에 \_atomModuleCache 라는 key에 version, dependency, extensions, folders 정보를 추가

 \* 이후 다시 package.json을 write \("~\atom\out\app\node\_modules\atom-dark-syntax\package.json"\)







-----------------------------------------------------------------------------------------------------------------

&gt; prebuildLessCache\(\)

 \* script/lib/prebuild-less-cache.js

 \* less to css

 \* "~\atom\out\app\less-compile-cache"에cache함







-----------------------------------------------------------------------------------------------------------------

&gt; generateMetadata\(\)

 \* script/lib/generate-metadata.js

 \* "~\atom\out\app\package.json" 파일 생성 \(기존에 있는거 덮어 쓰고 재생성\)

 \* package. menu, keymaps, deprecatedpackage을 추가해서 재생성하고, file write







-----------------------------------------------------------------------------------------------------------------

&gt; generateAPIDocs\(\)

 \* Generating API docs at "~\atom\docs\output\atom-api.json" 으로 api doc 생성

 \* "~/atom/." 위치의 모든 coffee script와 "~/atom/src/\*\*/\*.js"위치의 모든 js파일을 이용해서 api doc을 생성한다.

 \* require\('donna'\), require\('tello'\), require\('joanna'\) 3개의 lib을 사용해서 api doc 생성

  \*\* 참고: https://www.npmjs.com/package/tello

 \* atom doc을 만드는 lib







-----------------------------------------------------------------------------------------------------------------

&gt; dumpSymbols\(\)

 \* script/lib/dump-symbols.js

 \* Skipping symbol dumping because minidump is not supported on Windows \(윈도우의 경우 skip\)

 \* minidump라는 lib을 이용해서 dump작업을 한다.

  \*\*"~/atom/out/app/node\_modules/\*\*/\*.node", 즉 "\*.node"파일을 Listup 하고 해당 list파일을 하나씩 dump

 \* 참고: https://www.npmjs.com/package/minidump

 \* promise을 리턴







-----------------------------------------------------------------------------------------------------------------

&gt; packageApplication\(\)

 \* script/lib/package-application.js

 \* electron package 작업을 하는 부분

 \* Running electron-packager on "~\atom\out\app" with app name "atom"

 \* 내부적으로 runPackage\(option\)을 호출

  \*\* option에는, version, arch, name, outputdir, copyright등을 설정하게 되어 있음





&gt;&gt;&gt;&gt; 함수\(runPackage\(option\)\) call시 step

 \* electronPackager\(\) 함수 호출

 \* electronPackager는 electron 기반 app을 package해서, \(.app, .exe\)와 같은 실행파일로 만들어 주는 유틸임

 \* 즉 여기서는 electron기반 app을 package해서, 시작점인 atom.exe \(window의 경우\)을 만드는 작업

 \* 참고: https://www.npmjs.com/package/electron-packager

 \* 해당 함수가 정상 동작하고 나면, "~/atom/out/Atom x64" 디렉토리가 생성되고, 해당 디렉토리 아래 atom.exe파일이 생성되어 있음.





&gt;&gt;&gt;&gt; 함수 copyNonASARResources\(\)는 runPackage\(option\) 함수 호출 성공시 다음 step에서 바로 호출됨

 \* "~\atom\out\Atom x64\resources" 위치에 non-ASAR resource을 copy함.

 \* ASAR??

  \*\* electron에서 만든 tar 비슷한 archive format인듯함 - 참고: https://github.com/electron/asar

  \*\* 참고: https://github.com/electron/electron/blob/master/docs/tutorial/application-packaging.md

 \* electorn에서 electorn-package을 하면, resource가 복사되는데\(ASAR??되는데?\) 빠진 부분을 수동으로 copy하는 역할로 보임

 \* 일단 APM을 copy함

  \*\* "~\atom\apm\node\_modules\atom-package-manager" 을 copy해서 --&gt; "~\atom\out\Atom x64\resources\app\apm"에 넣음

  \*\* 따라서 atom이 build된 결과물에는 apm이 잘 존재하게 됨

 \* \[ 'atom.cmd', 'atom.sh', 'atom.js', 'apm.cmd', 'apm.sh', 'file.ico', 'folder.ico' \] 파일도 copy함

 \* LICENSE.md 파일도 만들어서 copy

  \*\* script/lib/get-license-text.js을 이용해서 생성







-----------------------------------------------------------------------------------------------------------------

&gt; generateStartupSnapshot\(\)

 \* script/lib/generate-startup-snapshot.js

 \* "~\atom\out\startup.js" 을 만듬.

 \* coreModules: new Set\(\['electron', 'atom', 'shell', 'WNdb', 'lapack', 'remote'\]\)

 \* electron-link 모듈 사용해서 함수 콜

 \* 참고: https://github.com/atom/electron-link

  \*\* 시작점부터 필요한 모든 module\(require한거\)을 모아서 취합하는 것으로 보임

  \*\* snapshot\_blob.bin 파일을 생성하고, mksnapshot을 이용해서 검증\(verifying\)\(childProcess.execFileSync이용해서\)까지 함

 \* 생성한 snapshot 파일 이동

  \*\*"~\atom\out\snapshot\_blob.bin 파일을" ===&gt;  "~\atom\out\Atom x64\snapshot\_blob.bin" 이동

  





-----------------------------------------------------------------------------------------------------------------

&gt; build내 promise chain에서 generateStartupSnapshot 이후 첫번째 then

 \* build 옵션에서, installer생성하는 옵션을 준 경우 이 코드에서 installer을 생성한다.

 \* 인스톨러 생성 옵션이 있으면, createWindowsInstaller\(윈도우의 경우\)함수를 호출해서 installer을 생성함.

  \*\* linux: createDebianPackage

  \*\* max: code sign on mac만 호출하고 별도로 묶는 코드 없음...

 \* 인스톨러 생성시 패키징 되어야 하는 atom은 &gt;&gt; "~/atom/out/Atom x64" 이 됨

 \* 인스톨러 생성 옵션이 없는 경우, 인스톨러 생성안함

 \* codeSign옵션에 따라 codeSign과정 수행

  \*\* codeSign은 p12파일로 code를 signing하는 과정임

  \*\* build결과물 코드를 signing할 필요가 있는 경우 사용



&gt;&gt;&gt;&gt; 함수 createWindowsInstaller \(윈도우의 경우\) call시 step

 \* script/lib/create-windows-installer

 \* electorn window installer 모듈을 사용해서 window installer을 생성함.

 \* 참고: https://github.com/electron/windows-installer

 \* 



\* 커스텀 인스톨러를 생성하려면, 위 "script/lib/create-windows-installer" 코드를 수정해야 함

 \*\* setup.exe파일을 만들었다고 가정하면, 해당 파일의 아이콘, 해당파을 실행햇을때 progress시 화면 등을 여기서 다 설정 가능

\* 참고: https://github.com/electron/windows-installer 를 살펴보면, 인스톨러 생성용 옵션들이 존재함.







-----------------------------------------------------------------------------------------------------------------

&gt; build내 promise chain에서 generateStartupSnapshot 이후 두번째 then \(마지막 then\)임

 \* 1\) build명령어에 compressArtifacts 옵션에 따른 동작

  \*\* 옵션이 true면 compress 과정

 \* 2\) build명령어에 install 옵션에 따른 동작

  \*\* 옵션이 true면 install 과정



&gt;&gt;&gt;&gt; 함수 compressArtifacts\(\) call시 step

 \* script/lib/compress-artifacts.js

 \* win, linux, mac에 따라서 소스를 압축하는 동작을 함

  \*\* 'atom-mac.zip', \`atom-windows.zip\`, \`atom-${getLinuxArchiveArch\(\)}.tar.gz\`

 \* 압축시 사용하는 util도 다름 \(압축시 node의 spawnSync사용\)

  \*\* zip, 7z.exe, tar

  \*\* 따라서 window의 경우 7z.exe가 환경변수 설정되어 있어서 바로 실행 가능해야함.





&gt;&gt;&gt;&gt; 함수 installApplication\(\) call시 step

 \* script/lib/install-application

 \* build의 결과는 default로 "out/Atom x64" 위치에 저장됨. 이걸 install 위치에 copy해서 install하는 형태임

 \* 이미 설치되어 있으면 지우고 설치함

 








