---
  title: "[Outlier Detection in High-dimensional data] (IF) Isolation Forest 리뷰"
categories: 
  - OutlierDetection
last_modified_at: 2020-09-21T13:00:00+09:00
toc: true
author_profile: true
use_math: true
---
  
  
  ## Paper
  [Isolation Forest](https://cs.nju.edu.cn/zhouzh/zhouzh.files/publication/icdm08b.pdf?q=isolation-forest)

---
  
  ## **Introduction**
  
  **Anomaly detection 이란?**
  
  Anomaly detection은 대다수의 정상 데이터들과 다른 양상을 보이는 희귀한 케이스를 탐지하는 걸 목표로 하는 Machine Learning의 연구분야 중 하나입니다. Anomaly detection이 사용될 수 있는 분야는 굉장히 다양한데, 대표적으로 금융회사에서 사기 고객을 판별하거나 (Fraud detection) 네트워크를 가지고 있는 포탈 회사에서 정상적이지 않은 접근을 탐지 (Network intrusion detection) 예들을 생각해 볼 수 있습니다.

**Anomaly detection 문제의 특성**
  
  Anomaly detection은 그 문제의 특성상 class의 imbalanced가 굉장히 심합니다. 극단적인 경우, 전체 데이터 셋의 99.97%가 정상 포인트이고 0.03%만이 anomaly인 경우에 이 0.03%를 정확히 분류해내야 하는 상황도 일어납니다. 또한 anomaly의 양상이 각기 다르게 나타날 수 있기 때문에 anomaly들을 하나의 정형화된 패턴으로 묶어서 분류하는 것 역시 어려운 문제가 됩니다. 더하여 많은 양의 데이터에 대해 labeling을 하는 것 자체로 많은 시간과 노력이 들어가기 때문에 일반적인 classification 문제처럼 supervised learning으로 접근하기보단 (label이 없는) unsupervised learning 방법으로 접근하는 경우가 많습니다. 

**Unsupervised learning based Anomaly detection**
  
  Unsupervised learning을 이용한 anomaly detection의 기존 방법들은 대부분 training 데이터를 이용해 observation들간의 distance나 density를 계산하여 정상 범주를 정의한 다음, test 데이터에서 그 값을 계산하여 정상 범주에 들어가지 않으면 anomaly라고 판단하는 식으로 작동했습니다.

그런데 여기엔 두가지 단점이 있습니다.(저자의 주장)

1.  모형이 normal 포인트에 대해서 최적화되어 있기 때문에 anomaly detection 성능은 떨어진다.

2.  density나 observation 계산은 computational cost가 높아서 large dataset이나 high-dimensional 데이터에는 적용하기 어렵다


Isolation Forest의 저자는 Isolation Forest가 위에서 언급한 두 가지 문제점을 극복할 수 있다고 주장합니다. 즉, anomaly detection 성능이 더 좋고 computational cost가 적다는 것이죠. 최종적으로 모형을 적합하는데 들어가는 time complexity가 선형적(linear)이다 라고 이야기합니다. time complexity가 선형적이라는 건 데이터 양이 늘어나도 적합 시간이 기하급수적으로 증가하는 게 아니라 늘어난 데이터 양에 비례하여 늘어난다는 것이죠.  즉, 데이터 양이 늘어남에 따라 들어가는 적합 시간이 이차함수 모양의 곡선이 아니라 일차 함수 형태의 선형 형태를 띤다는 것입니다. 그래서 large dataset이나 high-dimensional 데이터에도 충분히 적용할 수 있다고 말합니다.

**Assumtion(가정)**
  
  뒤에서는 Isolation Forest가 작동하는 방식과 그 핵심 논리를 알아볼 겁니다.

그 전에, Isolation Forest가 상정하는 anomaly에 대한 가정을 먼저 하고 넘어갈 필요가 있습니다.

앞으로 다룰 anomaly는 다음과 같은 특징을 가지고 있다고 생각할 겁니다

1.  **전체 데이터에서 anomaly가 차지하는 비율이 작다.**
  
  2.  **anomaly는 정상 데이터 포인트들과 다른 데이터 값을 가지고 있다. **
  
  
  즉, anomaly는 적고 다르다. 라고 생각할 수 있습니다. 

---
  
  ## **Basic Idea**
  
  Isolation Forest의 작동 원리를 이해하기 위해선 Decision Tree, Bagging에 선이해가 필요합니다. 관련 개념이나 용어가 생소한 분들은 Wikipedia를 참고하시기 바랍니다. ([https://en.wikipedia.org/wiki/Decision\_tree](https://en.wikipedia.org/wiki/Decision_tree), [https://en.wikipedia.org/wiki/Bootstrap\_aggregating](https://en.wikipedia.org/wiki/Bootstrap_aggregating))

**Isolation Forest의 작동 방식 in a nutshell**
  
  Isolation Forest는 anomaly detection을 하기 위해 Decision tree 구조를 이용합니다. 작동 과정을 간단하게 나타내면 다음과 같습니다. 

1.  Construct an unsupervised decision tree which randomly partition the feature space with binary split until every single instance is isolated. Call this tree as **iTree**
  
  2.  Build an **iForest** which is an ensemble of iTrees for sub-sampled (bootstrapped without replecement) data sets

3.  Calculate the avarage path length of each data point and get anomaly score by using some metric

4.  Identify some data points as anomalies which have high anomaly scores


정리하면, 어떠한 splitting rule에 구애받지 않는 unsupervised tree를 build up 하는데 이때 split을 멈추는 stopping rule을 모든 sub-region (terminal node)에 단 하나의 데이터 포인트들이 속할 때까지 라는 조건을 적용하여 특수한 unsupervised tree를 건설합니다. 그리고 이 tree를 **iTree**라고 부릅니다. 

원래 데이터셋에 대해 bootstrapping without replacement 를 하여 여러 개의 sub-sample 들을 만듭니다. 이 sub-sample들을 이용해 각각의 iTree를 sub-dataset 개수만큼 build up 합니다. 이렇게 만들어진 iTree들을 모두 모아 (ensemble) **iForest**를 도출합니다.

iForest에 있는 각각의 iTree를 이용해 i번째 데이터 포인트가 terminal node 까지 분류되는데 소요된 split의 개수 혹은 root node에서부터 terminal node까지의 거리 (path length)를 계산합니다. 각각의 iTree에 대해 계산한 path length를 average 하여 average path length를 얻고 뒤에서 정의할 anomaly score metric을 이용해 i번째 데이터 포인트의 anomaly score를 도출합니다. 

이 anomaly score가 높을수록 anomaly 일 가능성이 높다고 판단합니다. 

**작동 방식에 대한 즉각적인 의문**
  
  -   왜 모든 데이터 포인트가 isolation 될 때 까지 random split을 하는 iTree 구조를 사용하지?
  
  -   왜 bootstrapping을 이용해 여러개의 sub-sample들을 만든 다음 각각에 대해 iTree를 만들어 ensemble 한 iForest를 만들지?
  
  -   각각의 iTree에 대해 path length를 계산하는데 이게 어떤 의미가 있지?
  
  
  위 의문들에 대한 답으로 아래의 글을 작성하였습니다.

**왜 모든 데이터 포인트가 isolation 될 때 까지 random split을 하는 iTree 구조를 사용하지?**
  
  Isolation Forest의 기본적인 아이디어는 "anomaly가 normal 포인트 들보다 수가 적고 데이터 값이 다르다"는 가정에서 시작됩니다. 예를 들어, 변수 X들로 이루어진 feature space에 데이터 포인트들이 아래의 그림과 같이 놓여있다고 생각해보죠. 이때, 빨간색으로 표시된 $x\_{0}$는 anomaly 포인트 이고 파란색으로 표시된 $x\_{i}$는 normal 포인트 입니다. normal 포인트인 $x\_{i}$는 여러 데이터들과 가까이 있는 반면, anomaly인 $x\_{0}$는 다른 데이터들과 확연히 떨어져 있고 주변에 다른 데이터 포인트들이 많지 않은 걸 확인할 수 있습니다.

{% include figure image_path="/assets/images/OutlierDetection/IsolationForest/fig1.png" alt="this is a placeholder image" caption="Figure 1." %}

iTree는 위 관찰에 근거하여 고안되었습니다. 이 feature space를 어떠한 규칙 없이 random split을 통해 partition 하는데, 이 partition으로 생긴 모든 sub-region들이 단 하나의 데이터 포인트를 가지고 있을 때까지 반복하여 split을 진행하는 겁니다. 이때, 어떤 데이터 포인트가 단 하나의 region에 존재하게 되는 상황을 "isolation"이라고 부릅니다. 그랬을 때, 밀집하여 있는 normal 포인트 $x\_{i}$가 random partition을 통해 isolation 되는데 까지 걸린 split을 보여준 게 위의 그림 (a)입니다. isolation 되는데 총 12번의 split이 소요되었습니다. 반면 그림 (b)의 anomaly인 $x\_{0}$는 isolation 되는데 4번의 split 만이 소요되었습니다. 즉, anomaly일 때 isolation에 사용되는 split이 더 적은 것이죠. 

\=> anomaly 의 "few and different" 특징을 바탕으로 anomaly가 iTree에 의해 isolation 되는 데에 소요되는 split이 normal 포인트에 비해 작다는 것을 확인했다. 따라서 anomaly와 normal 포인트를 구별할 수 있는 measure로써 iTree의 isolation 되는 데 사용된 split 횟수 (path length)를 이용한다. 

**왜 bootstrapping을 이용해 여러개의 sub-sample들을 만든 다음 각각에 대해 iTree를 만들어 ensemble 한 iForest를 만들지?**
  
  그런데 이건 하나의 iTree에서 나온 결과에 불과하기 때문에 항상 이렇게 된다고 신뢰하기 어렵습니다. 그래서 여러번 시행했을 때 같은 데이터 포인트 $x\_{i}$, $x\_{0}$ 가 isolation 되는데 소요된 split 횟수를 확인하기 위해 bootstrapping 하여 여러 개의 sub-sample 들을 생성합니다. 이 생성된 sub-sample들에 대해 각각 iTree를 build up 하고 그때 $x\_{i}$와 $x\_{0}$가 isolation 되는데 소요된 split 횟수 (path length)의 평균을 나타낸 게 아래의 그림입니다.

{% include figure image_path="/assets/images/OutlierDetection/IsolationForest/fig2.png" alt="this is a placeholder image" caption=" " %}

위의 그림을 통해 normal 포인트 $x\_{i}$와 anomaly $x\_{0}$를 isolation 하는데 사용된 split 개수 (path length)가 iTree의 개수가 증가함에 따라 수렴한다는 걸 확인할 수 있다.

\=> sub-sample 갯수 (iTree의 개수)가 늘어남에 따라 average path length가 수렴하기 때문에 ensemble 하면 더 robust 한 모형 만들 수 있는 이점이 생긴다.

---
  
  ## **Methodology**
  
  앞선 Basic Idea 파트에서 Isolation Forest의 기본적인 작동방식과 핵심 논리에 대해 알아봤습니다. 이 장에선 구체적인 알고리즘에 대해 다룹니다. 

### **Notation**

-   $X=\\{x\_{1},\\cdots,x\_{n}\\}, \\quad x\_{i}\\in\\mathbb{R}^{d}$ for all $i=1,\\cdots,N$ 
  
  -   $T$: a node of an isolation tree

-   $h(x)$: path length of $x$ (iTree에서 isolation 되는데 사용된 split 횟수)

-   $E(h(x))$: average path length of $x$ ($x$가 각 iTree에서 isolation 되는 데 사용된 path length들의 평균)

-   $c(n)$: $h(x)$를 normalize 하기 위해 사용된 상수. (average path length of unsuccessful search in BST (Binary Search Tree)\[B. R. Presis, 1999\]).


$c(n) = 2H(n-1) - (2(n-1)/n)$,

where $H(i) \\approx ln(i) + 0.5772156649$
  
  -   s(x,n): anomaly score


$s(x,n) = 2^-{E(h(x))\\over c(n) }$
  
  When $E(h(x)) \\rightarrow c(n), \\quad s \\rightarrow 0.5$
  
  When $E(h(x)) \\rightarrow 0, \\quad s \\rightarrow 1$
  
  When $E(h(x)) \\rightarrow n-1, \\quad s \\rightarrow 0$
  
  \=> $s(x,n)$ 를 정의한 이유는 $E(h(x))$를 \[0,1\]로 bound 시키기 위함입니다..

$E(h(x))$가 클수록 anomaly score인 $s(x,n)$은 0에 가까워지고

$E(h(x))$가 작을수록 anomaly score인 $s(x,n)$은 1에 가까워지게 됩니다. 

이 값을 기준으로 모든 데이터 포인트들의 anomaly score를 계산하여 서로 비교할 수 있게 됩니다. 

### **Algorithm**

Isolation forest의 알고리즘은 2단계로 나눠서 진행됩니다.

1) Training stage

2) Evaluation stage

Training stage에선 input data $X$를 이용해 $t$개의 sub-sample과 iTree를 만들어서 iForest로 ensemble 해줍니다.

Evaluation stage에선 Training stage에서 생성한 $t$개의 iTree에 대해 모든 데이터 포인트 $x$의 path length를 계산합니다. 여기서 계산된 path length를 기반으로 각 데이터 포인트의 anomaly score를 도출합니다. 

**Training stage**
  
{% include figure image_path="/assets/images/OutlierDetection/IsolationForest/fig3.png" alt="this is a placeholder image" caption=" " %}

1\. iForest를 저장할 빈 객체 생성

2\. tree의 heigh limit 설정 

3\. for loop (sub-sampling 갯수 $t$ 회 반복)

4. sub-sampling (sub-samping size: $\\psi$)

5. iTree construction & iForest에 저장

6\. end for loop

7\. return iForest

{% include figure image_path="/assets/images/OutlierDetection/IsolationForest/fig4.png" alt="this is a placeholder image" caption=" " %}

1\. sampling 된 $X^{'}$이 isolaiton 된다면 exter node로 return
  
  4\. $X^{'}$의 변수들을 $Q$에 list로 저장

5\. 리스트 $Q$의 원소 중 하나 $q$ 를 랜덤하게 선택 

6\. split에 사용될 criterion 값 $p$를 랜덤 하게 선택. ($X^{'}$가 가진 변수 $q$의 범위 중에서)
  
  7\. $q<p$를 만족하는 $x$들을 $X\_{l}$로 할당
  
  8\. $q\\leq p$를 만족하는 $x$들을 $x\_{r}$로 할당
  
  9\. 모든 데이터 포인트가 isolation 될 때까지 iTree 반복. split 될 때마다 그 정보 $(q, p)$ 저장. 
  
  **Evaluation stage**
  
  {% include figure image_path="/assets/images/OutlierDetection/IsolationForest/fig5.png" alt="this is a placeholder image" caption=" " %}
  
  만들어진 iTree에 데이터 포인트 하나인 $x$를 흘려보낸다.
  
  1\. 현재 $x$가 있는 노드가 external node (terminal node) 일 경우, $e + c(T.size)$를 return
  
  $e$: 현재 path length
  
  $c(T.size)$ 현재 노드에 있는 데이터 포인트 갯수로 계속 split 했을 때 기대되는 path length. 이걸 더해주는 이유는 
  
  위의 Training stage에서 height limit에 걸려 tree split이 멈춘 경우 그 노드를 external node로 return 하는데, 그 노드 안에 1개가 아닌 여러 개의 데이터 포인트가 존재할 수 있다. split을 멈추지 않고 계속했을 때의 사전에 기대되는 path length를 $c(T.size)$로 계산해 더해준다.
  
  4\. 현재 노드 $T$에 저장되어 있는 split attribute를 $a$로 선언한다.
  
  5\. 만약 데이터 포인트 $x$의 $a$ 변수 값을 노드 $T$에 저장되어 있는 split value $q$ 와 비교한다.
  
  $x\_{a} < T.splitValue$ 일 경우 현재 노드 $T$의 왼쪽 노드인 $T.left$로 흘려보내고 다시 path length 알고리즘을 적용한다. 
  
  7\. $x\_{a} \\geq T.splitValue$ 일 경우, 현재 노드 $T$의 오른쪽 노드인 $T.right$로 $x$를 흘려보내고 다시 path length 알고리즘을 작동시킨다. 
  
  \=> 위의 Algorithm3를 통해 모든 데이터 포인트의 path length를 구할 수 있다. 이를 $t$ 개의 iTree에 적용한 후 average 하면 모든 데이터 포인트에 대해 average path length 인 $E(h(x))$를 얻을 수 있다. 이를 이용해 최종적으로 Anomaly score 인 $s(x,n)$을 얻는다. 
  
  이렇게 얻은 Anomaly score $s(x,n)$은 \[0,1\] 사이의 값을 보입니다. 이 값을 기준으로 데이터 포인트를 정렬하면 데이터 포인트들의 랭킹을 만들 수 있습니다. 이 랭킹을 기준으로 anomaly로 예상되는 데이터 포인트들을 추려낼 수 있습니다. 이 값에 대해 anomaly로 판별하는 어떠한 threshold 값을 지정할 수 있겠지만, 그건 각 데이터의 상태와 분석가의 판단에 의해 결정되는 요소입니다. 혹은 threshold를 설정하지 않고 랭킹이 높은 데이터 포인트들을 개별적으로 분석해서 anomaly인지 판단할 수 있습니다. 그래서 이렇게 anomaly score를 결과값을 주는 건 전체 데이터 중에서 anomaly로 의심되는 데이터 포인트들을 구별해내는데 드는 모니터링 비용이 줄어든다는 이점이 있습니다.
  
  ---
  
  ## **Conclusion**
  
  Isolation forest는 unsupervised learnin-based anomaly detection 알고리즘 입니다. 
  
  Isolation forest는 anomaly의 "few and different" 특징에 착안하여 anomaly와 normal 데이터 포인트를 구별하길 목표로 만들어진 알고리즘인 걸 확인했습니다.
  
  Isolation forest의 이점은 anomaly를 detect 하는 성능이 다른 모델들과 비교하여 더 좋을 뿐 아니라 모형 적합에 들어가는 time complexity가 효율적이라는 점입니다. (empirical result는 원 논문을 참고해주세요)
  
  **Future wokrs:**
  
  생각해볼 수 있는 발전 방향은 다음과 같습니다.
  
  -   Deal with categorical variable
  
  -   Develop an elaborated random partition method
  
  -   Develop a dimension reduction, either variable selection or variable extraction) method
  
  
  ---
  
  ## **References**
  
  \[1\] Liu, Fei Tony, Kai Ming Ting, and Zhi-Hua Zhou. ”Isolation forest.” In 2008 Eighth IEEE International Conference on Data Mining, pp. 413-422. IEEE, 2008.
  
  \[2\] Preiss, Bruno R. ”Data Structures and Algorithms.” (1999).
  
