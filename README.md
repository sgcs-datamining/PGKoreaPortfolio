# [CSEG312] Data Mining project: Korean Stock Market Portfolio Manager
https://github.com/ZhengyaoJiang/PGPortfolio 를 본 프로젝트의 baseline으로 설정
따라서 위 코드와 논문에 대한 분석으로부터 프로젝트를 시작하려 한다.

프로젝트의 구조적 설계와 진행, 그리고 팀원간의 프로젝트 이해 공유를 위해,
앞으로 README에 프로젝트 전반적인 내용을 작성할 것임

## PGPortfolio의 구조

일단 PGPortfolio를 정상적으로 설치하였다는 가정 하에 진행한다.
### download data
`> python main.py --mode download_data`
정상적으로 실행 시, 약 180MB의 ~/database/data.db 파일이 생성됨 

### generate
이는 모델과 관련된 여러 하이퍼파라미터 값들을 정의한 config 파일을 생성하는데에 이용된다.
train시작하기 전에 반드시 필요한 과정으로 보면 된다.
`> python main.py --mode generate`

* 만약 과거에 train을 진행한 상황이라면, train_package 폴더에 이미 log 파일들이 존재할 것이다.
이 경우, train_package폴더를 삭제한 후 다시 진행하면 정상적으로 진행될것이다.

### Train 과정
무엇보다 train 과정을 명확하게 이해하는 것이 우선이다.
`> python main.py --mode train`
위 코드를 실행하면 train_package 폴더에 존재하는 모든 config 폴더에 대해 train이 진행된다.
(우리의 목표는 구조를 이해하고 한국 금융 시장에 적용하는 것이므로, config 폴더는 1개만 있어도 좋다.)
train 후 test까지 자동으로 진행된다.

PGPortfolio 의 train 절차를 넓은 틀에서 class와 method 관점에서 보면 다음과 같다.

**main.py** main에서 training.py의 train_all을 부르는 것으로 시작
**training.py**
* **train_all**
train_package 폴더 안의 모든 config에 대해 train 하는 함수
여러 프로세스를 생성하여 각각의 config에 대해 train을 병렬적으로 진행할 수 있도록 구현되어있음
각각의 프로세스는 train_one 메소드를 의미함

* **train_one**
TraderTrainer().train_net을 return하는 메소드
이 한 줄이 train의 모든 과정을 진행하게 한다.

**tradertrainger.py** class TraderTrainer
config로 부터 모델에 관한 여러 하이퍼파라미터를 기준으로 데이터 전처리부터, 뉴럴넷 구성, 학습까지의 모든 과정을 처리하는 class
이해해야 하는 중요한 과정은 다음과 같다.
1. self._matrix = DataMatrices.create_from_config(config)
DataMatrices는 db로부터 data matrix를 구축하는 class이다.
우리가 가진 한국 주식 데이터를 올바르게 변환할 수 있도록 수정해야 한다.

2. self._agent = NNAgent(config)
우리가 원하는 것은 매 timestamp에서, 우리가 가진 포트폴리오의 weight를 최적으로 설정하는 AI이다.
이 NNAgent로부터 생성된 self._agent가 바로 그 AI라고 보면 된다.
따라서 최종적으로 우리가 한국 시장에 맞게 모델을 변경해야하는 부분이 NNAgent class이다.

3. self.train_net()
config에 정의된 train step만큼 for문을 돌며 train을 진행하는 부분.
전체 프로그램에서 시간을 제일 많이 차지하고 진행되는 부분이다.
train 과정이 종료되면 self._evaluate로 test를 진행한다.
매 step별 진행되는 과정을 간단히 나타내면 다음과 같다.
```
x, y, last_w, setw = self.next_batch()
self._agent.train(, y, last_w, setw)
```
