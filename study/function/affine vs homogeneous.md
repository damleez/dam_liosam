AFFINE vs HOMOGENEOUS
===
## 0. Eigen의 geometry module 
### - detailed description
- fixed-size homogeneous transformations
- translation, scaling, 2D and 3D rotations
- 쿼터니언
- 외적(MatrixBase::cross, MatrixBase::cross3)
- 직교 벡터 생성(MatrixBase::unitOrthogonal)
- 일부 선형 구성요소: parametrized-lines and hyperplanes 
- axis aligned bounding boxes 
- least-square transformation fitting 
### - space transformations
- 2D 및 3D 회전과 projective 또는 affine transformation을 처리하기 위해 geometry module이 제공하는 많은 가능성을 소개
- Eigen의 Geometry module은 두 가지 다른 종류의 geometry transformation을 제공
1. Abstract transformation, rotation(각도와 축으로 표현 / 쿼터니안), translations, scalings
  - 이러한 변환은 행렬로 표현되지 않지만 표현식에서 행렬 및 벡터와 혼합하고 원하는 경우 행렬로 변환할 수 있음
2. Projective, affine transformation matrices
  - 변환 클래스를 참조
  - 이들은 실제로 행렬

## 1. Eigen::Affine
- 일반 Affine TF는 내부적으로 (Dim+1)^2 행렬인 Transform 클래스로 표현
- 여기서 마지막 행은 ```[0 ... 0 1]```로 가정
- 항상 저 linear은 가역행렬이 되어야 함
  - 가역행렬 : 역행렬이 존재하는 n×n 행렬 A를 가역행렬

![image](https://user-images.githubusercontent.com/108650199/213060672-7258eee1-e925-433c-8a58-b2ac6debc52c.png)

- In LIO-SAM, Eigen::Affine3f은 4x4행렬 float형이며,여기서 마지막 행은 ```[0 0 0 1]```

## 2. Homogeneous coordinate
- 포인트의 transformation과 벡터의 transformation을 한번에 표현하는 transformation matrix를 구하는 법
  - Affine3과 같은 모양으로 4x4 행렬 
- 차원의 좌표를 n+1개의 좌표로 나타내는 것으로 Affine과 동일
  - ```[R T | 0 1]```로 구성되며, 여기서 Rotation Matrix는 ![image](https://user-images.githubusercontent.com/108650199/213064438-70eec4cc-19c4-48ee-8348-55a8269eb015.png) 와 같은 조건을 만족해야함
  - 직교 행렬 Q는 반드시 Intervtible 해야 함
    - 💥️ 즉, 회전 행렬 R은 가역행렬이라고도 할 수 있음! 💥️
    - 💥️ 그렇기 때문에, Affine3에 한해 Homogeneous coordinate와 같다고 볼 수 있음 💥️
  
## 3. Affine TF vs Homogeneous coordinate

![image](https://user-images.githubusercontent.com/108650199/213062750-6539b092-b3d6-4677-8f77-cb883d0b1ecc.png)

