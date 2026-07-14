---
title:  "[AI] 개발자의 딥러닝 입문기 #3"
excerpt: "PyTorch 기본 — 학습 루프 5줄이 전부다 (그리고 조심해야 할 것들)"

categories:
  - AI

tags:
  - [AI, DeepLearning, PyTorch, Numpy, Python]

toc: true
toc_sticky: true
toc_label: "[PyTorch 기본]"

date: 2026-07-14
last_modified_at: 2026-07-14
---

### 개발자의 딥러닝 입문기 시리즈

1. [입문기 인트로 — 학습 로드맵과 멘탈 모델](https://ymkmoon.github.io/AI-01-Intro/)

2. [개발 환경 준비와 numpy 기초](https://ymkmoon.github.io/AI-02-Env-Numpy/)

3. [PyTorch 기본 — 학습 루프 5줄이 전부(현재글)](https://ymkmoon.github.io/AI-03-PyTorch-Basics/)

4. CNN 과 오토인코더 — 라벨 없이 학습하기 (예정)

5. 군집·시각화와 입문기 회고 (예정)

---

## 이번에 배운 것

PyTorch 공식 튜토리얼 **"Learn the Basics"** 를 처음부터 끝까지 따라가며 챕터별로 코드를 남겼다. Tensor → Dataset/DataLoader → 모델(nn.Module) → 자동미분 → 학습 루프 → 저장/로드 순서인데, 끝나고 보니 결론은 하나였다.

> **딥러닝 학습의 메인 루프는 5줄이 전부다.** 나머지는 이 5줄에 데이터를 공급하고(전처리), 이 5줄의 결과를 읽는(평가) 작업이다.

```python
for X, y in dataloader:
    pred = model(X)              # 1. 예측
    loss = loss_fn(pred, y)      # 2. 얼마나 틀렸나 (loss)
    loss.backward()              # 3. 어느 방향으로 고칠지 계산 (자동미분)
    optimizer.step()             # 4. 파라미터를 그 방향으로 조금 이동
    optimizer.zero_grad()        # 5. 다음 배치를 위해 기울기 초기화
```

백엔드 개발자의 눈으로 보면 — 이건 그냥 **이벤트 루프**다. 배치(이벤트)가 들어오면 처리하고 상태(파라미터)를 갱신한다. 낯선 것은 루프 구조가 아니라 3번(backward)이 자동이라는 점, 그리고 5번을 빼먹으면 조용히 망가진다는 점이었다(아래 주의사항 참조).

---

## Tensor — numpy 와 거의 같다

2편에서 numpy 를 익혀둔 덕에 Tensor 는 금방 끝났다. API 가 거의 같고, 차이는 두 가지다.

```python
import torch
import numpy as np

data = np.array([[1, 2], [3, 4]])
t = torch.from_numpy(data)     # numpy ↔ tensor 왕복
print(t.shape)                 # torch.Size([2, 2]) — shape 감각 그대로

# 차이 1: device — 텐서는 CPU/GPU 어디에 올릴지 지정할 수 있다
device = "mps" if torch.backends.mps.is_available() else "cpu"
t = t.to(device)

# 차이 2: 자동미분 — requires_grad=True 면 연산 기록을 추적한다
```

**device** 는 "이 데이터를 어느 하드웨어 메모리에 두고 연산할 것인가"다. Apple Silicon 은 `mps`, NVIDIA 는 `cuda`. 단, 모델과 데이터가 **같은 device 에 있어야** 연산이 된다 — 한쪽만 옮기면 에러가 난다(이건 친절하게 에러라도 난다).

---

## Dataset & DataLoader — 커넥션 풀 같은 것

`Dataset` 은 "샘플 하나를 어떻게 읽는가"만 정의하는 인터페이스다. `__len__` 과 `__getitem__` 두 개만 구현하면 된다 — 자바로 치면 `List` 인터페이스 구현체, 파이썬으로 치면 시퀀스 프로토콜이다.

```python
from torch.utils.data import DataLoader
from torchvision import datasets
from torchvision.transforms import ToTensor

training_data = datasets.FashionMNIST(
    root="data", train=True, download=True, transform=ToTensor()
)
test_data = datasets.FashionMNIST(
    root="data", train=False, download=True, transform=ToTensor()
)

train_dataloader = DataLoader(training_data, batch_size=64)
test_dataloader = DataLoader(test_data, batch_size=64)
```

배치 묶기·셔플·병렬 로딩은 전부 `DataLoader` 가 해 준다. 샘플 하나 읽는 법만 알려주면 공급 파이프라인이 완성되는 구조 — 관심사 분리가 잘 되어 있다.

`transform=ToTensor()` 는 PIL 이미지(0~255)를 텐서(0.0~1.0)로 바꾼다. **2편에서 손으로 했던 `/255` 정규화가 여기 숨어 있다.**

---

## nn.Module — 모델 조립

FashionMNIST(28×28 흑백 의류 이미지 10종) 분류 모델이다. 레이어를 쌓고 `forward()` 에 데이터가 흐르는 길을 적는다.

```python
from torch import nn

class NeuralNetwork(nn.Module):
    def __init__(self):
        super().__init__()
        self.flatten = nn.Flatten()                  # (1, 28, 28) → (784,)
        self.linear_relu_stack = nn.Sequential(
            nn.Linear(28*28, 512),
            nn.ReLU(),
            nn.Linear(512, 512),
            nn.ReLU(),
            nn.Linear(512, 10),                      # 10개 클래스 점수(logits)
        )

    def forward(self, x):
        x = self.flatten(x)
        logits = self.linear_relu_stack(x)
        return logits

model = NeuralNetwork()
```

여기서도 결국 **shape 이 전부**다. 입력 `(배치, 1, 28, 28)` → flatten `(배치, 784)` → 레이어를 거쳐 `(배치, 10)`. 레이어 연결이 틀리면 shape 불일치 에러가 나는데, 이건 numpy 때와 달리 에러라도 내 줘서 고맙다.

---

## 학습 루프 완성 — epoch 마다 train/val 을 나란히

튜토리얼 원본은 100배치마다 그 순간의 loss 를 출력하는데, 이걸 **epoch 단위 평균**으로 고쳤다. 이유는 아래 주의사항 3번에.

```python
def train_loop(dataloader, model, loss_fn, optimizer):
    model.train()                          # 학습 모드
    total_loss = 0
    for X, y in dataloader:
        pred = model(X)
        loss = loss_fn(pred, y)
        loss.backward()
        optimizer.step()
        optimizer.zero_grad()
        total_loss += loss.item()
    return total_loss / len(dataloader)    # epoch 평균 train loss

def test_loop(dataloader, model, loss_fn):
    model.eval()                           # 평가 모드
    test_loss = 0
    with torch.no_grad():                  # 기울기 계산 끄기
        for X, y in dataloader:
            pred = model(X)
            test_loss += loss_fn(pred, y).item()
    return test_loss / len(dataloader)

loss_fn = nn.CrossEntropyLoss()            # 분류용 손실 함수
optimizer = torch.optim.SGD(model.parameters(), lr=1e-3)

for t in range(10):
    avg_train_loss = train_loop(train_dataloader, model, loss_fn, optimizer)
    avg_val_loss = test_loop(test_dataloader, model, loss_fn)
    print(f"Epoch {t+1}: train={avg_train_loss:.2f}, val={avg_val_loss:.2f}")

torch.save(model.state_dict(), "fashion_model.pth")
```

10 epoch 을 돌리면 두 loss 가 나란히 내려가는 것이 보인다. 이 **두 곡선의 관계**가 학습 상태 진단의 기본 도구다 — 같이 내려가면 정상, train 만 내려가면 위험 신호(주의사항 4번).

---

## 추론은 별도 스크립트로

학습이 끝난 모델을 쓰는 것(추론, inference)은 **학습과 분리된 프로세스**다. 실무에서 학습 잡과 서빙 코드가 분리되어 있듯, 연습도 별도 파일로 했다.

```python
# 08_inference.py — 학습 스크립트와 별개 파일

model = NeuralNetwork()                                  # 1. 같은 구조를 재정의
model.load_state_dict(torch.load("fashion_model.pth"))   # 2. 학습된 가중치 로드
model.eval()                                             # 3. 평가 모드 (필수!)

x, y = test_data[0][0], test_data[0][1]

with torch.no_grad():
    x = x.unsqueeze(0)               # (1, 28, 28) → (1, 1, 28, 28) 배치 차원 추가
    pred = model(x)
    predicted = classes[pred[0].argmax(0)]
    actual = classes[y]
    print(f'예측: "{predicted}", 정답: "{actual}"')
# 예측: "Ankle boot", 정답: "Ankle boot"
```

`unsqueeze(0)` 이 필요한 이유 — 모델은 항상 **배치 단위 입력**을 기대한다. 1장을 넣을 때도 "크기 1짜리 배치"로 만들어야 한다. 2편의 shape 감각이 여기서 또 쓰인다.

---

## ⚠️ 주의해야 할 것들

튜토리얼을 그대로 따라 해도 동작은 한다. 문제는 **왜 그렇게 하는지 모르고 넘어가면, 내 코드를 짤 때 조용히 틀린다**는 것. 이번 스테이지에서 정리한 함정들이다.

### 1. `model.eval()` 과 `torch.no_grad()` 는 다른 일을 한다

혼동하기 쉬운데, 역할이 완전히 다르다.

| | `model.eval()` | `torch.no_grad()` |
| :-- | :-- | :-- |
| 본질 | **행위의 변경** | **연산의 중단** |
| 하는 일 | 드롭아웃·배치 정규화를 평가 모드로 전환 | 자동미분 엔진을 꺼서 기울기 계산·연산 기록을 중단 |
| 생략하면 | 추론 결과가 매번 달라지거나 배치 크기에 영향받음 | 결과 값은 같지만 역전파용 연산 기록이 메모리에 쌓임 |

즉 **추론 시에는 둘 다 필요하다.** `eval()` 만 쓰면 메모리·연산이 낭비되고, `no_grad()` 만 쓰면 (드롭아웃이 있는 모델에서) 결과가 비일관해진다.

### 2. `zero_grad()` 를 빼먹으면 학습이 조용히 망가진다

PyTorch 는 `loss.backward()` 가 계산한 기울기를 **덮어쓰지 않고 기존 값에 더한다(accumulate)**. 매 반복 `zero_grad()` 로 초기화하지 않으면 이전 배치들의 기울기가 계속 누적되어, 옵티마이저가 현재 데이터가 아니라 **과거의 모든 오차가 뒤섞인 방향**으로 파라미터를 움직인다. 에러는 안 난다 — loss 가 이상하게 안 내려갈 뿐이다. 1편의 멘탈 모델 2번("틀려도 에러가 안 난다")의 두 번째 실례.

### 3. 배치 loss 스냅샷으로는 곡선을 읽을 수 없다

튜토리얼 원본처럼 "100배치마다 그 순간 loss" 를 출력하면 값이 요동쳐서 추세를 읽기 어렵다. **epoch 전체의 평균 train loss 와 val loss 를 나란히** 찍어야 두 곡선을 비교할 수 있고, 그래야 다음 항목(과적합)을 진단할 수 있다.

### 4. train 은 내려가는데 val 이 오르기 시작하면 — 과적합

모델이 데이터의 보편적 패턴을 넘어 **학습 데이터에만 있는 노이즈까지 암기**하기 시작한 신호다. 처음 보는 데이터에 대한 대처 능력(일반화 성능)은 오히려 떨어지고 있는 것. 대응은:

1. **조기 종료(Early Stopping)** — val loss 가 다시 오르기 시작하는 지점에서 중단, 그때의 가중치 사용
2. **드롭아웃** — 일부 노드를 무작위로 꺼서 특정 뉴런 의존을 방지
3. **가중치 규제(Weight Decay/L2)** — 옵티마이저에 `weight_decay` 추가
4. **데이터 증강** — 단, 2편에서 다뤘듯 **도메인 물리가 허용하는 축**만 (이미지라고 아무 축이나 뒤집으면 안 된다)

### 5. 저장은 `state_dict` 로 — 그리고 튜토리얼의 함정 하나

모델 저장은 두 방식이 있다. `torch.save(model, ...)` (모델 통째로)는 pickle 직렬화라 **로드하는 쪽에 클래스 정의가 있어야 하고** 리팩토링에 취약하다. **`torch.save(model.state_dict(), ...)` (가중치만)** 가 표준이다 — 대신 로드 시 같은 구조를 먼저 만들어야 한다(위 추론 스크립트의 1번).

그리고 소소한 함정: 튜토리얼 저장/로드 챕터를 그대로 실행하면 예제 모델이 **VGG16(사전학습 가중치 553MB)** 이라, 내 FashionMNIST 모델(2.7MB)과 무관한 **1.1GB 짜리 파일 두 개**가 생긴다. 따라 하고 나서 지우자.

### 6. test 를 val 처럼 쓰고 있다는 것을 인지할 것

튜토리얼은 편의상 test 셋을 매 epoch 평가에 쓴다. 입문 단계에선 괜찮지만 개념은 구분해야 한다 — **모델을 고르거나 학습을 멈추는 판단에 쓰는 순간 그 데이터는 val 이다.** test 는 모든 결정이 끝난 뒤 최종 성적표로 한 번만. 섞어 쓰면 성능이 낙관적으로 왜곡된다(데이터 누수의 일종).

### 7. seed 고정은 습관

같은 코드가 돌 때마다 다른 결과를 내는 것이 딥러닝의 기본값이다. `torch.manual_seed()`/`np.random.seed()` 고정은 "재현 가능한 실험"의 최소 조건 — 이번 연습에서는 생략했지만 다음 편(오토인코더 실습)부터는 기본으로 깐다.

---

## 마무리

| 배운 것 | 요지 |
| :-- | :-- |
| 학습 루프 | 예측→loss→backward→step→zero_grad, 5줄이 전부 |
| Dataset/DataLoader | 샘플 읽는 법만 정의하면 배치·셔플은 프레임워크가 |
| nn.Module | 레이어 조립 + forward. 여기서도 shape 이 전부 |
| eval vs no_grad | 행위의 변경 vs 연산의 중단 — 추론엔 둘 다 |
| 진단 도구 | epoch 별 train/val 곡선. train↓ val↑ = 과적합 |
| 저장/추론 | state_dict 저장, 별도 스크립트에서 로드·eval·추론 |

다음 편은 이번 시리즈의 핵심이다 — **CNN 과 오토인코더**. 합성곱이 이미지의 지역 패턴을 잡는 원리를 보고, "정답 라벨 없이" 학습하는 오토인코더를 직접 만들어 이상탐지의 원형을 확인한다.
