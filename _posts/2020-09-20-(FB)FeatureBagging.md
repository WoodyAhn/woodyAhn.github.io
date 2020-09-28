---
title: "\[Outlier Detection in High-dimensional data\] (FB) Feature Bagging 리뷰"
categories: 
  - OutlierDetection
last_modified_at: 2020-09-20T13:00:00+09:00
toc: true
author_profile: true
---

## **Paper**

[Feature Bagging](https://dl.acm.org/doi/abs/10.1145/1081870.1081891?casa_token=fJmH04B3mzMAAAAA:Qpesfom07GOIXjeXBWxcIkHNmHrdFTDtJyjE1VXV3HmWQtyeeTKQlb2RE3GQa7ALJ-lFWnYzSPPu)

---

## **Introduction**

Feature Bagging은 supervised learning에서 널리 사용되는 Bagging 아이디어에 착안하여 High-Dimensional 데이터에서 Unsupervised learning 관점으로 Outlier를 탐지하는 새로운 방법을 제안합니다.

---

## **Motivation**

기존 outlier detection 방법론의 경우 데이터의 전체 변수 (full-dimension)를 모두 이용해 이상치를 탐지합니다. 하지만 이 방법을 High-dimensional 데이터에 그대로 적용할 경우 두가지 문제가 발생합니다.

첫째, 'Data sparsity'. \[2\]에 따르면 데이터의 object를 공간상에 표현할 때, 차원이 추가됨에 따라 object간의 거리가 멀어지고 결국 모든 object간의 거리가 비슷해집니다. 이는 차원의 저주 (curse of dimensionality)라고 불리기도 합니다. 이 특성이 outlier detection에 있어 문제가 되는 이유는 대부분의 outlier detection 방법론이 한 object가 다른 대다수의 object 혹은 집단과 떨어진 정도 (deviation)를 기준으로 outlierness를 계산하기 때문입니다. 차원이 매우 큰 High-dimensional 데이터의 경우 object간의 거리는 모두 비슷해져 outlierness 계산에 더이상 의미있는 정보로 사용될 수 없습니다.

둘째, 'Local feature relevance'. object의 outlierness는 전체 변수 중 일부 변수들의 조합에서 더 잘 나타나기도 합니다 (only the subset of attributes is useful for detecting anmalous behavior). 전체 변수 (full-dimension)에는 outlierness를 평가하는데 방해가 되는 irrelevant attributes가 포함되어 있는 경우가 있기 때문입니다. 예를들어, 아래의 Figure 1을 보면 object ●의 outlierness는 변수 {1,2}의 관점에서 더 잘 드러나는 반면, object ■의 경우엔 변수 {3,4}의 관점에서 더 잘 나타나는 것을 알 수 있습니다. 또한, {1,2}에 {3}을 추가하여 만든 {1,2,3}에선 {1,2}에서 관측되던 object ●의 outlierness가 더이상 보이지 않는 걸 확인 할 수 있습니다. 즉, 차원이 늘어남에 따라 더 낮은 차원에서 관측되던 outlierness가 오히려 가려지는 경우가 생기는 것입니다.

{% raw %}![alt]({{ woodyahn.github.io }}/assets/images/OutlierDetection/FeatureBagging/Example-Outliers-in-arbitrary-subspaces.png){% endraw %}

{% include figure image_path="/assets/images/OutlierDetection/FeatureBagging/Example-Outliers-in-arbitrary-subspaces.png" alt="this is a placeholder image" caption="This is a figure caption." %}

---

## **Basic Idea**


## **Reference**

\[1\] Lazarevic, A., & Kumar, V. (2005, August). Feature bagging for outlier detection. In _Proceedings of the eleventh ACM SIGKDD international conference on Knowledge discovery in data mining_ (pp. 157-166).

\[2\] Beyer, K., Goldstein, J., Ramakrishnan, R., & Shaft, U. (1999, January). When is “nearest neighbor” meaningful?. In _International conference on database theory_ (pp. 217-235). Springer, Berlin, Heidelberg.

\[3\] Müller, E., Schiffer, M., & Seidl, T. (2011, April). Statistical selection of relevant subspace projections for outlier ranking. In _2011 IEEE 27th international conference on data engineering_ (pp. 434-445). IEEE.

\[4\] Breunig, M. M., Kriegel, H. P., Ng, R. T., & Sander, J. (2000, May). LOF: identifying density-based local outliers. In _Proceedings of the 2000 ACM SIGMOD international conference on Management of data_ (pp. 93-104).
