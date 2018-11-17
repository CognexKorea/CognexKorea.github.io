---
layout: post
title:  "Deep Learning Theory / Introduction to Aritificial Neural Networks Part 2"
date:   2018-11-16 23:25:00
author: Alex Choi
categories: Deep-Learning
---

[Part 1.](https://cognexkorea.github.io/deep-learning/2018/11/09/IntroductionToArtificialNeuralNetworks.html)에 이어 이번 글에서는 R에서 인공신경망(Artificial Neural Networks, ANN)을 이용하여 분류(Classification) 문제를 풀어보도록 하겠습니다.

------
## 데이터 획득
ANN을 이용하여 분류할 학습 데이터와 테스트 데이터가 필요합니다. 대표적인 Machine Learning 저장소인 [UCI Machine Learning Data Repository](http://archive.ics.uci.edu/ml/index.html)에서 데이터를 획득하도록 합니다.

이 중 [몽크 문제(Monk's Problem)](http://archive.ics.uci.edu/ml/datasets/MONK's+Problems) 데이터를 이용하기로 하겠습니다. 몽크 문제는 최초의 학습 알고리즘 비교를 위한 기초라고 할 수 있습니다.

먼저 몽크 문제의 학습 데이터를 다운받도록 합니다: [몽크 문제 학습 데이터](http://archive.ics.uci.edu/ml/machine-learning-databases/monks-problems/monks-1.train)

또한 몽크 문제의 테스트 데이터를 다운받도록 합니다: [몽크 문제 테스트 데이터](http://archive.ics.uci.edu/ml/machine-learning-databases/monks-problems/monks-1.test)

다운받은 파일들의 확장자를 모두 .csv로 변환합니다.
- [monk_train_1.csv](http://cinema4dr12.tistory.com/attachment/cfile29.uf@237B24445866529F16C02A.csv)
- [monk_test_1.csv](http://cinema4dr12.tistory.com/attachment/cfile5.uf@244C45445866529F3BF4D9.csv)

------
## 데이터 관찰 및 가공
이제 데이터를 불러오고 살펴보도록 하겠습니다

```
monk_train <- utils::read.csv(file = "monk_train_1.csv", sep = " ")
```

불러온 데이터를 몇 개 살펴보면,

```
> utils::head(monk_train)
   X X1 X1.1 X1.2 X1.3 X1.4 X3 X1.5  data_5
1 NA  1    1    1    1    1  3    2  data_6
2 NA  1    1    1    1    3  2    1 data_19
3 NA  1    1    1    1    3  3    2 data_22
4 NA  1    1    1    2    1  2    1 data_27
5 NA  1    1    1    2    1  2    2 data_28
6 NA  1    1    1    2    2  3    1 data_37
```

과 같은데, 각 피쳐(Feature)의 이름을 정해놓지 않았기 때문에 자동으로 생성되어 있음을 알 수 있습니다.

다음과 같이, 우선 X열을 삭제합니다.

```
monk_train$X <- NULL
```

다시 데이터 몇 개를 살펴보면,

```
> utils::head(monk_train)
  X1 X1.1 X1.2 X1.3 X1.4 X3 X1.5  data_5
1  1    1    1    1    1  3    2  data_6
2  1    1    1    1    3  2    1 data_19
3  1    1    1    1    3  3    2 data_22
4  1    1    1    2    1  2    1 data_27
5  1    1    1    2    1  2    2 data_28
6  1    1    1    2    2  3    1 data_37
```

첫번째 열이었던 X 피쳐가 삭제되었음을 확인할 수 있습니다.

이제 피쳐들의 이름을 지정하도록 합니다.

[UCI Machine Learning Repository의 몽크 문제 사이트](http://archive.ics.uci.edu/ml/datasets/MONK's+Problems)를 살펴보면, Attribute Information에서 각 피쳐의 이름을 알 수 있습니다:

```
Attribute Information:

1. class: 0, 1
2. a1: 1, 2, 3
3. a2: 1, 2, 3
4. a3: 1, 2
5. a4: 1, 2, 3
6. a5: 1, 2, 3, 4
7. a6: 1, 2
8. Id: (A unique symbol for each instance)
```

총 8개의 피쳐가 있고 이름은 위와 같습니다. 따라서, 위와 같이 피쳐의 이름을 부여하고,

```
base::names(monk_train) <- c("class","a1","a2","a3","a4","a5","a6","id")
```

다시 데이터를 몇 개 살펴보면,

```
> utils::head(monk_train)
  class a1 a2 a3 a4 a5 a6      id
1     1  1  1  1  1  3  2  data_6
2     1  1  1  1  3  2  1 data_19
3     1  1  1  1  3  3  2 data_22
4     1  1  1  2  1  2  1 data_27
5     1  1  1  2  1  2  2 data_28
6     1  1  1  2  2  3  1 data_37
```

이 중 사실 id는 필요가 없으므로, 다시 이 피쳐를 삭제하도록 합니다:

```
monk_train$id <- NULL
```

이제 id 피쳐가 삭제되었습니다:

```
> utils::head(monk_train)
  class a1 a2 a3 a4 a5 a6
1     1  1  1  1  1  3  2
2     1  1  1  1  3  2  1
3     1  1  1  1  3  3  2
4     1  1  1  2  1  2  1
5     1  1  1  2  1  2  2
6     1  1  1  2  2  3  1
```

이제 전체적으로 데이터를 살펴보면,

```
> utils::str(monk_train)
'data.frame':	123 obs. of  7 variables:
 $ class: int  1 1 1 1 1 1 1 1 1 0 ...
 $ a1   : int  1 1 1 1 1 1 1 1 1 1 ...
 $ a2   : int  1 1 1 1 1 1 1 1 2 2 ...
 $ a3   : int  1 1 1 2 2 2 2 2 1 1 ...
 $ a4   : int  1 3 3 1 1 2 2 3 1 1 ...
 $ a5   : int  3 2 3 2 2 3 4 1 1 2 ...
 $ a6   : int  2 1 2 1 2 1 1 2 2 1 ...
```

`monk_train`은 7개의 피쳐를 갖는 총 123개의 데이터로 구성되어 있음을 알 수 있습니다.

학습 데이터와 마찬가지로, 테스트 데이터 또한 동일한 방식으로 가공하도록 합니다:

```
monk_test <- utils::read.csv(file = "monk_test_1.csv", sep = " ")
monk_test$X <- NULL
base::names(monk_test) <- c("class","a1","a2","a3","a4","a5","a6","id")
monk_test$id <- NULL
utils::str(monk_test)
```

피쳐마다 값의 범위가 제각각이기 때문에 이를 정규화(Normalization)할 필요가 있으므로, 정규화 함수를 정의하도록 합니다.

정규화 함수는 벡터의 값에서 해당 벡터의 최소값을 뺀 값을 최대값과 최소값과의 차이로 나눈 것입니다:

```
normalize <- function(x) {
  return((x - min(x)) / (max(x) - min(x)))
}
```

따라서, 위의 함수를 이용하면, 입력된 값에 [0,1]의 범위로 변환됩니다.

`lapply` 함수를 이용하여 `monk_train`의 각 열에 `normalize()` 함수를 적용하고, 이를 Data Frame으로 변환합니다:

```
monk_train_norm <- base::as.data.frame(base::lapply(monk_train, normalize))
```

그러나, class 피쳐는 정규화 되지 않기를 원하기 때문에 이 피쳐에 대해서는 다시 원래 데이터를 저장하도록 합니다:
(사실 class 피쳐값들은 0 아니면 1이므로 굳이 데이터를 되돌릴 필요는 없지만, 몽크 문제가 아닌 다른 데이터인 경우에는 이 과정이 반드시 필요합니다.)

```
monk_train_norm$class <- monk_train$class
```

테스트 데이터에 대해서도 동일한 과정을 수행합니다:

```
monk_test_norm <- base::as.data.frame(base::lapply(monk_test, normalize))
monk_test_norm$class <- monk_test$class
```

------
## `neuralnet` 패키지 설치
이제 R의 인공신경망 패키지 중 하나인 `neuralnet`을 다운 받습니다:

```
> install.packages("neuralnet")
```

그리고 이 패키지를 로딩합니다:

```
base::library(neuralnet)
```

------
## ANN 모델 및 가시화
은닉 레이어를 1개만 갖는 간단한 ANN 모델을 세웁니다:

```
monk_model <- neuralnet::neuralnet(formula = class
                                   ~ a1 + a2 + a3 + a4 + a5 + a6,
                                   data = monk_train,
                                   hidden = 1)
```

이제 세워진 ANN 모델에 대한 네트워크 토폴로지를 가시화해 봅니다.

```
> graphics::plot(monk_model)
```

<img src="{{ site.baseurl }}/assets/posts/2018-11-16-IntroductionToArtificialNeuralNetworks2/01.png">

구성된 노드 및 레이어와 노드 가중치를 한 눈에 볼 수 있습니다.

상기 이미지의 파란색 노드는 편향(Bias)값을 나타내며, 이 값은 노드의 값이 이동하도록 하는 상수값인데, 마치 선형 회귀모델에서 좌표 Intercept와 유사합니다.

그러나, 에러값이 상당히 높으므로 이 모델로 예측할 경우 상당히 안 좋은 결과를 얻게 될 것이라는 것을 알 수 있습니다.

------
## ANN 모델을 이용한 테스트 데이터 예측
세워진 모델을 이용하여 테스트 데이터의 모델 결과를 얻어보도록 합니다.

```
model_results <- neuralnet::compute(monk_model, monk_test[2:7])
```

위의 코드의 의미는 `monk_model`을 이용하여 `monk_test[2:7]`에 대한 예측 모델 결과를 얻는 것입니다.

`monk_test[2:7]`는 `monk_test[1]`인 Class 피쳐에 대한 모델 결과를 얻기 위하여 나머지 피쳐인 a1~a6를 이용한다는 의미입니다.

`model_results`가 가지고 있는 데이터 이름를 살펴보면,

```
> names(model_results)
[1] "neurons"    "net.result"
```

인데, neurons는 뉴럴 네트워크의 각 레이어의 뉴런 출력값들이며, `net.result`는 뉴럴 네트워크의 전체 결과를 행렬 형태로 저장한 값(현재 우리가 풀고 있는 문제에서는 벡터)입니다.

`net.result`이 테스트 데이터에 대한 분류 결과입니다.

이 값을 예측 결과값이라는 다른 변수에 저장합니다.

```
predicted_monk <- model_results$net.result
```

값을 몇 개 살펴보면,

```
> head(predicted_monk)
             [,1]
[1,] 1.0102054821
[2,] 0.3010884442
[3,] 0.3014953569
[4,] 0.3010585095
[5,] 0.3010585096
[6,] 0.3010585095
```

첫번째 값을 볼 수 있듯이 테스트 데이터에서 class의 값은 0과 1사이였는데 1을 살짝 넘는 값도 보입니다.

즉, 계산이 충분히 정확하지 않음을 다시 한 번 확인할 수 있습니다.

그러면 실제 데이터의 class와  ANN을 통해 예측된 값의 오차를 확인해 보도록 합니다.

이를 확인하려면 두 값 사이의 의존성을 분석하면 되는데, 이것을 상관관계(Correlation) 분석이라고 하며 대표적인 것으로 피어슨(Pearson), 켄달(Kendall), 스피어만(Spearman) 상관계수 등이 있다.

테스트 데이터와 ANN을 통해 예측된 데이터와의 상관관계를 계산해 보도록 합니다:

```
> stats::cor(predicted_monk, monk_test$class)
             [,1]
[1,] 0.5479135178
```

상관관계 결과값은 [-1,1] 범위를 갖는데, -1은 음의 완전상관관계, 0은 상관관계가 하나도 없는 것이며, 1은 양의 완전상관과계입니다.

값이 0.5 근처로 두 값 사이에 상관관계는 있긴 하지만 크지 않음을 알 수 습니다.

즉, 계산 결과가 그리 만족스럽지 못하며, 실제 에러도 컸음을 알 수 있었습니다.

------
## ANN 모델 성능 개선
은닉 레이어를 단지 1개만 사용한 경우, 결과가 썩 만족스럽지 못했습니다.

은닉 레이어를 추가할 수록 해가 조금씩 개선될 것이지만 계산량이 매우 늘어날 것입니다.

계속해서 은닉 레이어를 추가하다보면 결과는 더이상 눈에 띄게 개선되지 않을 수 있는데 해가 수렴되었다고 볼 수 있을 것입니다.

또한 은닉 레이어를 너무 많이 추가하다보면 오히려 결과가 더 안 좋아지는 경우도 있습니다.

어쨌든 은닉 레이어를 2로 지정해서 결과를 확인해 보겠습니다.

```
set.seed(12345)
monk_model <- neuralnet::neuralnet(formula = class
                                   ~ a1 + a2 + a3 + a4 + a5 + a6,
                                   data = monk_train,
                                   hidden = 2)

graphics::plot(monk_model)
model_results <- neuralnet::compute(monk_model, monk_test[2:7])
predicted_monk <- model_results$net.result
```

<img src="{{ site.baseurl }}/assets/posts/2018-11-16-IntroductionToArtificialNeuralNetworks2/02.png">

이 때의 결과는,

```
> stats::cor(predicted_monk, monk_test$class)
             [,1]
[1,] 0.5604624596
```

이며, 은닉 레이어가 1개인 경우보다 결과가 약간 개선 되었음을 알 수 있습니다.

은닉 레이어를 5개까지 늘리면서 테스트 해 보면,

| No. of Hidden Layers 	| Errors 	| Steps 	| Correlation Coefficient 	|
|:--------------------:	|:------:	|:-----:	|:-----------------------:	|
|           1          	| 10.858 	|  2523 	|         0.54791         	|
|           2          	|  6.829 	|  3884 	|         0.50605         	|
|           3          	|  0.024 	|  4553 	|         0.99862         	|
|           4          	| 4.2716 	|  8082 	|         0.67534         	|
|           5          	|  0.018 	|  7760 	|         0.99712         	|

와 같은데, 은닉 레이어가 3개 사용된 경우 극적으로 결과가 좋아졌음을 알 수 있습니다.

그러나, 3개 사용된 경우에 비해 4개 사용되는 경우 계산량은 2개 가까이 많아졌는데 결과는 오히려 더 안 좋아졌으며, 5개 사용된 경우 결과는 다시 좋아졌음을 알 수 있습니다.

위의 이미지는 시험삼아 은닉 레이어를 10개 사용한 경우인데 눈이 돌아갈 정도로 복잡해 보입니다.

UCI Machine Learning Repository의 [몽크 문제(Monk's Problem)](http://archive.ics.uci.edu/ml/datasets/MONK's+Problems) 데이터에서 데이터셋 2와 3에 대해서도 연습삼아 위의 과정을 테스트해 보길 바랍니다.

이로써 R의 ANN 패키지인 `neuralnet`을 이용하여 분류 문제를 푸는 방법에 대해서 알아보았습니다.
