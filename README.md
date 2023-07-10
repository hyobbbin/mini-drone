# 대한전기학회 2023 미니드론 자율비행 경진대회
대한전기학회 2023 미니드론 자율비행 경진대회  
팀 국민대통합 기술워크샵
***
## 대회 진행 전략
* 맵 상세 규격
  * 고정 사항
    * 드론 이륙 지점과 1단계 링 사이 거리 : 1m
    * 1, 2, 3, 4단계 링 지름 : 78cm, 78cm, 57cm, 50cm
    * 1, 2, 3단계 링과 표식 사이 거리 : 2m
    * 4단계 링과 표식 사이 거리 : 1m
    * 드론 착륙 지점과 4단계 링 사이 거리 : 1m
   
  * 변동 사항
    * 링의 높이(바닥과 천 사이 간격) : 50 ~ 150cm 이내
    * 링의 좌우 : -2 ~ 2m 이내
    * 링의 각도 : 30 ~ 60° 이내

* 링 중점 찾기
  * 파란색 RGB 설정
  * center_point 설정
  * regionprops 함수로 파란색 사각형[^1] 중심 좌표 계산
  * center_point와 파란색 사각형 중심 좌표 상하좌우 차이를 이용하여 드론 위치 조절
  * 오차 범위 내에 드론 위치하면 중점으로 인식    

* 링 통과하기
  * 파란색 HSV 설정
  * 원 검출 후 regionprops 함수로 장축 길이 측정
  * 장축 길이에 따른 드론 이동 거리 계산 후 링 통과

* ~~표식에 따른 드론~~
* 4단계 각도 조절
  * 각도 변동 범위가 30 ~ 60° 이내이므로 30°부터 5°씩 회전시키며 최적의 각도[^2] 탐색

## 알고리즘
## 소스 코드
### 변수 선언

 count = 0;  % 전진 여부 확인 변수
 center_point = [480,240];   % 사각형 중심점이 center_point와 일치해야 통과
 centroid = zeros(size(center_point));   % 사각형 중심점



[^1]: 원이 아닌 사각형 중심 좌표를 계산한다. 왜? 2단계 링과 3단계 링의 거리가 가까울 때 2단계에서 원을 추출하려 하면 3단계 원까지 함께 인식되는 문제점이 발생한다. 이를 해결하고자 원이 아닌 파란색 사각형을 추출하는 방식을 이용한다.

[^2]: 드론이 천을 일직선으로 바라보는 각도가 최적의 각도이다. 이때 파란색 픽셀 수가 가장 많이 검출된다. 이를 이용해 드론 각도를 조절한다.
