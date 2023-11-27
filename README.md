<img src= https://github.com/seonggegun/lolrecord/assets/79897862/8751e854-c04b-45da-8ba1-1769ff2c2371># lolrecord
# 1. 개 요
League Of Lengend(이하 LOL)는 전 세계에서 즐기는 대표적인 컴퓨터 게임이다. LOL은 e-sports 산업에서 대표적인 콘텐츠로 자리매김하고 있으며, 2022년 항저우 아시안 게임에서는 정식 종목으로 채택될 만큼 그 영향력은 이미 검증되어 있는 사실이다. 

LOL을 개발한 Riot Games는 비단 게임 자체의 재미뿐만 아니라 분석할 수 있는 데이터를 무료로 공개하고 있다. Riot Games API에서는 LOL 소환사의 개인적인 게임 정보와 더불어 경기 데이터까지 제공한다[1]. 통계적으로 하루 24시간 동안 수집되는 경기 수는 약 20만 건에 달하며, 소정의 절차를 거치면 손쉽게 얻을 수 있다[2].

이번 프로젝트에서는 20만 건의 하루치 LOL 게임 데이터를 분석하여 승패를 예측하는 모델을 만들어 보고자 한다.

# 2. 데이터
## 2.1 데이터 소스
이번 프로젝트에 활용할 데이터는 온라인 게임 코칭 전문기업인 더매치랩(The Match LAB)에서 가공한 LOL 게임 데이터를 바탕으로 한다. 데이터는 2023년 8월 25일, 9월 15일, 9월 17일 각각 하루 동안 수집된 LOL 경기에 대한 세부 항목들로 구성되어 있다.

## 2.2 탐색적 데이터 분석
티어에 대한 정보 (표)
  <img src = https://github.com/seonggegun/lolrecord/assets/79897862/78f03c88-4312-4c3c-bf6d-5e58524bc7ae>
  <img src = https://github.com/seonggegun/lolrecord/assets/79897862/9f548192-65b7-4860-abe1-c9c2ab69f151><br>
<table>
  <tr><th>티어</th><th>단계</th><th>분포</th></tr>
  <tr><th rowspan='1'>챌린쳐 (21,281)</th><th>I</th><td>0.0018</td></tr>
  <tr><th rowspan='3'>부정 (1637)</th><th>3</th><td>1,045</td></tr>
  <tr><th>2</th><td>291</td></tr>
  <tr><th>1</th><td>301</td></tr>
  <tr><th rowspan='1'>제외 (3)</th><th>0</th><td>3(블라인드조치된 언행)</td></tr>
  <tr><th colspan='2'>계</th><td>22921</td></tr>
</table> <br>
  티어는 아이언부터 다이아까지 IV~I 단계로 구분되며 그이상 마스터부터는 랭킹순으로 배치되며 휴면강등이란것이 있어 계속 게임을하여 등급을 유지해야한다.
<hr>
데이터는 5대5 솔로 랭크 경기 약 20만 건으로 구성되어 있으며, 포지션별로 데이터가 구분되어 있다. 5가지 포지션에 대한 내용은 다음과 같다.

<img src = https://github.com/seonggegun/lolrecord/assets/79897862/0d6dc375-4a51-4b5e-9180-fe89ae0e529a>

| 포지션    |위치|역할|
|--------|---|---|
| 탑(TOP) |지도의 윗부분에 위치한 라인|탱커, 딜러 |
| 정글(JUNGLE) |지도의 중앙에 위치하여 모든 라인을확인|cc기 및 이동기좋은 딜러|
| 미드(MID) |지도의 중간에 위치한 라인| 딜러|
| 원딜(ADC) |지도의 아래에 위치한 라인| 원거리형 딜러|
| 서폿(SUP) |지도의 아래에 위치한 라인| 원딜 보조형|

※단 역할은 보통 쓰는거에 대한 설명이지 확실히 정해져있는게 아니다. ex) 딜러형 서폿 <br>
제공된 데이터의 항목은 총 185개로 구성되어 있으며, 전반적으로 다음과 같은 내용으로 정리해볼 수 있다.

| 항목           | 데이터 속성 |
|--------------|--------|
| 플레이어에 대한 데이터 |kda, cs, largestmultikill, xp ... |
| 팀에 대한 데이터    | baron, dragon, turretkills, goldearned ... |
|라인전 전후에대한 데이터 |assist_at14, cs_at14, damagedone_at14 ...|

여기서 중요하게 고려되는 항목들은 다음과 같다. (10개 이상)

| 데이터 속성 | 데이터의 의미     | 중요한 이유 |
|-------|-------------|--------|
| KDA   | Kill/Death/Assist의 비율|실력을 나타내는 일반적인 지표. 데미지 지표, 팀 킬 비율과 같이 보면 전투 스타일 및 더 정확한 실력을 파악가능|
| DPD |데스 당 가한 데미지  챔피언에게 넣은 데미지 / 데스 수(데스가 0일경우 챔피언에게 넣은 데미지)|낮은 데스를 기록해 높게 나온다 - 안정적인 성향의 유저, 높은 데스를 기록해 높게 나온다 - 캐리력이 있는 유저|
| CSPM | 분 당 획득 CS |챔피언을 제외한 협곡의 모든 개체들을 처치한 횟수|
|Diffgold|Gold 격차|골드 격차는 라인전에서 우세하거나 많은 킬을 얻었음을 알수있고 상위 아이템을 빠르게 구비해 더 큰 전투 유리성을 얻을 수 있음을 나타낸다.|
|Goldearned100|블루팀 총 획득 골드|팀원이 더욱 유리한 상황을 이끌어갔음을 알수있다.|
|Goldearned200|레드팀 총 획득 골드|"|
|Dragon|드래곤 처치 수|드래곤 처치로 얻는 버프는 게임에서 유리한 상황을 조성하며, 팀 간의 경쟁에서 드래곤을 성공적으로 먹는 것은 전체적인 게임 흐름에서 우위를 차지했음을 알수있다.|
|Baron|내셔남작 처치 수|내셔남작 처치로 얻는 버프는 게임에서 유리한 상황을 조성하며, 팀 간의 경쟁에서 드래곤을 성공적으로 먹는 것은 전체적인 게임 흐름에서 우위를 차지했음을 알수있다.|
|Visionscore100|블루팀 시야점수|적의 진형에 더 많이 침입하여 상대의 행동 및 위치를 파악함으로써, 전투 상황에서 상대를 예측하고 유리한 전략을 세울 수 있다.|
|Visionscore200|레드팀 시야점수|"|

## 2.3 데이터 전처리
20만 건의 경기 데이터는 수준 별로 편차가 어느정도 존재할 것으로 예상된다. 따라서 일정 수준 이상의 데이터로 지표의 상관성을 통해 승패예측 모델을 만드는 것이 합리적일 것이다.

이에 이번 프로젝트에서는 다음과 같이 정의되는 LOL게임의 등급체계(tier)를 바탕으로, 모든 플레이어가 플래티넘 이상인 경기를 추출하여 분석해보고자 한다.

* 티어에 대한 정보 (표)
* 티어별 히스토그램
* 플래티넘 이상인 경기의 수

도출된 플래티넘 이상의 경기에 대해서, 핵심 데이터 속성으로 ???, ???, ??? 등을 추출했다. 그 이유는 ~~~때문이다. 그리고 승패를 예측하는 것이기 때문에, 경기 데이터를 기준으로 데이터 속성별 정규화(normalize)를 수행했다. 

## 2.4 데이터 프레임 설계

탐색적 데이터 분석과 데이터 전처리를 통해 다음과 같은 데이터 프레임을 만들고자 한다.

| Id  | 팀  | 주요지표 | TOP |MID|
|-----|-----|---------|-----|---|
| 고유번호 | Red/Blue | kda, dpd, ...| ... |...|

# 3. 승패예측 모델

## 3.1 모델 개요
CNN을 썼다. 단순한 합산을 썼다.

## 3.2 성능
플래티넘 이상 4.5만건
데이터 전체 20만건

## 3.3 소결
* 성능에 대한 의미
* 핵심 데이터 항목에 대한 추정 및 분석

# 4. 결론 및 배운점
