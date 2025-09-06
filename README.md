# ELK Stack Docker Compose

Docker Compose를 사용한 완전한 ELK (Elasticsearch, Logstash, Kibana) 스택 설정으로 로그 수집, 분석 및 시각화를 제공합니다.

## 개요

이 설정은 다음을 제공합니다:
- **Elasticsearch 7.17.10** - 검색 및 분석 엔진
- **Logstash 7.17.10** - 데이터 처리 파이프라인
- **Kibana 7.17.10** - 데이터 시각화 및 탐색
- **8.x 로 업데이트할 예정**

## 빠른 시작

1. **프로젝트 클론 및 이동**
   ```bash
   cd elk-docker-compose
   ```

2. **ELK 스택 시작**
   ```bash
   docker-compose up -d
   ```

3. **서비스 접근**
   - Kibana: http://localhost:5601
   - Elasticsearch: http://localhost:9200
   - Logstash: TCP 포트 5010, UDP 포트 5009

4. **기본 인증 정보**
   - 사용자명: `elastic`
   - 비밀번호: `changeme`

## 서비스 구성

### Elasticsearch
- **포트**: 9200 (HTTP), 9300 (Transport)
- **메모리**: 256MB 힙 크기
- **보안**: X-Pack 활성화 및 평가판 라이선스
- **데이터**: Docker 볼륨에 저장

### Logstash
- **포트**: 
  - 5010/tcp - JSON 라인 입력
  - 5009/udp - 추가 입력
  - 9600 - 모니터링 API
- **파이프라인**: JSON 로그 수신 및 Elasticsearch로 전달 구성
- **메모리**: 256MB 힙 크기

### Kibana
- **포트**: 5601
- **기능**: 모니터링 활성화된 Elasticsearch 전체 액세스

## Logstash로 로그 전송

### TCP JSON 입력 (포트 5010)
```bash
echo '{"message":"안녕 ELK","level":"info","timestamp":"2024-01-01T12:00:00Z"}' | nc localhost 5010
```

### 애플리케이션 통합
애플리케이션에서 다음 설정으로 JSON 로그 전송:
- **호스트**: localhost
- **포트**: 5010
- **프로토콜**: TCP
- **형식**: JSON lines

## 디렉토리 구조

```
elk-docker-compose/
├── docker-compose.yml           # 메인 compose 구성
├── elasticsearch/
│   └── config/
│       └── elasticsearch.yml    # Elasticsearch 구성
├── kibana/
│   └── config/
│       └── kibana.yml           # Kibana 구성
└── logstash/
    ├── config/
    │   └── logstash.yml         # Logstash 구성
    └── pipeline/
        └── logstash.conf        # Logstash 파이프라인 구성
```

## 사용자 정의

### Logstash 파이프라인 수정
`logstash/pipeline/logstash.conf` 파일을 편집하여 다음을 사용자 정의:
- 입력 소스 (beats, syslog 등)
- 데이터 처리 및 필터링
- 출력 대상

### 보안 구성
⚠️ **중요**: 프로덕션 사용 전 기본 비밀번호를 변경하세요!

1. 다음 파일에서 비밀번호 업데이트:
   - `docker-compose.yml` (ELASTIC_PASSWORD)
   - `kibana/config/kibana.yml`
   - `logstash/config/logstash.yml`
   - `logstash/pipeline/logstash.conf`

### 성능 튜닝
`docker-compose.yml`에서 메모리 설정 조정:
```yaml
environment:
  ES_JAVA_OPTS: "-Xmx512m -Xms512m"  # 프로덕션용으로 증가
  LS_JAVA_OPTS: "-Xmx512m -Xms512m"  # 프로덕션용으로 증가
```

## 모니터링

모니터링 기능 접근:
- Elasticsearch: http://localhost:9200/_cluster/health
- Logstash: http://localhost:9600
- Kibana 스택 모니터링: http://localhost:5601/app/monitoring

## 문제 해결

### 서비스 상태 확인
```bash
docker-compose ps
```

### 로그 확인
```bash
docker-compose logs elasticsearch
docker-compose logs logstash
docker-compose logs kibana
```

### 일반적인 문제

1. **메모리 부족 오류**
   - docker-compose.yml에서 힙 크기 증가
   - Docker에 충분한 메모리가 할당되었는지 확인

2. **연결 거부**
   - 모든 서비스가 시작될 때까지 대기 (Elasticsearch가 가장 오래 걸림)
   - 서비스 의존성 및 시작 순서 확인

3. **인증 오류**
   - 모든 구성 파일에서 인증 정보 일치 확인
   - X-Pack 초기화 완료까지 대기

## 프로덕션 고려사항

- 기본 비밀번호 변경
- 적절한 SSL/TLS 인증서 구성
- 외부 데이터 지속성 설정
- 적절한 백업 전략 구현
- 리소스 제한 및 모니터링 구성
- 인증 정보를 위한 시크릿 관리 사용
- 감사 로깅 활성화
- 적절한 방화벽 규칙 구성
