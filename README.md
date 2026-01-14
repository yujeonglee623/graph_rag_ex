# graph DB(neo4j) 활용 영화 추천
## graph RAG 소개
- Graph 데이터베이스를 기반으로 한 질의 응답 시스템(GraphRAG)은 전통적인 벡터 기반 RAG 시스템보다 더 정확하고 연관성 있는 답변을 제공함. 
- 자연어 질문을 Neo4j Cypher 쿼리로 변환하여 지식 그래프를 효과적으로 탐색함.

### 특장점:
- 정확한 관계 검색: 그래프 데이터베이스의 관계 중심 구조를 활용해 복잡한 연결 패턴을 찾을 수 있음.
- 컨텍스트 유지: 엔티티 간의 관계를 유지하여 더 풍부한 컨텍스트를 제공함.
구조화된 정보 검색: 단순 텍스트 검색이 아닌 구조화된 방식으로 정보를 검색함.


# 설치 라이브러리
```
pip install langchain, langchain-neo4j, langchain-openai
pip install pandas
```

# 실습 설명
## 실습1
- 파일 : 1.neo4j_movie_graphdb구축_csv.ipynb
### 내용
#### 1. 환경설정 및 DB 연결
- 사용 라이브러리: dotenv, langchain_neo4j, langchain_core, pandas
- .env 파일: neo4j desktop 정보, openai api key, gemini api key
- DB 초기화: reset_database(graph) 사용 > 코드 다시 돌릴 때 마다 DB 초기화 후 시작하게 해줌
#### 2. 스키마 정의
- 온톨로지: 특정 도메인 내 추상화된 개념과 그 관계를 체계적으로 표현하는 지식구조
    -> Neo4j: 제약 조건과 인덱스를 활용하여 데이터 모델 구현
- Node: Movie, Person, Genre
- Relationship
    - :DIRECTED - Person(감독) -> Movie
    - :ACTED_IN - Person(배우) -> Movie
    - :IN_GENRE - Movie -> Genre
- 제약조건/인덱스
    - 제약조건(Constraint) 생성하여 데이터의 무결성 보장
        - Movie, Person, Genre 노드의 각 ID와 이름에 고유성 제약조건 설정
    - 인덱스(Index) 생성하여 검색 성능 최적화
- Constrains 생성 코드
```
constraints = [
    "CREATE CONSTRAINT movie_id_unique IF NOT EXISTS FOR (m:Movie) REQUIRE m.id IS UNIQUE",
    "CREATE CONSTRAINT person_name_unique IF NOT EXISTS FOR (p:Person) REQUIRE p.name IS UNIQUE",
    "CREATE CONSTRAINT genre_name_unique IF NOT EXISTS FOR (g:Genre) REQUIRE g.name IS UNIQUE"
]
```
- Index 생성 코드
```
indexes = [
    "CREATE INDEX movie_title_index IF NOT EXISTS FOR (m:Movie) ON (m.title)",
    "CREATE INDEX movie_release_index IF NOT EXISTS FOR (m:Movie) ON (m.released)",
]
```
#### 3. 데이터 로드 및 전처리
- CSV 파일을 pandas로 읽어온 후 df으로 생성
- df.shape를 이용하여 데이터 크기 파악
#### 4. 그래프 변환
- node_dict: Person, Genre 중복 방지하기 위해 node_dict={} 딕셔너리 초기화 후 생성된 노드 객체 저장
- relationships: 관계 저장할 리스트 초기화
- batch size를 100으로 설정 후 배치 내 나누어 처리
    - 배치 내 csv 데이터를 순회하며 그래프 구조로 변환
    - 영화 노드 이미 존재하는지 확인 후 존재하지 않으면 생성
        - 영화 속성: id(영화 고유 ID), title(영화 제목), released(개봉 일), rating(평점)
        - 기타 속성
    - 감독, 배우, 장르 정보 처리 및 컬럼 전처리
        1. 결측치 처리 - pd.notna()를 사용하여 데이터가 비어있다면 노드 및 관계를 만들지 않도록 함
        2. 감독, 배우, 장르 각각의 노드 생성
        3. 각각의 컬럼을 |로 쪼개서 DIRECTED, ACTED_IN, IN_GENRE 관계 설정
#### 5. DB 저장
- GraphDocument: 그래프전체 문서를 표현
    - 모든 노드와 관계 수집 후 graph.add_graph_documents() 쿼리 사용하여 DB에 저장

## 실습2
- 파일 : 2.neo4j_movie_basic_search.ipynb
### 내용
- 
#### 1. 기초 현황 파악
1) DB 연결
2) 평점 Top 10 영화 조회
    - ORDER BY m.rating DESC를 이용하여 평점이 높은 순서대로 정렬
    - LIMIT 10을 이용하여 상위 10개만 조회
    - 확인하기 용이하게 데이터 프레임 변환
3) 다작 배우 Top 10 조회
    - MATCH (p)-[:ACTED_IN]->(m)
    - count(m), ORDER BY, LIMIT 10쿼리를 이용하여 배우별로 연결된 영화 노드의 수 계산 후 상위 10개 배우 정렬
4) 장르별 영화 분포
    - MATCH (m)-[:IN_GENRE]->(g) 패턴을 이용하여 장르별 영화 갯수 계산
#### 2.관계 심층 분석
- 특정 배우와 함께 출연한 배우 찾기
    - MATCH ($actor)-[:ACTED_IN]->(m)<-[:ACTED_IN]-(co_actor)
    - WHERE actor <> co_actor

## 실습3
- 파일 : 3.neo4j_movie_full-text_search.ipynb
### 내용


## 실습4
- 파일 : 4.neo4j_movie_vector_search.ipynb
### 내용


## 실습5
- 파일 : 5.neo4j_movie_text2cypher.ipynb

### 내용


## 실습6
- 파일 : 6.neo4j_movie_graphVector_RAG.ipynb

### 내용

## 실습7
- 파일 : 7.neo4j_movie_graphRAG_hybrid.ipynb

### Neo4j 기반 하이브리드 RAG
- Vector RAG + Graph RAG를 활용한 서비스 구현하기

