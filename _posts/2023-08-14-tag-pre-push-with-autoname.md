---
layout: post
title: "Git hook pre-push로 tag name 변경"
---

개발팀 미팅 중에 현재 구축 중인 CI/CD 시스템에서 회귀(regeression) 테스트 실행에 git tag를 이벤트 트리거로 사용한다는 말을 들었다.
특정 네이밍의 tag가 push 되면 회귀 테스트가 동작하는 시나리오인데, 네이밍 특성상 여러 명이 회귀 테스트 실행을 시도하면 같은 tag 명이 push 되어 테스트 실행이 비정상적으로 될 우려가 있다고 생각했다.
그래서 해결방안으로 tag push 전에 git hook으로 tag에 고유값을 넣어 중복을 막을 수 있을 것 같다고 의견을 내었고,
이 작업은 고대로 나에게 할당되었다. (껄껄..)

목표는 tag 가 push 되기전에 회귀 테스트 트리거 용 tag를 획득해서 tag 가 위치하는 commit object name을 tag 명에 붙여서 push를 하자!!
요렇게 하면 commit object name 을 고유값으로 사용할 수 있어 tag 중복을 막을 수 있고, 개발자가 직접 commit object name을 붙여야 하는 번거로움도 피할 수 있으니깐.
위에 주저리주저리 적었지만 요약해보면 크게 세 가지이다.
* 특정 네이밍의 tag 가져오기
* tag 위치의 commit object name 가져오기
* pre-push 시점에 tag 수정

# 특정 네이밍 tag 가져오기
``` git tag -l <pattern> ```
간단하게 git tag 명령어를 사용해서 특정 패턴을 가지는 tag 리스트를 얻어올 수 있다.
패턴은 shell wildcard을 사용할 수 있다.

예를 들어 'test' 문자열을 포함하는 tag 를 획득하려면 아래와 같이 입력하면 된다.
``` git tag -l "*test" ```

# tag 위치의 commit object name 가져오기
``` git rev-list --max-count=1 --abbrev-commit <tag name> ```
rev-list 명령은 최근 생성된 commit 부터 표시하는 명령어이고,
여기에 축약된 commit object name 을 가져오도록 --abbrev-commit 옵션을 주었다.
원래 commit object name 은 40자의 hex값이지만 축약된 값을 사용하더라도 고유값으로 사용할 수 있어서 축약된 값을 사용했다.

# pre-push 시점에 tag 수정
이제 위 두 가지를 사용하여 hook 스크립트를 작성해 보자.
git 에서 동작하는 모든 hook 스크립트들은 git 프로젝트 폴더의 '. git/hooks/' 경로에 있어야 한다.
나는 tag 가 push 되기 전에 tag를 수정할 것이므로 pre-push hook을 사용하였다.

'. git/hooks/pre-push' 파일을 생성하고 스크립트를 아래와 같이 작성했다.

<pre><code>
#!/bin/sh

regression_test_tags=$(git tag -l "*/Test")

if [[ "$regression_test_tags" == "" ]]; then
  exit 0
fi

for prev_tag in $regression_test_tags
do
  commit_id=`git rev-list --max-count=1 --abbrev-commit $prev_tag`
  renamed_tag=$prev_tag/$commit_id
  
  `git tag -d $prev_tag`
  `git tag $renamed_tag $commit_id`
done
 
exit 0
</code></pre>