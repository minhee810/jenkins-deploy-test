# 📘 Jenkins를 활용한 GitHub → 로컬 자동 배포 테스트 구성 문서 (with. GPT)

## ✅ 1. 목적

* GitHub 저장소에 Push된 코드를 Jenkins가 자동으로 받아서,
* 내 로컬 PC의 특정 디렉토리에 복사하여 웹서버 테스트 환경처럼 사용할 수 있도록 자동 배포 파이프라인 구성

---

## ✅ 2. 구성 요약

| 구성 요소   | 설명                                      |
| ------- | --------------------------------------- |
| Jenkins | Docker 컨테이너 기반으로 로컬에서 실행                |
| GitHub  | 테스트용 코드 저장소                             |
| 배포 디렉토리 | 로컬 PC의 `~/jenkins-deploy-test`          |
| 연결 방식   | Docker 볼륨 마운트 (`-v`)를 통해 컨테이너와 로컬 폴더 연동 |
| 복사 방식   | Jenkins 빌드 단계에서 `rsync`로 파일 복사 및 동기화 수행 |

---

## ✅ 3. Docker로 Jenkins 설치 및 볼륨 마운트

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v ~/jenkins-deploy-test:/deploy-test \
  jenkins/jenkins:lts
```

> `~/jenkins-deploy-test` 폴더가 Jenkins 컨테이너 내부의 `/deploy-test`로 연결됨

---

## ✅ 4. Jenkins 설정 (Freestyle Job)

### 📌 Job 이름: `jenkins-deploy-test`

1. **GitHub project**

   * `https://github.com/your-name/your-repo`

2. **Source Code Management > Git**

   * Repository URL: `https://github.com/your-name/your-repo.git`
   * Branch: `*/main` 또는 `*/develop`

3. **Build Step > Execute shell**

```bash
echo "==== 빌드 시작 ===="
echo "현재 워크스페이스: $WORKSPACE"

rsync -av --delete "$WORKSPACE/" "/deploy-test/"

echo "==== 배포 완료! ===="
```

---

## ✅ 5. 배포 결과 확인

빌드 후 로컬 디렉토리 확인:

```bash
cd ~/jenkins-deploy-test
ls -al
```

또는 PHP 테스트 서버 실행:

```bash
php -S localhost:8000
```

→ 브라우저에서 `http://localhost:8000` 접속

---

## ✅ 6. 트러블슈팅

### ✅ 문제: `rsync: not found`

* Jenkins 컨테이너에는 기본적으로 `rsync` 미설치
* 해결: 컨테이너 내에서 root 권한으로 설치

```bash
docker exec -u root -it jenkins bash
apt update
apt install -y rsync
```

### ✅ 문제: 로컬 디렉토리에 아무 파일도 복사되지 않음

* 원인: Jenkins가 Git 저장소에서 코드를 clone하지 못했거나 clone된 파일이 없음
* 해결: Git 설정 확인 (Repo URL, 브랜치), `.gitignore` 여부 확인

---

## ✅ 7. 기타

* 컨테이너 삭제 및 재시작 시 Jenkins 설정 유지됨 (`jenkins_home` 볼륨 덕분)
* 자동화를 위해 GitHub Webhook + ngrok 설정 가능 (후속 작업)


