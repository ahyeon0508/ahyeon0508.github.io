---
layout: post
title:  "XGBoost: A Scalable Tree Boosting System"
date: 2022-02-06
categories: [부스트캠프 AI Tech, 논문]
tags: [부캠, 논문]
comments: true
---
해당 포스팅은 부스트캠프 AI Tech를 참여하며 멘토링 시간에 발표했던 논문을 바탕으로 작성하였습니다.

<b>📖 논문</b>  
[XGBoost: A Scalable Tree Boosting System](https://dl.acm.org/doi/pdf/10.1145/2939672.2939785)

<b>📆 발표일시</b>  
2022.02.04 (금)

<b>📈 발표자료</b>  
아래의 텍스트를 클릭해주세요!  
<b>[구글 드라이브(클릭)](https://drive.google.com/drive/folders/1Qfi4ZZ63E0amK_KvMOl9QFRYQMhW1q2H?usp=sharing)</b>

<br>

-----

<br>

<h2>1. XGBoost 논문을 선택해서 발표한 이유?</h2>
- XGBoost는 성능이 좋기에 Kaggle과 같은 데이터 분석 경진 대회에서 사용 빈도가 높고 많은 우승자들이 사용했음
- 과제, 공모전을 하면서 본 모델을 사용해보았는데 성능이 좋아 생각없이 사용한 경우가 꽤 있어 XGBoost 모델의 본질을 알아보고 싶어졌음
- 이에 논문을 통해 본 모델이 왜 성능 좋은지 알아보고자 함


<h2>2. 요약</h2>
<b>2-1. What is XGBoost?</b>  
- 기본적으로 GBM과 같이 Decision Tree의 앙상블 모형임
- GBM의 단점인 느린 수행 시간과 과적합을 보완하기 위해 나온 모델
- 분류와 회귀 영역에서 뛰어난 예측 성능을 발휘
- GBM 대비 빠른 수행 시간(병렬 처리)
- 과적합 규제 지원(Regularization)
- Early Stopping(조기 종료) 기능 제공
- 다양한 옵션을 제공해 Customizing이 용이
- 결측치를 내부적으로 처리해줌

<b>2-2. TREE BOOSTING IN A NUTSHELL</b>  
- Regularized Learning Objective

    ![image](https://user-images.githubusercontent.com/44939208/152690926-f213be5e-a0ef-4602-b0bd-ea3780aea01c.png)

- Gradient Tree Boosting

    ![image](https://user-images.githubusercontent.com/44939208/152690859-88513670-efed-4325-89c0-917e777cc8d9.png)

    ![image](https://user-images.githubusercontent.com/44939208/152690896-a1316d12-f418-4d0a-accd-d694482a4530.png)

    ![image](https://user-images.githubusercontent.com/44939208/152690964-aab7a299-1e35-4117-805a-850e8e1fad0e.png)

- Shrinkage and Column Subsampling  
    <b>Shrinkage</b>  
        - 부스팅 트리의 각 단계 이후마다 새롭게 추가된 가중치를 요인 𝜂로 Scaling 함  
        - Stochastic optimization에서 Learning Rate를 조정했던 것과 같이 각 트리의 개별적인 영향을 줄여줌  
        (Why? 나중에 만들어질 트리에게 기회를 주기 위해서)  
    <b>Column Subsampling</b>  
        - 랜덤포레스트처럼 칼럼을 다 뽑지 않음으로써 다양성을 부여하고 Overfitting을 방지함  
        - 병렬 알고리즘 계산 속도를 가속화함

<b>2-3. SPLIT FINDING ALGORITHMS</b>  
트리 학습의 핵심적인 문제인 가장 좋은 분할을 찾는 방법 4가지
- <b>Exact Greedy Algorithm(EGA)</b>  
    - 그리디 알고리즘처럼 Features에서 가능한 분할을 열거하는 알고리즘으로 항상 Optimal한 solution을 제공함 (효율적인 수행을 위해 Feature 값에 따라 데이터 정렬이 필요함)  
    - 하지만 데이터가 한꺼번에 로딩되지 않으면 알고리즘이 실행 불가하며 분산 환경에서도 수행할 수 없음


-  <b>Approximate Algorithm</b>  
    - EGA의 단점을 해결하기 위한 알고리즘
    - percentiles에 따라 후보 분할 지점을 제시함
    - 연속적인 Feature들을 이러한 후보 지점으로 나뉜 Bucket에 매핑하고 통계량을 집계한 다음 이중 가장 좋은 솔루션을 선택함
    - proposal이 주어진 시기에 따라 두 가지 버전으로 나뉘게 됨
    1. Global : 초기에 모든 후보 분할들을 제시하는 경우
    2. Local : 매번 분할이 진행된 후에 다시 제시되는 경우
    * 입실론이 같다면 global 보다는 local의 성능이 더 좋음
    * 입실론을 충분히 작게 해준다면 global 역시 EGA와 거의 같은 성능을 낼 수 있음


- <b>Weighted Quantile Sketch</b>  
    - weight의 합이 각 quantile에서 같으며, 이를 이용하여 weight의 합이 동일한 quantile로 구성된 것
    - 회귀 문제의 경우에는 weight가 모두 1이고 분류 문제의 경우는 weight가 확률에 따라 달라짐


- <b>Sparsity-aware Split Finding</b>  
    - XGBoost가 확장성 측면에서 우수하다는 것을 보여주는 이유
    - 실제 많은 문제들에서 입력이 Sparse 한 것은 흔함 => 이를 해결하기 위한 것
        - 데이터 내 결측치의 존재 유무
        - 통계량에서 빈번한 0 엔트리
        - 원핫인코딩 등
    - 각 트리 노드마다 Default Direction을 추가
    - 최적의 Default Direction은 데이터에서 학습됨
    - Basic Algorithm보다 속도가 빠름

<b>2-4. SYSTEM DESIGN</b>  
- Column Block for Parallel Learning
- Cash-aware access
- Blocks for Out-of-core Computation
=> 빠른 속도로 학습하는 등의 다양한 이점을 제공

<b>2-5. RELATED WROKS & END TO END EVALUATIONS</b>  
- Gradient Tree Boosting은 classification, learning to rank, structured prediction 등의 분야에서 우수함을 보임
- XGBoost는 오버피팅을 방지하기 위해 Regularized 모델을 채택함
- Random Forest에서 차용한 Column Sampling을 사용함
- Sparsity-aware Split Finding은 모든 종류의 희소성 패턴을 처리하기 위한 최초의 통합 접근법임
- Parallelizing tree learing이 가능함
- Cash-aware과 같은 기술을 제공함
- Python, R, Julia 등에서 사용 가능함
- 분산환경에서 작동할 수 있음
- 다양한 데이터셋을 통한 비교


<h2>3. 리뷰하고 느낀점</h2>  
원래부터 논문을 계속 읽어보고 싶었지만 영어와 수학의 결합으로 되어 있기에, 읽기를 기피했었다. 그러기를 몇년.. 이번에 부캠 멘토링 시간에 발표를 해야기에 책임감을 가지고 드디어 논문을 제대로 읽어보게 되었다! 완벽히 논문을 이해한 건 아니지만 열심히 읽어본 결과 아예 못 할 건 아니라는 것을 느꼈다. 물론 강의와 다양한 자료를 아~~~주 많이 참고했지만 덕분에 더 유익한 정보를 많이 얻을 수 있었던 것 같다. 이에 도전할 수 있는 힘이 길러진 것 같고, 영어와 수식에 대한 거부감이 조금은 사라진 것 같다. 또한 XGBoost 모델에 대한 이해도가 높아진 것 같아 XGBoost 모델에 대해 물어본다면 자신 있게 대답할 수 있을 것 같다고 느꼈다. 이번 세션을 기회로 쉬운 논문이라도 자주 논문 리뷰를 해보고자 한다.  

<b>틀리거나 어색한 부분이 있을 시에 댓글 혹은 메일로 알려주시면 감사하겠습니다 🙋‍♀️</b>