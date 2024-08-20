# ML 모델로 신분증 검증 이미지 분류 모델 적용해보기

최근 들어 IT 기업들은 인공지능(AI)과 머신러닝(ML) 기술을 다양한 분야에 적용하여 효율성을 높이고 사용자 경험을 개선하는 데 주력하고 있다. 특히, 신분증 검증과 같은 보안이 중요한 서비스에서는 이러한 기술들은 매우 유용하게 사용되고 있다. 이번 글에서는 토스뱅크가 ML 모델을 활용해 신분증 검증 시스템을 구축한 사례를 소개하고, 이를 바탕으로 적용해본 과정을 공유하려고 한다.

### 토스뱅크의 ML 모델 활용 사례

토스뱅크는 사용자가 제출한 신분증을 빠르고 정확하게 검증하기 위해 다양한 ML 모델을 도입했다고 한다. 초기에는 모든 신분증 검증을 사람이 직접 수행했지만, 고객 수가 증가함에 따라 검증에 소요되는 시간과 비용이 문제로 대두되었다. 이를 해결하기 위해 신분증의 진위 여부, 정보의 가독성, 얼굴 인식 등의 작업을 자동화하는 ML 모델을 개발하게 되었다고 한다.

이 시스템에서는 이미지 분류 모델과 객체 탐지 모델이 사용되어 신분증의 각 요소를 분석하고, 이를 통해 신분증이 유효한지 여부를 판단한다. 또한, 특정 임계값을 설정해 자동으로 신분증을 반려하거나 경고를 표시함으로써 수기 검증의 부담을 줄이고 있다.

### 실제 프로젝트 적용

토스뱅크의 이와 같은 접근법과 관련한 글을 읽고 간단한 신분증 검증 시스템을 구축해보았다. 이 프로젝트에서는 ResNet50 모델을 사용하여 이미지 분류 작업을 수행했다.

### (1) 환경 설정 및 데이터 준비

먼저, Python과 필요한 라이브러리를 설치하고, 학습에 사용할 신분증 이미지 데이터를 준비했다. 데이터는 공개된 신분증 이미지와 자체 생성한 위조 신분증 이미지로 구성했다.

### (2) 모델 선택 및 훈련

ResNet50 모델을 사전 학습된 가중치로 초기화한 후, 신분증 이미지 데이터셋을 사용해 모델을 훈련시켰다. 훈련 과정에서 학습률, 배치 크기 등의 하이퍼파라미터를 조정해 최적의 성능을 도출하였다.

```
from tensorflow.keras.applications import ResNet50
from tensorflow.keras.models import Model
from tensorflow.keras.layers import Dense, GlobalAveragePooling2D

base_model = ResNet50(weights='imagenet', include_top=False, input_shape=(224, 224, 3))
x = base_model.output
x = GlobalAveragePooling2D()(x)
x = Dense(1024, activation='relu')(x)
predictions = Dense(1, activation='sigmoid')(x)

model = Model(inputs=base_model.input, outputs=predictions)
model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])

```

### (3) 임계값 설정 및 모델 테스트

모델 훈련이 완료된 후, 모델의 출력 값에 대한 임계값을 설정하여 신분증의 유효성을 판단하는 기준을 세웠다. 이 과정에서 False Positive와 False Negative의 균형을 맞추기 위해 A/B 테스트를 수행했다.

```
def evaluate_model(test_generator, threshold):
    correct = 0
    total = 0
    for x, y in test_generator:
        preds = model.predict(x)
        preds = (preds > threshold).astype(int)
        correct += (preds == y).sum()
        total += len(y)
    accuracy = correct / total
    return accuracy

```

### (4) 웹 애플리케이션 개발 및 배포

훈련된 모델을 Flask 웹 애플리케이션에 통합하여, 사용자가 신분증 이미지를 업로드하면 자동으로 검증 결과를 제공하는 시스템을 구축했다. 이 시스템은 사용자의 신분증 이미지를 받아 모델로 처리한 후, 유효성 여부를 반환한다.

```
from flask import Flask, request, jsonify
import numpy as np
import cv2

app = Flask(__name__)
model = load_model('trained_model.h5')

@app.route('/predict', methods=['POST'])
def predict():
    img = cv2.imdecode(np.frombuffer(request.files['image'].read(), np.uint8), cv2.IMREAD_COLOR)
    img = cv2.resize(img, (224, 224))
    img = np.expand_dims(img, axis=0) / 255.0
    prediction = model.predict(img)[0][0]
    return jsonify({'prediction': 'rejected' if prediction > 0.5 else 'accepted'})

if __name__ == '__main__':
    app.run(debug=True)

```

이번 과정을 통해, IT 기업의 기술 블로그에서 소개된 내용을 바탕으로 실제로 적용해보는 경험을 할 수 있었다. 특히, ML 모델을 이용해 보안과 효율성을 높이는 방법을 학습하고, 이를 적용할 수 있는 능력을 키울 수 있었다. 이미지 분류 모델에 대해서는 작년 AI 경진대회에서 한번 접해본적이 있는데, 그때는 "이미지 분류"에 대한 이해는 거의 없이 데이터를 학습시키기 바빴었다. 하지만 이번 여름방학 동안 SK에서 진행한 AI data Academy 프로그램을 통해 실제 데이터로 CCTV 이미지 분류 모델을 개발해 보고 토스 뱅크의 글을 읽어보니 이미지 분류에 대한 개념이 더욱 명확하게 자리잡힌 것 같다.

앞으로도 이러한 경험을 바탕으로 다양한 분야에서 AI와 ML을 활용해볼 수 있도록 열심히 공부해야겠다!

참고한 글 : [https://toss.tech/article/secure-efficient-ai](https://toss.tech/article/secure-efficient-ai)

[토스뱅크가 AI로 보안과 효율도 챙기는 방법
은행 창구에서 본인확인을 위해 뭘 요구할까요? 바로, 신분증입니다. 비대면 은행인 토스뱅크는 ML 모델로 어떻게 신분증 검증 과정의 수기 작업을 최소화했는지 알려드릴게요.
toss.tech](https://toss.tech/article/secure-efficient-ai)