AI 서비스 출시 시 고려해야 할 요소
1. Hallucination (환각 현상)
LLM이 사실과 다르거나 의도와 다른 결과를 생성하는 문제
특히 신뢰성이 중요한 도메인(금융, 의료 등)에서 치명적일 수 있다.

고려사항
* 결과 검증 로직 추가 (rule-based, 외부 API 등)
* RAG(Retrieval-Augmented Generation) 적용
* 출력 포맷 제한 및 가이드라인 강화

2. Poor Data Retrieval (데이터 검색 품질 문제)
잘못되거나 신뢰할 수 없는 데이터가 프롬프트에 포함되는 경우

고려 사항
* 데이터 소스의 신뢰성 관리
* 검색 결과 필터링 및 랭킹 개선
* 최신 데이터 유지 (freshness)

3. Evaluation (평가 및 품질 관리)
사용자에게 제공되는 결과의 품질을 어떻게 검증할 것인가

고려 사항
* 자동 평가 지표 구축 (정확도, 유사도)
* 휴먼 평가 (Human-in-the-loop)
* A/B 테스트를 통한 성능 비교

4. LLM Cost Management (비용 관리)
LLM 호출 비용이 서비스 운영 비용에 큰 영향을 미침

고려 사항
* 토큰 사용량 최적화 (프롬프트/응답 길이 제한)
* 캐싱 전략 적용
* 적절한 모델 선택 (성능 vs 비용 트레이드오프)

5. Technical Issues (기술적 문제)
트래픽 증가나 외부 API 의존성으로 인한 장애 가능성

고려 사항
* Rate Limit 대응 (retry, backoff 전략)
* fallback 모델 또는 graceful degradation
* 비동기 처리 및 큐 기반 아키텍처

## ETL vs ELT
### ETL (Extract → Transform → Load)
외부 소스에서 데이터를 추출한 뒤, 애플리케이션 레벨(Java/Python 등)에서 가공하고, 최종 결과만 DB에 저장하는 방식
가공 로직(Transform)이 저장 이전에 실행되기 때문에 가공 방식을 바꾸고 싶으면 원본 데이터를 처음부터 다시 수집해야 한다
예를들어 로그 수집 → 파싱 → DB 저장 순서에서 파싱 로직이 바뀌면 이미 저장된 데이터는 원본이 아닌 가공된 결과이므로 과거 데이터를 새 로직으로 다시 처리하기 어렵다.

### ELT (Extract → Load → Transform)
외부 소스에서 데이터를 추출한 뒤, 원본(Raw) 그대로 내부 저장소에 먼저 적재하고 이후 필요에 따라 가공하는 방식.

```text
외부 데이터 소스 (Confluence, Slack, S3 등)
        ↓  Extract
Raw 저장소 (DB / S3 / Data Lake)
        ↓  Transform
가공된 데이터 (검색용 인덱스, Vector DB 등)
```
예를들어 로그 수집 → raw JSON 그대로 저장 → 나중에 원하는 방식으로 파싱
원본이 내부 저장소에 남아있으므로 가공 로직을 바꿀 때 외부 소스를 다시 수집할 필요 없이 Transform만 재실행하면된다.
