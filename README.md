A. 인쇄 전 즉시 수정 필요

인트로에 MobiCare 인섭 하이브리드

B. Q&A에서 나올 수 있는 질문 (미리 답변 준비 필요)

1. Gray zone 수치가 세 곳에서 서로 다릅니다 — 가장 눈에 띄는 불일치입니다:

System Flow 다이어그램: Grey Zone 0.5~0.55
Table 1 주석: Gray zone : 0.85 ~ 0.9
Table 2 주석: Gray zone : 0.1 ~ 0.3

교수님이 반드시 물어볼 것입니다: "어느 것이 실제 값입니까? 다이어그램에는 하나, 결과표에는 다른 두 개가 있는데, 실제 시스템은 어떤 값으로 동작합니까?" 만약 데이터셋마다 grid search로 다르게 조정된 값이라면 (아마 그럴 것 같습니다), 다이어그램에 짧은 설명을 추가하는 것이 좋습니다 — 예: "illustrative threshold; actual values tuned per dataset (see Table notes)". 그렇지 않으면 다이어그램이 "거짓" 정보를 보여주는 것처럼 보일 수 있습니다.

4. Acc thr : 0.0 (Table 1 주석) — 이전에 제가 말씀드린 기술적 세부사항이 이제 포스터에 그대로 노출되어 있습니다. acceleration threshold가 0이면, 이 조건은 사실상 거의 항상 통과됩니다 (필터링 효과가 없음). 예리한 교수님이라면: "threshold가 0이면, 사실상 velocity에만 의존하는 것 아닙니까? 그런데 왜 이걸 두 개 파라미터를 가진 'Kinematic Filter'로 소개합니까?" 라고 물을 수 있습니다. 답변을 준비해 두세요 — 예를 들어 "전체 데이터셋에서는 grid search 결과 velocity 신호만으로 충분하다는 것이 최적값으로 나왔다" 등.

5. 가장 중요한 방법론적 질문 — 17명 subject 평가가 subject-independent(LOSO)입니까? Table 1은 "17 Subjects - 19,645 Windows"라고 되어 있지만, 포스터 어디에도 이 train/test split이 subject 기준으로 분리되었는지(즉 테스트한 사람이 훈련 데이터에 전혀 포함되지 않았는지), 아니면 모든 subject를 섞어서 분할했는지(subject-dependent) 명시되어 있지 않습니다. 이 차이는 매우 중요합니다: subject-dependent라면, F1=0.972라는 수치는 실제로 한 번도 보지 못한 새로운 사람에게 얼마나 잘 작동하는지를 보여주지 않습니다 (모델이 이미 그 사람의 동작 패턴을 "학습"했기 때문입니다). 이것은 어떤 ML 학회에서든 가장 먼저 나오는 질문입니다. 아직 subject-independent split을 하지 않았다면, 이는 Limitations로 명확히 밝혀야 합니다 (현재 포스터에는 Limitations 섹션이 아예 없습니다 — 이것도 별도로 아래에서 다루겠습니다).
   
6. FPS(30.3)는 어디서 측정되었습니까 — Jetson입니까, 아니면 개발용 컴퓨터입니까? 제목에 "Real-Time"이라는 주장이 있지만, Comparison Table의 FPS 수치가 어떤 하드웨어에서 측정되었는지 나와 있지 않습니다. 만약 개발용 GPU/CPU에서(PyTorch, TensorRT 없이) 측정된 것이라면, 이는 Jetson에서의 실제 배포 속도가 아닙니다. 교수님: "이 FPS는 어떤 하드웨어입니까? Jetson에서 TensorRT를 사용해도 같은 수치가 나옵니까?"
   
7. "TCN+TE" — "TE"가 무엇을 의미합니까? 포스터 어디에도 풀어서 설명되어 있지 않습니다 (Temporal Embedding? Transformer Encoder?). 약어는 처음 사용할 때 전체 명칭을 함께 표기해야 합니다.
   
9. Privacy 주장과 Backend Server 사이의 모순. "Local Edge AI for Real-Time, Privacy-Preserving Processing"이라고 써놓았지만, System Flow 다이어그램에서는 낙상 클립이 Backend Server로 전송되어 "Files on Disk"에 저장됩니다. 교수님: "영상이 Backend Server로 전송된다면, 프라이버시 관점에서 클라우드 기반 시스템과 무엇이 다릅니까? Backend server가 가정 내부에(로컬 서버로) 위치합니까, 아니면 외부/클라우드 서버입니까?" 만약 backend가 실제로 가정 내부에서(Jetson 자체 또는 로컬 네트워크의 컴퓨터에서) 동작한다면, 이는 매우 강력한 답변이 되므로 명확히 밝혀야 합니다. 클라우드라면 "privacy-preserving"이라는 표현을 조정해야 할 수도 있습니다.

11. UP-Fall 데이터셋의 subject들이 실제로 노인입니까? 이 데이터셋은 일반적으로 젊고 건강한 사람들이 시뮬레이션한 낙상으로 구성되어 있습니다 (일반적으로 알려진 내용이며, 확인이 필요할 수 있지만 흔히 제기되는 지적입니다). 만약 그렇다면, 교수님: "subject들이 젊고 건강한 사람들입니까? 노인의 움직임 속도, 걸음걸이 패턴은 완전히 다른데, 이 결과가 실제 노인에게 얼마나 일반화됩니까?" — 이는 Future Work의 "real-world testing"으로 부분적으로 답변되지만, 그래도 질문이 나올 수 있습니다.

12. 흥미롭고 유용한 포인트 — Recall/FP trade-off 방향이 두 표에서 반대입니다. Table 1에서는 Hybrid의 recall이 감소(0.99→0.97), FN이 증가(5→11)했지만, FP는 크게 감소(31→12)했습니다. Table 2에서는 반대입니다: Hybrid의 recall이 증가(0.79→0.91), FN이 감소(16→7)했습니다. 이것은 사실 좋은 스토리입니다 (전체 데이터에서는 filter가 "정제(cleaning)" 역할을, 적은 데이터에서는 "구조(rescue)" 역할을 한다는 것), 하지만 안전성 관점에서 한 가지 질문이 나올 수 있습니다: "전체 데이터셋에서 FN이 증가한 것(낙상을 놓치는 것)은 FP가 감소하는 것(오경보)보다 안전 시스템 입장에서 더 위험하지 않습니까? 왜 Recall이 아닌 F1을 최적화했습니까?" 이는 깊이 있고 예리한 질문이지만, 동시에 여러분에게 강력한 답변 기회도 제공합니다 (예: "안전이 중요한(safety-critical) 시스템에서는 recall이 더 중요하므로, 향후에는 F1 대신 recall-weighted 최적화를 검토할 예정이다" — 이는 Future Work에도 멋진 추가 내용이 될 것입니다).
C. 제안사항 (요약)


Gray zone 수치를 통일하거나, 다이어그램에 "illustrative" 주석 추가
"TE"를 한 번은 전체 명칭으로 표기
FPS가 어떤 하드웨어에서 측정되었는지 명시 (표 하단에 짧은 주석 하나면 충분)
Backend server 위치에 대한 한 문장 (로컬이라면 명확히 밝히는 것이 privacy 주장을 강화함)
Limitations 섹션이 전혀 없음 — Conclusion 하단에 짧은 한 줄이라도 (예: "subject-independent evaluation and real elderly testing are ongoing") 추가하면, 이러한 개방성은 교수님들에게 (숨기는 것보다) 좋게 받아들여집니다
