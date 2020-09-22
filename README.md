# AFL-Docs.ko

## WorkFlow

1. 원본 저장소 fork
2. 자신의 저장소 (fork된 저장소)를 로컬에 clone
3. 번역작업 (.md파일 생성, 편집)
4. git add, git commit
5. git push origin master 자신의 저장소에 푸시
6. 원본 저장소로 pull-request

## Guideline
* 1)커버리지 측정을 참고하세요!
* 제목은 h3(###)및 영어로, 다음 줄은 ---(줄 변경)으로 바꿔주세요!
* 중간중간 나오는 텍스트 표 혹은 코드는 ```<표>```를 사용하세요!
* 번역은 원문과 최대한 비슷하게, 맥락을 전달하는 한에서만 의역하도록 노력합니다!

## Commit example
```bash
    git commit -m "translate:00-design_statement.md"
```
### 참고: git 버전관리
http://forum.openframeworks.kr/t/1-git/81