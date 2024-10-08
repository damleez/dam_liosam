## 1. dynamic avoidance
- 동적 회피(Dynamic Avoidance)는 주변 환경에서 동적으로 변하는 장애물이나 위험 요소를 감지하고 회피하는 기능을 의미
- 동적 회피 시스템은 센서 데이터를 분석하고 알고리즘을 활용하여 장애물을 피해 안전한 경로를 선택하거나 피할 수 있는 조치를 함
- 즉, local avoidance를 의미 
### - checking list
- 메모리 및 computing time 낮아야함
- 실시간 avoidance
  - 앞으로 우리는 GCS에서 처음위치와 최종목표만 찍어주면 NUC에서 실시간 회피를 하려고 함 
    - 왜냐하면 앞으로 GCS 무거워 질거라서, 또 dynamic 환경에 대처하기 위해  

## 2. new package related to avoidance
- move_base: ROS 네비게이션 스택에 포함된 move_base 패키지는 로봇의 경로 계획과 동적 회피를 담당
  - 로봇의 현재 위치와 목표 지점을 입력으로 받아 경로를 생성하고, 장애물이 발생할 경우 이를 회피하여 안전한 이동을 수행
- teb_local_planner: Trajectory Execution Framework(TebLocalPlanner)은 로봇의 로컬 경로 계획과 회피를 위한 패키지
  - 불확실한 환경에서도 빠르고 매끄러운 이동 경로를 생성하고, 동적 장애물 회피를 수행
- dwa_local_planner: Dynamic Window Approach(DWA) 로컬 플래너는 로봇의 동적 경로 계획을 담당하는 패키지로, 센서 정보를 기반으로 로봇의 위치와 주변 환경을 고려하여 회피 경로를 계산
- obstacle_detector: obstacle_detector 패키지는 센서 데이터를 활용하여 주변 장애물을 감지하고 표시하는 기능을 제공
- Rapid Exploring Random Tree


## 3. Dynamic Window Approach (DWA)
Dynamic Window Approach (DWA)는 샘플 기반 최적화를 수행합니다. 이는 실행 가능한 속도 공간에서 제어 동작을 샘플링하고, 해당 샘플된 동작에 대한 경로를 롤아웃합니다. 롤아웃이란, 지정된 모션 모델에 따라 특정 샘플링된 동작을 기반으로 예측된 경로를 시뮬레이션하는 것을 의미합니다. 중요한 점은, 제어 동작은 예측 기간 동안 일정하게 유지됩니다. 따라서 운동의 반전 등을 예측할 수 없습니다. 모든 샘플에 대해 예측을 롤아웃한 후, 지정된 비용 함수와 제약 조건(전역 경로까지의 거리, 부드러움, 장애물 제거 등)을 기반으로 가장 좋은 후보가 선택됩니다. 따라서 DWA는 계산 시간을 낮추면서 일정 수준의 제어 성능을 달성하기 위해 두 가지 단순화가 포함되어 있습니다. 안정성 및 재귀적 우려에 대한 자세한 내용은 다루지 않겠지만, 이 접근 방식은 차동 구동 및 전방향 로봇에 대해 잘 작동합니다. 격자 기반 평가를 위해 비용 함수가 매끄럽지 않아도 잘 맞습니다. 예를 들어, 경로를 격자지도에 래스터화하여 비용을 평가할 수 있습니다(치명적인 격자 셀 및 팽창 비용을 고려함). 또한, DWA는 초기화에 기반한 로컬 최소값에 갇히지 않습니다(물론, 예측 기간이 제한되고 제어 동작의 자유도가 적기 때문에 로봇은 여전히 갇힐 수 있습니다). DWA는 예측 기간 동안 제어 동작이 일정하다고 가정하므로, 차 유형의 로봇 제어는 제한적입니다. 차 유형 호환 가능한 속도 탐색 공간을 제한하는 몇 가지 기본적인 확장이 있습니다. 그러나 모션의 반전은 여전히 최적화의 대상이 아니며, 따라서 협소한 공간에서의 주차 동작은 다루기 어렵습니다. 물론, 폐쇄 루프 제어에서 모션의 반전이 발생할 수 있지만, 이는 개방형 루프 솔루션/예측의 일부가 아닙니다. 요약하면:

- 모션 반전이 없는 최적화의 부분해 (제어 동작은 예측 기간 동안 일정하게 유지됨)
- 차동 및 전방향 로봇에 적합하지만, 차 유형 로봇에는 적합하지 않음
- 매끄럽지 않은 비용 함수 지원

## 4. Timed-Elastic-Band (TEB)
타임드 엘라스틱 밴드 (TEB)는 주로 시간 최적 솔루션을 찾으려고 하지만 전역 참조 경로의 정확성에 대한 구성도 가능합니다. 이 접근 방식은 예측 기간 동안의 경로를 시간적으로 이산화하고 연속적인 수치 최적화 방법을 적용합니다. 따라서 이산화 해상도에 따라 예측 기간 내의 자유도가 매우 높아지며, 운동의 반전도 지원됩니다. 또한, 제약이 있는 최적화 문제를 제약 없는 최적화 문제로 변환하여 계산 시간을 줄입니다. 하지만 이는 제약 (예: 장애물 회피, 속도 제한 등)이 모든 경우에 보장되지 않는다는 것을 의미하므로, 로봇의 기본 드라이버나 전용 노드에서 비상 상황을 확인하는 것이 좋습니다. 또한, 옵티마이저는 지역해만 찾습니다. 예를 들어, 경로가 장애물의 왼쪽에서 초기화되면 거기에 남아 있습니다. teb_local_planner는 여러 토폴로지에서 동시에 여러 경로를 최적화하여 해결할 수 있습니다. 이 접근 방식은 연속적인 최적화에 의존하기 때문에 비용 함수는 매끄러워야 합니다. 그리드 및 비용 맵으로 함수를 평가할 수는 없습니다. 현재 치명적인 장애물 셀은 점 모양의 장애물로 간주되어 소형/중형 지역 비용 맵 크기 (그리고 상당히 그런 비용 맵 해상도)로 접근이 제한됩니다. 이 플래너는 다각형 모양의 장애물도 처리할 수 있으며 (costmap_converter 패키지를 사용하여 비용 맵을 더 기본적인 장애물로 변환함, 하지만 실험적인 기능입니다). 그러나 일정한 계산 성능과 적절한 문제 크기가 주어진 경우, 이 플래너는 훨씬 우수한 컨트롤러 성능을 달성하고 더 많은 시나리오를 해결하며 차 유형의 로봇 운동도 지원합니다.
최근 버전의 teb_local_planner에서는 동적 장애물 지원이 도입되었습니다. 성능은 장애물 추적 및 상태 추정의 정확성에 크게 의존합니다. 그러나 costmap-converter 패키지는 로컬 비용 맵에서 동적 장애물을 추적하려고 시도합니다 (실험적 기능).
요약하면:

- 계획된 경로는 실제 최적 솔루션에 더 가까우나 제약은 패널티로 구현됩니다.
- 모든 로봇 유형에 적합합니다.
- 여러 토폴로지에서 여러 경로 후보의 계획
- 동적 장애물 지원 (실험적 기능)
- 큰 계산 부하를 요구합니다.
