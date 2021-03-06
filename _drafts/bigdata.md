# 빅데이터(Big Data)

## 3장

- 데이터가 불변성과 영원성을 지니는 것이 왜 중요한지?
- 마스터 데이터 집합을 위한 데이터 모델과 그 모델을 그래프 스키마로 표현하는 방법

## 4장 : 일괄처리 계층의 데이터 저장소

- 마스터 데이터 집합을 저장하는 데 필요한 요구 사항
  - 불변성, 영원한 진실 -> 데이터는 오직 한 번만 기록됨 -> 새로운 데이터를 데이터 집합에 추가하는 것만이 유일한 쓰기 연산 -> 꾸준히 증가하는 대량의 데이터를 처리하는 데 최적화되어야 함
  - 데이터 집합에 대한 함수를 계산하여 일괄처리 뷰를 생성하는 역할 -> 한 번에 많은 데이터를 읽는 데 적합해야 함
- 적합한 솔루션 선택
  - 키/값 저장소
    - 일괄처리 시에는 키가 필요치 않음
    - 변경 가능한 저장소에 사용하도록 만들어짐
  - 분산파일 시스템
    - 파일은 디스크에 순차 저장
    - 불변성 강제 좋음
    - 보통의 파일시스템 단점은 오직 하나의 장비에만 존재, 클러스터링 된 파일 시스템 - 분산 파일 시스템
- 분산 파일 시스템 동작 방식(HDFS)
  - 네임노드 1개, 데이터 노드 n개
  - 파일은 확장성을 위해 여러 장비로 분산 저장되며, 이로써 병렬 처리도 가능해집니다.
  - 파일 블록은 내결함성을 위해 여러 노드로 복제됩니다.
- 분산 파일 시스템이 어째서 마스터 데이터 집합을 저장하는 데에 제격인지?
- 일괄처리 계층 저장소가 어떻게 분산 파일 시스템에 대응하는지

## 5장 : 일괄처리 계층의 데이터 저장소: 사례

- 