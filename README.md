# Pupil-Reflex-AI

[관련 dataset](https://ieee-dataport.org/documents/independent-bilateral-eye-stimulation-gaze-pattern-recognition-based-steady-state-pupil
-> data set 

https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=DIKO0013697325
-> 관련 논문 device 동공반사 인지)



1. CNN(Convolutional Neural Network) 모델 선정 이유
CNN은 이미지 데이터 처리를 위해 설계된 모델로, 다음과 같은 특징 때문에 적합합니다:

특징 추출에 강점:

CNN은 이미지의 공간적 관계(예: 동공의 크기, 밝기 변화)를 효과적으로 학습합니다.
동공 반사 감지는 이미지 내 동공 주변의 작은 차이를 분석하는 작업으로, CNN이 이와 같은 패턴을 자동으로 추출하는 데 적합합니다.
파라미터 공유:

CNN의 필터는 이미지의 모든 영역에 적용되므로 학습해야 할 파라미터 수가 적습니다.
이는 모델의 메모리 요구 사항을 줄이고, 과적합 가능성을 낮춥니다.
지역적 연결성:

CNN은 이미지의 지역적 특성(예: 특정 부분의 반사 여부)을 분석하며 전역 특성을 결합합니다.
2. 각 레이어 구성과 파라미터 설정 근거
2.1 Conv2D
python
코드 복사
Conv2D(32, (3, 3), activation='relu', input_shape=(64, 64, 1))
32 Filters:
초기 필터 수는 적당히 작은 값(32)을 선택합니다.

너무 적으면 특징을 충분히 학습하지 못하고, 너무 많으면 학습 시간이 증가합니다.
동공 반사는 비교적 간단한 이미지 패턴을 포함하므로 32개의 필터가 적합합니다.
Kernel Size (3, 3):

작은 크기(3x3)는 세부적인 패턴을 학습하는 데 효과적입니다.
너무 큰 커널은 작은 변화를 포착하지 못할 가능성이 있습니다.
Activation Function (ReLU):

비선형성을 추가하여 필터가 다양한 특징을 학습할 수 있도록 합니다.
계산 효율이 높고, vanishing gradient 문제를 방지합니다.
2.2 MaxPooling2D
python
코드 복사
MaxPooling2D((2, 2))
Pooling Window Size (2, 2):

이미지 크기를 절반으로 줄이면서 중요한 특징을 유지합니다.
동공 반사 감지에서 이미지의 전체적 특징보다는 국소적 패턴이 중요하기 때문에 다운샘플링이 효과적입니다.
이유:

연산량 감소: 모델 학습 속도를 높이고 메모리 사용량을 줄임.
과적합 방지: 작은 변화를 제거해 학습 데이터에 과하게 적합하는 것을 방지.
2.3 Dropout
python
코드 복사
Dropout(0.25)
0.25 비율:
학습 시 무작위로 뉴런을 끄는 기법으로 과적합 방지.
0.25는 일반적인 시작값으로, 더 많은 학습 데이터가 있다면 낮출 수 있음.
반대로 데이터가 적거나 복잡하면 값을 높여야 함(0.4~0.5).
2.4 Dense Layer
python
코드 복사
Dense(128, activation='relu')
128 Units:

CNN 레이어에서 추출된 특징을 종합적으로 학습하기 위해 적절한 크기의 뉴런 개수를 설정.
동공 반사의 패턴은 상대적으로 단순하므로 복잡한 특징 학습이 필요하지 않아 128로 설정.
ReLU Activation:

이전 Conv 레이어와 동일하게, 비선형성을 유지.
python
코드 복사
Dense(1, activation='sigmoid')
1 Unit:
이진 분류(정상/비정상) 문제이므로 출력 뉴런 1개 사용.
sigmoid 활성화 함수는 출력값을 [0, 1]로 정규화하여 확률로 해석 가능.
3. 학습 파라미터 설정
3.1 Optimizer (Adam)
python
코드 복사
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])
Adam:

학습률을 자동으로 조정하며 SGD보다 빠르고 안정적입니다.
동공 반사 데이터는 작은 차이를 감지해야 하므로 효율적인 최적화가 중요합니다.
Learning Rate:

기본값(0.001)을 사용. 데이터가 더 복잡하면 낮은 값(0.0001)을 시도할 수 있습니다.
3.2 Loss Function
Binary Crossentropy:
이진 분류 문제에서 표준으로 사용되는 손실 함수.
모델이 출력한 확률(0~1)과 실제 레이블 간의 차이를 측정.
3.3 Epochs
python
코드 복사
history = model.fit(..., epochs=20)
20 Epochs:
데이터 크기가 크지 않거나 간단한 경우에는 적은 에포크로도 충분히 수렴.
Early Stopping을 추가해 과적합을 방지할 수 있음.
3.4 Batch Size
python
코드 복사
datagen.flow(X_train, y_train, batch_size=32)
Batch Size 32:
일반적으로 GPU 메모리 효율과 학습 속도 사이의 균형을 맞추기 위해 32 사용.
메모리 여유가 많으면 64로 증가 가능.
4. 데이터 증강
python
코드 복사
ImageDataGenerator(
    rotation_range=10,
    width_shift_range=0.1,
    height_shift_range=0.1,
    horizontal_flip=True
)
Rotation (10):

동공 반사가 약간 기울어진 경우도 학습할 수 있도록 보강.
Shift Range (0.1):

이미지 중심이 약간 변하더라도 모델이 이를 감지하도록 학습.
Horizontal Flip:

좌우 반전된 이미지에도 동공 반사를 감지할 수 있도록 함.
5. 조정이 필요한 요소 및 영향
이미지 크기 (img_size):

GPU 성능에 따라 128x128 이상의 크기로 조정 가능. 크기가 클수록 더 세부적인 특징을 학습하나, 메모리 소모 증가.
필터 개수:

모델이 학습을 과도하게 하거나 부족하게 한다면 첫 Conv2D 레이어의 필터 수를 16~64 범위에서 조정.
데이터 증강:

실제 데이터가 충분히 많으면 증강 강도를 줄일 수 있음.
반대로 데이터가 적다면 더 다양한 증강 기법(예: 밝기 조정)을 추가.
Dropout 비율:

학습 데이터 크기에 따라 0.1~0.5로 조정 가능.
학습률(Learning Rate):

학습 속도가 너무 빠르거나 느리면 0.0001~0.01 사이에서 조정.
이 모델은 간단한 이미지 분류를 처리할 수 있도록 설계되었으며, 데이터와 사용 환경에 맞게 각 요소를 세밀히 조정해야 최적의 성능을 얻을 수 있습니다.
