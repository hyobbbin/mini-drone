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
  * 장축 길이에 따른 드론과 링 사이 거리 값 추출
  * 추출한 값들로 회귀 분석을 통해 드론 이동 거리 식 도출
  * 드론 이동시켜 링 통과

* ~~표식에 따른 드론~~
* 4단계 각도 조절
  * 각도 변동 범위가 30 ~ 60° 이내이므로 30°부터 5°씩 회전시키며 최적의 각도[^2] 탐색

## 알고리즘
## 소스 코드
**변수 선언**
```MATLAB
count = 0;                              % 전진 여부 확인 변수
center_point = [480,240];               % 사각형 중심점이 center_point와 일치해야 통과
centroid = zeros(size(center_point));   % 사각형 중심점
```
**ryze 객체 만들기 → 드론의 카메라에 연결 → 드론 이륙**
```MATLAB
drone = ryze();                         
cam = camera(drone);
takeoff(drone);
```
**1단계**
+ 링 중점 찾기
1. 파란색 RGB 설정
```MATLAB
frame = snapshot(cam);
r = frame(:,:,1);   detect_r = (r < 50);   
g = frame(:,:,2);   detect_g = (g > 10) & (g < 120);
b = frame(:,:,3);   detect_b = (b > 50) & (b < 190);
blueNemo = detect_r & detect_g & detect_b;
```
2. 파란색 사각형 중심 좌표 계산
```MATLAB
areaNemo = regionprops(blueNemo,'BoundingBox','Centroid','Area');   % 속성 측정; BoundingBox, Centroid, Area 값 추출
    areaCh = 0;
    for j = 1:length(areaNemo)
        boxCh = areaNemo(j).BoundingBox; 
        if(boxCh(3) == 960 || boxCh(4) == 720)  % 화면 전체를 사각형으로 인식하는 경우 예외 처리
            continue

        else
            if areaCh <= areaNemo(j).Area   % 가장 큰 영역일 때 속성 추출
                areaCh = areaNemo(j).Area;
                centroid = areaNemo(j).Centroid;
            end
        end
    end
```
3. center point와 사각형 중심 좌표와 차이를 이용해 드론 위치 조절
```MATLAB
dis = centroid - center_point;  % 사각형 중점과 center_point 차이

    % case 1
    if(abs(dis(1))<=35 && abs(dis(2))<=35)    % x 좌표 차이, y 좌표 차이가 35보다 작을 경우 center point 인식
        disp("Find Center Point!"); 
        count = 1;
   
    % case 2
    elseif(dis(2)<=0 && abs(dis(2))<=35 && abs(dis(1))>35)
        if(dis(1)<=0)
            disp("Move left");
            moveleft(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
        
        elseif(dis(1)>0)
            disp("Move right");
            moveright(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
        end    

     % case 3
     elseif(dis(2)<=0 && abs(dis(2))>35)
        if(dis(1)<=0 && abs(dis(1))>35)
            disp("Move left");
            moveleft(drone,'Distance',0.2,'Speed',1);
            disp("Move up");
            moveup(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
        
        elseif(dis(1)>0 && abs(dis(1))>35)
            disp("Move right");
            moveright(drone,'Distance',0.2,'Speed',1);
            disp("Move up");
            moveup(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
       
        elseif(dis(1)<=0 && abs(dis(1))<=35)
            disp("Move up");
            moveup(drone,'Distance',0.2,'Speed',1);
            pause(0.5);

        elseif(dis(1)>0 && abs(dis(1))<=35)
            disp("Move up");
            moveup(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
        end

    % case 4
    elseif(dis(2)>0 && abs(dis(2))<=35 && abs(dis(1))>35)
        if(dis(1)<=0)
            disp("Move left");
            moveleft(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
        
        elseif(dis(1)>0)
            disp("Move right");
            moveright(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
        end    

     % case 5
     elseif(dis(2)>0 && abs(dis(2))>35)
        if(dis(1)<=0 && abs(dis(1))>35)
            disp("Move left");
            moveleft(drone,'Distance',0.2,'Speed',1);
            disp("Move down");
            movedown(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
        
        elseif(dis(1)>0 && abs(dis(1))>35)
            disp("Move right");
            moveright(drone,'Distance',0.2,'Speed',1);
            disp("Move down");
            movedown(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
        
        elseif(dis(1)<=0 && abs(dis(1))<=35)
            disp("Move down");
            movedown(drone,'Distance',0.2,'Speed',1);
            pause(0.5);

        elseif(dis(1)>0 && abs(dis(1))<=35)
            disp("Move down");
            movedown(drone,'Distance',0.2,'Speed',1);
            pause(0.5);
        end
    end
```
+ 링 통과하기
1. 파란색 HSV 설정

















[^1]: 원이 아닌 사각형 중심 좌표를 계산한다. 왜? 2단계 링과 3단계 링의 거리가 가까울 때 2단계에서 원을 추출하려 하면 3단계 원까지 함께 인식되는 문제점이 발생한다. 이를 해결하고자 원이 아닌 파란색 사각형을 추출하는 방식을 이용한다.

[^2]: 드론이 천을 일직선으로 바라보는 각도가 최적의 각도이다. 이때 파란색 픽셀 수가 가장 많이 검출된다. 이를 이용해 드론 각도를 조절한다.
