# 실제 서비스 링크

[KCC 2023 홈페이지](https://www.kiise.or.kr/kcc2023/)

## 프로젝트 카피본 링크
### [관리자 페이지](https://kcc2023-2.web.app/admin.html)  
id : kcc2023@kiise.com  
pwd : kcc2023

### [유저 페이지](https://kcc2023-2.web.app/)  
id : test@test.com / 9@9.com  
pwd : 010123456 / 99999999

## 개요

KCC 2023 자료집 사이트는 한국정보과학회에서 개최된 KCC2023 행사 정보와 신청한 항목의 자료집 링크 제공을 위한 웹사이트입니다. 행사 참여 회원의 정보를 관리자 페이지에서 관리할 수 있습니다. 

## 관리자 페이지

- 관리자 로그인/로그아웃
- 검색 기능으로 참가자 정보 확인
- 행사 참여자 정보 생성, 삭제, 수정

## 유저 페이지

- 참가자 로그인/로그아웃
- 행사 일정
- 행사 정보
- 참가자가 신청한 튜토리얼, 워크숍의 발표 자료집 확인

## 제작 기간

2023년 6월 2일 ~ 2023년 6월 16일 (2주 소요)

## 기술 스택

HTML, CSS, Javascript, Node.js

Firebase DB(Realtime Database, Firestore), Firebase Authentication

## 문제 해결

### 1. Firestore 컬렉션 uid 에러
테스트 코드 작성 시 컬렉션의 uid를 식별하기 쉽도록 **‘유저의 이름 + 전화번호’** 로 생성했었습니다. 기존 시스템에서 어떻게 유저 데이터를 저장했는지 알 수 없는 상황이었고 실제 유저 데이터를 볼 권한이 없었기에 테스트 코드를 넘겨드렸고 실제 데이터를 JSON으로 변환해 Firestore에 임포트 하려는 순간 에러가 발생했습니다. JSON의 값의 어떤 오류가 있어 임포트를 할 수 없다는 것이었습니다. 실제 유저 데이터에 문제가 있다고 판단된다, 말씀을 드리고 함께 문제가 된 데이터를 찾아봤습니다. 

DB에 들어갈 정보는 **이메일 주소, 이름, 전화번호, 소속, 신청항목** 입니다.
Firestore의 uid에는 특수문자가 들어가면 안된다는 규칙이 있었습니다. uid 조합을 ‘이름 + 전화번호’로 했었기 때문에 이름에 특수문자가 들어간 데이터가 있는지 .csv 파일에서 수색한 결과, 유저의 이름 중 영어로 된 이름에 **‘.(온점)’** 이 포함되어 있었습니다. 데이터 확인 후 의논 끝에 해당 이름을 수정하여 성공적으로 임포트를 마쳤습니다.

### 2. 2천명의 유저 데이터 DB에 옮기는 과정
자료집 사이트에 접속한 유저가 로그인 할 수 있도록 기존 가입되어 있는 회원 정보가 담긴 엑셀 파일을 DB에 옮기는 작업을 해야했습니다. 데이터베이스 구조 설계 후 대량의 데이터를 한꺼번에 DB에 넣을 수 있도록 아래와 같은 순서로 개발을 했고 필요한 기능을 Javascript 파일로 만들어 node.js 로 실행하였습니다.

> 1) 유저 데이터 파일(CSV) → JSON 변환 (csv-parser.js 실행)
> 2) Realtime DB에 변환한 JSON 파일 import
> 3) node-firestore-import-export 패키지 설치
> 4) Generate Key 가 담긴 JSON, 변환한 유저 데이터 JSON 파일을 터미널에 입력 후 node-firestore-import-export 실행

### 3. 회원이 선택한 항목에만 자료집 링크 주소 연결
    
선택한 항목에 관하여 제공되는 자료집 링크 주소는 선택한 회원만 보이도록 구현해야 했습니다. 소스 코드 상에 주소가 유출되지 않도록 로그인한 유저가 선택한 항목 데이터를 DB에서 받아와 해당 데이터와 일치하는 항목의 link 주소를 화면의 버튼에 연결했습니다.
    
### 4. 보안 이슈
    
    Firebase DB에 접근하여 로그인을 시도한 유저의 정보와 일치할 경우 로그인이 가능하도록 구현했습니다. Firebase DB 보안 규칙도 테스트 모드로 시작을 했기 때문에 콘솔 창에서 DB에 접근이 가능해 유저 정보가 유출되는 이슈가 있었습니다.
    
    개발 설계 당시 Authentication 이용해 로그인이 가능하도록 구현하려 했으나 
    
    - 기존 시스템을 통해 이미 존재하는 대량의 회원 정보를 Authentication에 등록 시 걸리는 시간 및 대량 등록을 지원하지 않음
    - 관리자 페이지에서 회원 추가 및 수정 시 authentication에 반영되지 않음
    
    위와 같은 한계가 있어 DB에 접근하는 방식으로 구현했었습니다.
    
    <aside>
    💡 보안 이슈 해결 방안을 찾아본 끝에 대량의 회원 정보를 임포트할 수 있는 방법을 발견했고, Authentication을 이용하여 등록된 유저만 로그인이 가능하도록 하였습니다.
    
    </aside>
    
    - ****auth-parser.js : .csv 파일의 유저 데이터를 authentication에 등록하기 위해 데이터 구조를 만드는 로직****
    - ****auth-import.js  : auth-parser.js에서 만든 데이터를 authentication에 대량으로 import****
    - ****auth-delete.js  : authentication에 등록된 회원 정보를 한꺼번에 삭제하는 로직****
    
    보안 규칙을 세분화하여 
    
    - 안증된 유저 + 로그인한 유저 ⇒
        - 로그인한 유저의 정보만 확인 가능.
        - link 주소 데이터가 담긴 컬렉션에 접근 가능.
        - 다른 유저 정보 접근 불가.
    - 인증된 관리자 ⇒ 모든 유저의 정보 접근 및 수정 가능.

```jsx
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 인증된 사용자인지 확인하는 함수
    function isAuthenticated() {
      return exists(/databases/$(database)/documents/users/$(request.auth.uid));
    }

    // users 컬렉션에 대한 읽기 권한을 허용 (인증된 사용자만 접근 가능)
    match /users/{userId} {
      allow read: if isAuthenticated() && request.auth.uid == userId;
    }
    
    // users 컬렉션에 대한 접근 권한을 허용하기 위한 규칙 추가
    match /users/{document=**} {
      allow read, write: if request.auth.uid == 'admin';
    }
    
    // link documentId 대한 읽기 권한을 허용 (인증된 사용자만 접근 가능)
    match /link/{document=**} {
      allow read: if isAuthenticated();
      allow write: if false;
    }

    // 그 외 모든 컬렉션에 대한 읽기 및 쓰기 권한을 거부
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```
