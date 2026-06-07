---
layout: post
title: SO-ARM101 프로젝트 일지 1 (HW - Teleop)
date: 2026-05-29 01:59:00
description: 오픈소스 로봇팔 SO-ARM101 하드웨어 세팅부터 텔레오퍼레이션까지 과정
tags: so-arm101
categories: studies
thumbnail: assets/img/so-arm101/Follower_Arm_assembly.png
related_publications: false
giscus_comments: true
---

### 1. 하드웨어 준비 및 3D 프린팅

해당 프로젝트에서는 다양한 모델(ACT, Diffusion, ...) 적용을 위해 오픈소스 로봇팔 **SO-ARM101**을 도입했습니다.

지출을 최소화 하기 위해 구동에 필수적인 서보 모터와 컨트롤 보드 키트만 구매하고, 나머지 기구부 파츠는 오픈소스 STL 파일을 활용해 직접 3D 프린터로 출력하여 셋업을 진행했습니다.

참고로 저희는 해외 배송을 하였는데 환율, 통관비 등을 나중에 따져봤을 때 국내에 프레임부터 모터까지 완제품으로 판매하는 것과 금액적으로 큰 차이가 없었던 것 같습니다.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Servo_Motor_kit.png" title="서보 모터 및 하드웨어 키트" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/3D_Slicer_setting.png" title="3D 프린터 슬라이서 세팅" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    (좌) 프로젝트를 위해 구매한 Feetech 서보 모터 및 부품 키트 / (우) 기구부 파츠 출력을 위한 슬라이서(Slicer) 세팅 및 STL 모델링
</div>

---

### 2. 기구부 조립 과정

오픈소스 STL 파일의 초기 세팅으로 출력을 진행했으나, 조립 과정에서 결합 응력을 버티지 못하고 손목 부분 모터 홀더에 크랙이 발생하는 이슈가 있었습니다.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Motor_holder_Wrist_crack.png" title="파손된 Wrist 모터 홀더" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    초기 세팅 출력 후 모터를 강하게 결합하는 과정에서 응력을 버티지 못하고 파손된 손목 조인트 파트
</div>

#### **해결 과정**

단순히 파츠를 다시 출력하는 것으로는 동일한 증상이 반복된다 판단하여, 슬라이서상에서 간단히 수행할 수 있는 방법들로 해결하기로 하였습니다.

- 내구도를 향상시키기 위해 출력시 **Infill 퍼센트를 상향** 조절했습니다.
- 조립 당시 부품간 여유가 너무나 부족하여 **수평 비율을 미세 상승**하여, 모터 홀더가 너무 꽉끼지 않게 결합되도록 했습니다.

---

### 3. 경화제 도포 및 손목 카메라 주문

물체를 안정적으로 집기 위해서는 그리퍼 부분의 마찰력이 중요하다고 생각하였습니다. 출력한 PLA 재질의 미끄러운 한계를 보완하기 위해 그리퍼 팁 부분에 실리콘 고무 경화제를 도포하여 마찰력을 올릴려는 시도를 하였습니다.

<div class="row justify-content-sm-center">
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Silicone_hardener.png" title="실리콘 고무 경화제" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Applying_silicone_gripper.png" title="실리콘 코팅 과정" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    (좌) 안정적인 조작을 위해 준비한 액상 실리콘 / (우) 그리퍼 부분에 두껍게 코팅하여 말리는 과정
</div>

또한, 프로젝트에 필요한 손목 카메라(Wrist Cam)를 주문하여 구비하였고, 이로써 리더암과 팔로우암의 초기 HW 구성을 마무리하였습니다.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Wrist_Cam_mounted.png" title="손목 카메라" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Leader_Arm_assembly.png" title="리더 암" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Follower_Arm_assembly.png" title="팔로우 암" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    (좌) 마운트에 결합될 In-hand 시야용 Wrist Cam / (중) Leader Arm / (우) Follower Arm
</div>

---

### 4. 소프트웨어 세팅 및 텔레오퍼레이션(Teleoperation) 검증

하드웨어 조립이 완료된 후, **LeRobot**를 활용한 제어 환경을 구성했습니다. 관절별 모터 ID를 순차적으로 맵핑하고, 최대 가동 범위를 기록하는 영점 조절(Calibration)을 마쳤습니다.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Motor_ID_mapping.png" title="모터 ID 맵핑" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Calibration_testing.png" title="캘리브레이션 테스트" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    터미널을 통한 모터 ID 맵핑 과정(좌)과 영점 캘리브레이션 후 텔레오퍼레이션 구동을 준비하는 단계(우)
</div>

#### **실행 결과**

리더암과 팔로우암이 정상적으로 연동되었는지 확인하기 위해 첫 텔레오퍼레이션 시연을 진행했습니다. 우려했던 지연이나 떨림 현상 없이, Follower Arm이 Leader Arm의 궤적을 실시간으로 부드럽게 추종하는 것을 확인했습니다.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include video.liquid path="assets/video/so-arm101/Teleoperation_demo.mp4" class="img-fluid rounded z-depth-1" autoplay=true muted=true loop=true%}
    </div>
</div>
<div class="caption">
    리더 암의 움직임을 지연 없이 실시간으로 매끄럽게 따라 하는 텔레오퍼레이션 첫 시연 성공 영상
</div>
