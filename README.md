# AFL-Docs.ko

## WorkFlow

1. 원본 저장소 fork
2. 자신의 저장소 (fork된 저장소)를 로컬에 clone
3. 번역작업 (.md파일 생성, 편집)
4. git add, git commit
5. git push origin master 자신의 저장소에 푸시
6. 원본 저장소로 pull-request

두번째 pull-request부터는 1. 2. 를 생략하고, 아래 명령어 이후 3. 부터 진행합니다!
```bash
    git remote add upstream https://github.com/gnbon/AFL-Docs.ko.git
    git fetch upstream
    git merge upstream/master
```

## Guideline
* 1)커버리지 측정을 참고하세요!
* 제목은 h3(###)및 영어로, 다음 줄은 ---(줄 변경)으로 바꿔주세요!
* 중간중간 나오는 텍스트 표 혹은 코드는 ```를 사용하세요!
* 번역은 원문을 최대한 따라가도록 하며, 맥락을 더 낫게 전달하는 한에서만 의역하도록 노력합시다!

## Commit example
```bash
    git commit -m "translate:00-design_statement.md"
```
### 참고: git 버전관리
http://forum.openframeworks.kr/t/1-git/81