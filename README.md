# Code Question Bot
(어떤 프로젝트인지, 실행 예시)

## 목차
[1. Discord Webhook 생성 가이드](#1-discord-webhook-생성-가이드)

[2. Github Action yml 설정](#2-github-action-yml-설정)

[3. GitHub userId 추출](#3-github-userid-추출)


## 1. Discord Webhook 생성 가이드
1. 서버 생성
- 왼쪽 하단 탭에서 서버 추가하기(➕) 버튼을 눌러 서버를 생성합니다

    ![디코](https://github.com/user-attachments/assets/705b4f84-ec0b-408e-a741-903450064e86)

2. 채널 생성
- 왼쪽 채팅 채널에서 채널 만들기(➕) 버튼을 눌러 채널을 생성합니다 
    > 💡 본인이 생성한 서버에서만 채널 생성과 편집이 가능합니다

    ![image](https://github.com/user-attachments/assets/608a0b52-c041-422f-83ae-0bb72cbaeee1)

3. 채널 편집으로 이동
- 생성된 채널의 설정(⚙️) 버튼으로 이동합니다

    ![image](https://github.com/user-attachments/assets/05439fc5-f692-44fc-b276-e28d3915ff71)

4. 연동 탭에서 웹후크 생성
- 왼쪽 연동 탭을 클릭하고 웹후크 만들기 버튼을 클릭 웹후크를 생성합니다

    ![image](https://github.com/user-attachments/assets/a498a046-ae4a-42ac-94e4-6ba5eff59b96)

5. 웹후크 주소 복사
- 생성된 웹후크를 클릭하고 하단에 웹후크 URL 복사 버튼을 클릭하여 URL을 복사할 수 있습니다.

    ![image](https://github.com/user-attachments/assets/095945cd-af72-4ca9-93b6-e9905af3f9c1)

## 2. Github Action yml 설정

1. 레포지토리의 Add file -> Create File 선택
    ![image](https://github.com/user-attachments/assets/edb3c8a9-3f2b-4a22-a361-d715361f87bd)

2. 파일 이름을 적는 칸에 .github/workflows/cobot.yml 을 차례대로 입력합니다.
    > 💡 파일 이름은 바꿔도 무방

    ![image](https://github.com/user-attachments/assets/805b2855-0f5e-4afa-a432-c158f3af8d85)

3. 생성한 yml 파일에 해당 코드를 붙여넣기

    <details>
        <summary>yml 코드 보기</summary>

    ```yml
    name: Cobot workflow

    on:
    push:
        branches:
        - main

    jobs:
    diff-changes:
        runs-on: ubuntu-latest

        steps:
        - name: Checkout repository
            uses: actions/checkout@v2

        - name: Get the recent commit author
            run: |
            author_id=$(git log -1 --format="%an <%ae>" | grep -o '<[0-9]\+' | sed 's/<//')
            echo "author_id=$author_id" >> $GITHUB_ENV

        - name: save to diff.txt
            run: |
            git fetch origin ${{ github.event.before }} ${{ github.event.after }} # 이전 커밋과 현재 커밋 가져오기
            git diff ${{ github.event.before }} ${{ github.event.after }} | grep '^+[^+]' | sed 's/^+//' > diff.txt # 변경 사항 중 추가된 내용만을 가져와 txt 생성
            cat diff.txt

        - name: Create payload.json
            run: |
            diff=$(cat diff.txt | jq -Rs .) # diff.txt를 josn 형식의 값으로 변환
            echo "{\"author_id\": \"${{ env.author_id }}\", \"diff\": $diff}" > payload.json # author와 diff를 json 확장자로 변환
            cat payload.json

        - name: Send POST request with payload.json
            run: |
            curl -X POST "https://gdf2ppbhtkfw36sjoqy7pc3ar40istkm.lambda-url.ap-northeast-2.on.aws/" \
            -H "Content-Type: application/json" \
            -d @payload.json
    </details>

<br>

4. Commit changes를 통해 변경 사항 저장

    ![커밋](https://github.com/user-attachments/assets/6e9afa9b-72fe-4c52-bbe5-7cc03de05d65)


## 3. GitHub userId 추출
> 🚨 userId는 해당 서비스를 사용하는 유저를 식별하기 위해 사용됩니다


### 방법 1) curl을 통해 id값 추출
```
curl https://api.github.com/users/{깃헙id}
```
- 예시

    ![id](https://github.com/user-attachments/assets/4b93580c-959f-446a-9915-5d835fa60205)

### 방법 2) 깃헙 프로필 페이지 소스보기에서 값 찾기
1. 깃허브 로그인
2. 깃허브 홈(혹은 아무 페이지)에서 우클릭 후 페이지 소스 보기
3. Ctrl + F를 통해 user_id를 검색

<div align="right">
  
[목차로](#목차)

</div>
