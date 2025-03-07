## MonoRepo with yarn Berry 구축 실습

### 1️⃣ yarn 설치 <br/>
yarn 설치
```
$ npm install -g yarn
```
<br/>

### 2️⃣ yarn berry 환경 셋팅 <br/>
yarn classic(yarn v1)과 yarn berry(yarn v2)는 버전 차이가 많이 나므로 yarn berry로 추가적인 설정 작업
```
$ yarn set version berry
```
이후
```
$ yarn -v
```

명령어를 통해 yarn 2.0 이상의 버전이 설치되었는지 확인
(본인의 경우 yarn@4.7이 설치됨)
<img width="555" alt="스크린샷 2025-03-07 오후 4 13 19" src="https://github.com/user-attachments/assets/929079cb-85a7-4064-9334-d02f1628c2e9" />

```
$ yarn init
````
알맞게 버전 설치가 완료되었다면 위 명령어로 프로젝트 설정을 마무리
cf) 위 명령어 실행 시 package.json을 비롯한 yarn 설정 파일들이 생성됨
<br/><br/>


### 3️⃣ package.json - workspace 셋팅
package.json 파일에 아래와 같이 작업할 모노레포의 name과 workspace를 추가
(name은 자유롭게 설정 가능)

```
{
  "name": "monorepo-example",
  // 이 부분의 명령어를 통해 package 내부의 모든 폴더는 workspace에 속하도록 설정됨
  "workspaces": {
    "packages": [
      "package/*"
    ]
  },
  "packageManager": "yarn@4.7.0",
}
```

위와 같이 작성을 마친 뒤, package폴더 생성
<br/><br/>

### 4️⃣ project 생성
지금부터 생성할 project 코드들은 모두 package 폴더 내부에서 관리됨
```
$ cd package
```
위 명령어를 통해 package 폴더로 이동,<br/>
CRA 또는 vite로 프로젝트를 생성
```
create-react-app
> $ npx create-react-app front-mono

vite
> $ yarn create vite
```

cf) 빌드 속도가 빠르고 번들러 사이즈가 작은 vite를 선택함<br/>
(vite 설정 : front-mono(name) → react → Typescript 설정)
<br/><br/>

다음으로 package 폴더 내부에 common 폴더를 생성
생성이 완료되었다면 현재 폴더 구조가 다음과 같은지 확인한 뒤 5️⃣를 진행

📦MONOREPO-EXAMPLE <br/>
 ┣ 📂.git <br/>
 ┣ 📂.yarn <br/>
 ┣ 📂package <br/>
 ┃ ┣ 📂common <br/>
 ┃ ┗ 📂front-mono <br/>
 ┣ 📜.editorconfig <br/>
 ┣ 📜.gitignore <br/>
 ┣ 📜.yarnrc.yml <br/>
 ┣ 📜README.md <br/>
 ┣ 📜package.json <br/>
 ┗ 📜yarn.lock <br/> <br/> 

 
### 5️⃣ front-mono 프로젝트 서버 구동 확인
4️⃣에서 생성한 프로젝트 파일이 정상적으로 실행되는지 확인<br/>
생성한 폴더로 이동해 yarn(yarn install)과 yarn dev(vite 개발 서버 실행) 명령어 입력
```
$ cd front-mono
$ yarn && yarn dev
```
정상적으로 프로젝트가 실행된다면 모노레포 구축을 위한 프로젝트 서버 셋팅이 완료된 것<br/><br/>

#### 🔥 TypeScript 에러 수정<br/>
package/front-mono/src/App.tsx에 접근해보면 typescript가 적용되지 않아 에러가 발생함<br/>
본 문제는 패키지매니저 yarn berry에서와 npm에서 모듈을 불러오는 방식의 차이가 있기에 발생하는 오류<br/>
따라서 typescript와 eslint, prettier를 설치해준 뒤, vscode를 사용 중인 경우에 한해 sdk를 설치해줌<br/><br/>

루트 디렉토리로 이동, 아래 명령어 입력
```
$ yarn add -D typescript prettier eslint
$ yarn dlx @yarnpkg/sdks vscode
```
※ sdk부터 설치할 경우 문제가 발생할 가능성이 높음<br/>
※ 반드시 typescript와 같이 1️⃣ 문법에 영향을 주는 package를 먼저 설치한 뒤 2️⃣ sdk를 설치해야 함!<br/><br/>

#### 🔥 Typescript SDK 설정<br/>
command + shift + p를 통해 typescript 버전을 선택하는 창을 열고 Use Workspace Version(sdk)을 선택<br/><br/>


### 6️⃣ project 간 설정 공유 (common ↔️ front-mono)
monorepo를 통해 프로젝트 간 코드를 공유할 수 있는 동시에 5️⃣에서 진행하였던 개발 환경 설정 내용 역시 공유할 수 있음<br/><br/>

최상위 폴더(루트)에 tsconfig.base.json파일을 생성<br/>
tsconfig.base.json 파일의 내용을 아래와 같이 작성<br/>
```
{
    "compilerOptions": {
        "target": "ESNext",
        "useDefineForClassFields": true,
        "lib": ["DOM", "DOM.Iterable", "ESNext"],
        "allowJs": false,
        "skipLibCheck": false,
        "esModuleInterop": false,
        "allowSyntheticDefaultImports": true,
        "strict": true,
        "forceConsistentCasingInFileNames": true,
        "module": "ESNext",
        "moduleResolution": "Node",
        "resolveJsonModule": true,
        "isolatedModules": true,
        "noEmit": true,
        "jsx": "react"
    },
}
```

다음으로 package/front-mono 폴더 내에 위치하는 tsconfig.json의 내용을 아래처럼 수정
```
{
  "compilerOptions": {
  },
  "include": ["./src"],
  "extends": "../../tsconfig.base.json",
}
```
위의 수정사항을 통해 tsconfig.base.json에서 설정한 내용을 extends해 사용할 수 있음<br/><br/>

### 7️⃣ 프로젝트 간 코드 공유
현재 package/common폴더는 비어있는 상태<br/>
common 폴더로 이동한 뒤, 아래 명령어를 통해 package.json 파일을 생성함
```
$ yarn init
```
<br/>

생성된 package.json 파일 아래처럼 수정
```
{
  "name": "@monorepo/common",
  "version": "1.0.0",
  "description": "",
  "main": "index.ts",
  "author": "",
  "license": "ISC"
}
```
<br/>

수정 이후, 프로젝트간 공유하게 될 코드인 index.ts 생성<br/>
index.ts 파일에는 front-mono와 common 프로젝트에서 코드를 공유할 수 있음을 간단하게 확인하기 위해 임의의 변수를 선언

```
const str: string = "monorepo를 통해 공유할 변수";
export default str;
```
<img width="1149" alt="스크린샷 2025-03-07 오후 4 53 57" src="https://github.com/user-attachments/assets/8fca875e-d6de-4dce-af2b-900f54071f9b" />

common 폴더 구조 <br/>
📦common <br/>
 ┣ 📜README.md <br/>
 ┣ 📜index.ts <br/>
 ┗ 📜package.json <br/> <br/>

package/common 폴더에 공유할 코드를 만들어 주었으니 package/front-mono 폴더에서 이를 가져와 사용할 수 있도록 package.json파일을 수정해야 함<br/><br/>

- package/front-mono/package.json<br/>
```
...
  "dependencies": {
    "@monorepo/common": "*",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
...
```
dependencies 부분에 common의 package명을 입력해주고 버전의 경우 *로 설정 <br/>
이후 루트 또는 front-mono로 이동, 아래 명령어로 project간 연결 작업 시작
```
$ yarn
```

### 8️⃣ package 내 타 프로젝트에서 common의 변수 import하기
- package/front-mono/src/App.tsx
```
import React, { useState } from 'react'
import logo from './logo.svg'
import './App.css'
import str from '@monorepo/common'
function App() {
  const [count, setCount] = useState(0)
  console.log(str)
...
  return (
    <div className="App">
        ...
      <h1>{str}</h1>
        ...
    </div>
    )
}
```
<img width="1152" alt="스크린샷 2025-03-07 오후 4 55 11" src="https://github.com/user-attachments/assets/ae45af00-3f0b-4102-a77c-66aeaccc373b" />

<br/><br/>

위와 같이 @monorepo/common으로부터 선언해준 변수 str을 import해 사용할 경우,
<img width="1725" alt="스크린샷 2025-03-07 오후 4 38 58" src="https://github.com/user-attachments/assets/fde3e427-0f9c-4ba3-9af7-6c01968186b9" />
위 사진처럼 프로젝트 간 코드 공유가 자유롭게 잘 작동함을 확인할 수 있음
