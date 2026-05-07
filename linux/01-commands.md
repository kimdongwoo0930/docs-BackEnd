# 01. 자주 쓰는 명령어

<br>

## 1. 왜 Linux 명령어를 알아야 하는가?

백엔드 개발자는 결국 서버에서 작업합니다. 로컬에서 잘 돌아가던 애플리케이션이 서버에서 문제가 생겼을 때, GUI 없이 터미널만으로 원인을 파악하고 해결해야 합니다.

**실무에서 마주치는 상황:**

```
- 서버에 배포했는데 포트가 안 열림 → 프로세스가 살아있는지 확인해야 함
- 로그 파일이 너무 커서 디스크가 꽉 참 → 로그 파일 위치 찾고 정리해야 함
- 특정 유저로 실행되어야 하는데 권한 오류 → 파일 권한 확인 및 수정
- 배포 스크립트 자동화 → Shell 스크립트 작성
```

이 상황들을 해결하려면 Linux 명령어가 필수입니다.

<br>

## 2. 어떻게 동작하는가?

### 📁 파일/디렉토리 탐색

```bash
# 현재 위치 확인
pwd
# 출력: /home/ubuntu

# 디렉토리 내용 보기
ls          # 기본
ls -l       # 상세 정보 (권한, 소유자, 크기, 날짜)
ls -la      # 숨김 파일(.으로 시작)까지 포함
ls -lh      # 파일 크기를 사람이 읽기 쉬운 단위로 (KB, MB)

# 디렉토리 이동
cd /etc         # 절대경로
cd ../logs      # 상대경로 (한 단계 위로 → logs 폴더)
cd ~            # 홈 디렉토리로

# 파일/폴더 생성
mkdir my-dir            # 디렉토리 생성
mkdir -p a/b/c          # 중간 디렉토리까지 한번에 생성
touch app.log           # 빈 파일 생성

# 복사, 이동, 삭제
cp file.txt backup.txt      # 파일 복사
cp -r dir1 dir2             # 디렉토리 복사 (-r: recursive)
mv file.txt /tmp/           # 이동 (rename도 mv로 함)
rm file.txt                 # 파일 삭제
rm -rf dir/                 # 디렉토리 강제 삭제 (⚠️ 주의)
```

---

### 🔍 파일 내용 확인 & 검색

```bash
# 파일 내용 보기
cat app.log             # 전체 출력
head -n 20 app.log      # 앞 20줄만
tail -n 50 app.log      # 뒤 50줄만
tail -f app.log         # 실시간으로 추가되는 내용 모니터링 (로그 볼 때 필수)

# 내용 검색 (grep)
grep "ERROR" app.log                    # ERROR 문자열이 포함된 줄 출력
grep -i "error" app.log                 # 대소문자 무시
grep -n "NullPointer" app.log           # 줄 번호 포함
grep -r "TODO" ./src                    # 디렉토리 내 재귀 검색
grep -A 3 "Exception" app.log           # 매칭된 줄 + 아래 3줄 같이 출력

# 파일 찾기 (find)
find /var/log -name "*.log"             # 확장자로 찾기
find . -name "*.java" -mtime -1         # 1일 이내 수정된 .java 파일
find /tmp -size +100M                   # 100MB 이상 파일 찾기
```

---

### ⚙️ 프로세스 확인

```bash
# 실행 중인 프로세스 확인
ps aux                          # 전체 프로세스 목록
ps aux | grep java              # java 프로세스만 필터링
ps aux | grep spring            # Spring 서버 확인

# 실시간 프로세스 모니터링
top                             # CPU/메모리 사용량 실시간 확인
htop                            # top의 개선 버전 (더 보기 좋음)

# 특정 포트 사용 프로세스 확인
lsof -i :8080                   # 8080 포트를 사용 중인 프로세스
ss -tlnp | grep 8080            # 더 빠른 대안

# 프로세스 종료
kill 1234                       # PID로 정상 종료 요청 (SIGTERM)
kill -9 1234                    # 강제 종료 (SIGKILL, 저장 안 됨)
pkill -f "spring"               # 이름으로 찾아서 종료
```

---

### 💾 디스크 & 용량 확인

```bash
# 디스크 사용량
df -h                           # 전체 디스크 사용량 (마운트 포인트별)
df -h /                         # 루트 파티션만

# 디렉토리/파일 용량
du -sh /var/log                 # 특정 디렉토리 총 크기
du -sh *                        # 현재 디렉토리 항목별 크기
du -sh * | sort -rh | head -10  # 가장 큰 항목 Top 10
```

<br>

## 3. 다른 방법과 무엇이 다른가?

### grep vs find vs awk

| 명령어 | 목적 | 사용 상황 |
|--------|------|----------|
| `grep` | **내용** 검색 | 파일 안에서 특정 문자열 찾을 때 |
| `find` | **파일/디렉토리** 검색 | 파일 이름, 날짜, 크기로 찾을 때 |
| `awk` | **텍스트 처리/변환** | 특정 컬럼 추출, 집계할 때 |

```bash
# 실무 예시: 오늘 ERROR 로그 중 몇 건인지 세기
grep "ERROR" app.log | grep "2024-01-15" | wc -l

# 실무 예시: 로그에서 IP 주소만 추출
grep "Request from" app.log | awk '{print $3}' | sort | uniq -c
```

### kill vs kill -9

```
kill (SIGTERM): "정리하고 종료해" → 프로세스가 파일 저장, 연결 종료 후 정상 종료
kill -9 (SIGKILL): "지금 당장 죽어" → OS가 강제로 프로세스 제거, 저장 안 됨

→ 항상 kill로 먼저 시도하고, 안 될 때만 kill -9 사용
```

<br>

## 4. 실제 코드에서는 어떻게 쓰는가?

### Spring Boot 서버 배포/관리 시나리오

```bash
# 1. 현재 서버에서 실행 중인 Spring 프로세스 확인
ps aux | grep java

# 2. 기존 서버 종료
PID=$(ps aux | grep 'myapp.jar' | grep -v grep | awk '{print $2}')
kill $PID

# 3. 새 버전 실행 (백그라운드 + 로그 파일로)
nohup java -jar myapp.jar > /var/log/myapp/app.log 2>&1 &

# 4. 실행됐는지 확인
sleep 3
ps aux | grep java
curl http://localhost:8080/actuator/health
```

---

### 로그 트러블슈팅 시나리오

```bash
# 오늘 발생한 에러 로그 확인
tail -f /var/log/myapp/app.log | grep "ERROR"

# 특정 시간대 에러 수집
grep "ERROR" app.log | grep "2024-01-15 14:" > error_report.txt

# 가장 많이 발생한 에러 유형 확인
grep "ERROR" app.log | awk '{print $5}' | sort | uniq -c | sort -rn | head -5
```

<br>

## 5. 면접 Q&A

**Q. `kill`과 `kill -9`의 차이를 설명해주세요.**

> `kill`은 기본적으로 SIGTERM 신호를 보내는데, 이는 프로세스에게 "정상적으로 종료하라"는 요청입니다. 프로세스는 이 신호를 받으면 파일 저장, 네트워크 연결 종료 등 정리 작업을 한 뒤 종료됩니다. 반면 `kill -9`는 SIGKILL로, OS 커널이 프로세스를 강제로 즉시 제거합니다. 프로세스가 이 신호를 무시하거나 처리할 수 없기 때문에 저장되지 않은 데이터가 손실될 수 있습니다. 실무에서는 항상 `kill`로 먼저 시도하고, 응답이 없을 때만 `kill -9`를 사용합니다.

---

**Q. `ps aux`에서 각 컬럼이 의미하는 바를 설명해주세요.**

> `USER`는 프로세스 소유자, `PID`는 프로세스 ID, `%CPU`와 `%MEM`은 CPU/메모리 사용률, `VSZ`는 가상 메모리 크기, `RSS`는 실제 메모리 사용량, `STAT`은 프로세스 상태(R: 실행 중, S: 슬립, Z: 좀비 등), `COMMAND`는 실행 명령어입니다. 서버 트러블슈팅 시 PID와 %CPU, %MEM을 주로 확인합니다.

---

**Q. `tail -f`를 실무에서 어떻게 활용하나요?**

> `tail -f`는 파일의 끝을 실시간으로 모니터링합니다. 주로 배포 후 애플리케이션 로그를 실시간으로 보거나, 에러가 발생했을 때 즉시 확인하는 데 사용합니다. `grep`과 파이프로 연결하면 `tail -f app.log | grep ERROR` 처럼 에러 로그만 실시간으로 필터링해서 볼 수 있어 유용합니다.

---

**Q. 디스크가 꽉 찼을 때 어떤 순서로 조사하나요?**

> 먼저 `df -h`로 어느 파티션이 꽉 찼는지 확인합니다. 그 다음 해당 파티션에서 `du -sh * | sort -rh | head -10`으로 가장 큰 디렉토리를 찾고, 그 안을 파고들어 원인을 찾습니다. 실무에서는 `/var/log`의 로그 파일이나 `/tmp`의 임시 파일이 주범인 경우가 많습니다. 불필요한 로그는 삭제하거나 `logrotate` 설정으로 자동 관리합니다.
