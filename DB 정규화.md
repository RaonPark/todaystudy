# DB 정규화

## 좋은 스키마와 나쁜 스키마

| cpuID | cpuName | cpuInfrastructure | cpuGeneration | cpuManufacturedDate | cpuFactory |
| --- | --- | --- | --- | --- | --- |
| i129K | i5-12900K | Intel | 12 gen | 2021.11. | America |
| i129F | i5-12900F | Intel | 12 gen | 2022.01 | America |
| i129 | i5-12900 | Intel | 12 gen | 2022.02 | America |
| a76X | Ryzen 5 7600X | AMD | 5 gen | 2022.08 | Singapore |
| a77X | Ryzen 5 7700X | AMD | 5 gen | 2022.09 | Singapore |
- 위와 같은 스키마가 있다고 하면 총 세 가지 이상(anomaly)을 가지고 있다.
    - 갱신 이상: 만약, cpuInfrastructure을 변경해야할 일이 있다면, 여러 레코드를 변경시켜야 한다.
    - 삭제 이상: 만약, 모든 Intel 제품이 단종된다면, 그 상품 뿐 아니라, Intel 이라는 cpu 생산회사의 정보도 사라지게 된다.
    - 입력 이상: 만약, 새로운 생산 회사가 등장한다고 하더라도, 생산하는 제품이 없다면, 데이터를 입력할 수 없게 된다.

## 함수 종속성

- 함수 종속성은 유효한 관계 인스턴스에 대한 제약으로, 키 개념의 일반화라고 볼 수 있다.
- 유효한 관계 인스턴스에서 a 속성 값이 같은 임의의 튜플 두 개에서 b의 속성 값이 항상 동일하면, a는 b를 함수적으로 결정한다고 한다.
- 위의 예시에서, cpuInfrastructure가 모두 Intel로 같지만, 그것이 cpuName을 결정하지는 않으므로, cpuInfrastructure은 cpuName을 함수적으로 결정하지 않는다.

### Trivial 함수 종속성

- 함수 종속성이 테이블의 모든 인스턴스에 대해 만족이 되는 경우가 있으며, 이런 함수 종속성은 trivial 하다고 한다.
- a가 b를 포함하는 슈퍼집합일때, a가 b를 함수적으로 결정하는 것은 어떤 테이블 인스턴스에 대해 항상 만족하며, 이 경우, 함수 종속성이 trivial 하다고 한다.
- 위의 예시에서, cpuName이 cpuName을 함수적으로 결정하며, 어떤 테이블 인스턴스에 대해서도 항상 만족하므로 함수 종속성이 trivial 하다.

### 함수 종속성의 Closure

- A → B를 만족하고, B → C를 만족하면, A → C를 유추할 수 있다. 주어진 함수 종속성 집합으로부터 유추할 수 있는 모든 함수 종속성을 가진 집합을 함수 종속성 폐포(Closure)라고 한다.

## 암스트롱의 공리(Armstrong’s axiom)

- 새로운 함수 종속성을 유추할 수 있는 추론 규칙(inference rule)이라고 한다.
    1. reflexivity : a가 b의 슈퍼 셋인 경우, a → b이다.
    2. augmentation : a → b이면, ca → cb이다.
    3. transivity : a → b 이고 b → c 이면 a → c이다.
- 암스트롱 공리는 건전하며(sound) 완전하다(complete). 따라서, 모든 유효한 함수 종속성을 암스트롱 공리로 생성할 수 있다. 하지만 찾기 쉽지는 않다.

### 추가 추론 공리

1. union : a → b 고 a → c 이면, a → bc
2. decomposition : a → bc 이면, a → b 이고 a → c 이다.
3. pseudotransivity : a → b 이고 cb → d이면, ac → d이다.
- a → b이면 ca → cb(augmentation)이므로, ac → d이다.

## 속성 폐포(Closure)

- 주어진 속성이 함수적으로 결정할 수 있는 모든 속성을 속성 폐포라고 한다.
- 예를 들어, R = {A, B, C, G, H, I} 라고 하고, (AG)+ = AG의 슈퍼 셋을 찾는다고 해보자.
- F = {A → B, A → C, CG → H, CG → I, B → H}
    - closure = AG
    - closure = ABCG (A→B, A →C)
    - closure = ABCGH (CG→H, AGBC는 CG의 슈퍼 셋임)
    - closure = ABCGHI (CG → I, AGBCH는 CG의 슈퍼 셋임)
- 따라서, AG는 super key 이다. AG로 ABCGHI 모두를 결정할 수 있다. 하지만 부분집합인 A나 G는 super key가 아니다.

## 정규 커버(Canonical Cover)

- 중복되는 함수 종속성을 삭제하고 최소한의 속성 혹은 함수 종속성을 가지는 집합을 canonical cover라고 한다.
- 예를 들어, R = (A, B, C) 라고 하고, F = {A → BC, B → C, A → B, AB → C} 라고 해보자.
    - A → BC는 불필요하다.
        - A → B, AB → C ⇒ AA → BC(pseudotransivity) = A → BC
    - AB → C는 불필요하다.
        - A → B, B → C ⇒ AB → B(augmentation), B → C ⇒ AB → C
- 따라서 정규 커버는 Fc = {A→B, B→C} 이다.

## 정규화

- 정규화에는 1, 2, 3, BCNF, 4, 5 정규형이 있다. 실질적으로 3정규형 및 BCNF가 많이 쓰이며 그 이상은 효율성이 없다.

### 제1정규형

- 속성 값으로 원자 값만 사용가능하다.
- list, bag, set 등은 원자 값이 아니다.

| 이름 | 차량 이름 |
| --- | --- |
| 김철수 | K5, 911 |
| 최영희 | SELTOS |
- 위의 표에서 김철수는 여러 개의 차를 가지고 있으므로, 제 1정규형을 만족하지 못한다.

| 이름 | 차량 이름 |
| --- | --- |
| 김철수 | K5 |
| 김철수 | 911 |
| 최영희 | SELTOS |
- 제 1 정규형을 지킨 테이블은 위와 같다.

### 제2정규형

- 주요 속성(prime attribute) : 주요 속성이란, 임의의 후보키에 속하는 속성을 의미한다.
- 비주요 속성(nonprime attribute) : 비주요 속성이란, 임의의 후보키에 속하지 않는 속성을 의미한다.
- 제2정규형은 제1정규형 중에서 모든 비주요 속성이 모든 후보 키에 의존적이어야 한다.
- 예를 들어 스키마 R = { A, B, C, D, E }에서, F = { A B → C, A → D, B → E } 라고 하고, A B가 주요 속성이라고 하면,
    - D는 A에 의해서는 결정되지만, B에 의해 결정되지 않는다. E는 B에 의해 결정되지만 A에 의해 결정되지 않는다.
- 모든 비주요 속성이 모든 후보 키에 의존적이지 않으므로 다음과 같이 스키마를 나눠야 제2정규형을 지킨다.
- R1 = { A, B, C }, R2 = { A, D }, R3 = { B, E }

| 이름 | 차량이름 | 구매처 | 가격 |
| --- | --- | --- | --- |
| 김철수 | K5 | KIA | 1000 |
| 이준희 | 911 | FERRARI | 3000 |
| 최영희 | SELTOS | KIA | 1500 |
| 안홍진 | MINI COOPER | BMW | 2000 |
- 위의 표에서 {구매처 차량이름 → 가격} 이고, {차량이름 → 이름} 이므로, 다음과 같이 분리하면 된다. (차량이름, 구매처가 복합 기본키)

| 구매처 | 차량이름 | 가격 |
| --- | --- | --- |
| KIA | K5 | 1000 |
| FERRARI | 911 | 3000 |
| KIA | SELTOS | 1500 |
| BMW | MINI COOPER | 2000 |

| 차량이름 | 이름 |
| --- | --- |
| K5 | 김철수 |
| 911 | 이준희 |
| SELTOS | 최영희 |
| MINI COOPER | 안홍진 |

### 제3정규형

- 제2정규형 중에서 비주요 속성이 모든 후보키에 이행적으로 의존적이 아니면 제3정규형이다.
- 모든 의미있는 함수 종속성 a → b 에서 a가 슈퍼 키거나 또는 b가 주요 속성이어야 한다.
- 이행적(transivity)란 a → b이고 b → c이면 a → c를 의미한다.
- 스키마 EMP_DEPT(SSN, Ename, Bdate, Addr, D#, Dname, Dmgr)을 생각해보자.
    - SSN은 이 스키마의 기본 키이다.
    - F = { SSN → Ename Bdate Addr D#, D# → Dname Dmgr }
    - 이 때, SSN → Dname Dmgr 이 가능하다.(이행적 종속)
    - 따라서 스키마 EMP_DEPT는 다음 두 개의 스키마로 나누어야 한다.
        - ED1(SSN, Ename, Bdate, Addr, D#)
        - ED2(D#, Dname, Dmgr)

| 게임이름 | 개발처 | 수익 | 나라 |
| --- | --- | --- | --- |
| MapleStory | Nexon | 10000 | Korea |
| MasterDuel | Konami | 5000 | Japan |
| Witcher 3 | CDPR | 3000 | Poland |
- 게임이름 → 수익 개발처, 개발처 → 나라, 게임이름 → 나라 가 가능하므로 이행적 종속이 존재한다.

| 게임이름 | 개발처 | 수익 |
| --- | --- | --- |
| MapleStory | Nexon | 10000 |
| MasterDuel | Konami | 5000 |
| Witcher 3 | CDPR | 3000 |

| 개발처 | 나라 |
| --- | --- |
| Nexon | Korea |
| Konami | Japan |
| Witcher 3 | Poland |
- 위와 같이 제3정규형을 지킨 테이블로 나눌 수 있다.

### BCNF

- Boyce/Codd Normal Form의 줄임말이다.
- 관계형 스키마가 모든 의미 있는 함수 종속성 a → b에서 a가 슈퍼 키이면 BCNF 정규형이 된다.
- MovieStudio(title, year, lenght, filmType, studioName, studioAddr)이라는 스키마를 생각해보자.
    - F = {title year → length filmType studioName, studioName → studioAddr}
    - 후보키는 (title, year) 라고 하면, 위 스키마는 제2정규형이다.
    - (title year) → studioName, studioName → studioAddr 이므로 transivity가 존재하므로 제3정규형은 아니다.
    - 따라서 다음과 같이 2개로 나누면 BCNF를 만족한다.
        - MS1(title, year, length, filmType, studioName)
        - MS2(studioName, studioAddr)
- 속성이 2개인 테이블은 모두 BCNF이다.
