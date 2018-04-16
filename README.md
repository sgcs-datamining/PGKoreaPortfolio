*This repo is started from https://github.com/ZhengyaoJiang/PGPortfolio (2018/04/08)*

# [CSEG312] Data Mining project: Korean Stock Market Portfolio Manager
https://github.com/ZhengyaoJiang/PGPortfolio 를 본 프로젝트의 baseline으로 설정,
따라서 위 코드와 논문에 대한 분석으로부터 프로젝트를 시작하려 한다.

프로젝트의 구조적 설계와 진행, 그리고 팀원간의 프로젝트 이해 공유를 위해,
앞으로 README에 프로젝트 전반적인 내용을 작성할 것임

## PGKoreaPortfolio의 구조
PGKoreaPortfolio는 PGPortfolio로 부터 변형해 나가고 있기 때문에,
https://github.com/weed-stock/PGPortfolio 의 README를 먼저 참고하기 바란다.

### download data
현재 download_data를 지원하지 않는다.
https://github.com/gyusu/Creon-Datareader 를 이용하면 asset별 가격 데이터를 다운로드할 수 있으며, 그 데이터로부터 가공하여 input data를 생성해야 한다.

sample data를 PGKoreaPortfolio/database/Data.db 에 올려두었다.

### generate
현재 generate을 지원하지 않는다.
하이퍼파라미터 튜닝보다 comprehensible한 모델을 만드는 것에 초점을 둔다.

### Train

`> python main.py --mode train`

위 코드를 실행하면 train_package 폴더에 존재하는 모든 config 폴더에 대해 train이 진행된다.


## 한국 증권 시장의 특이사항 (암호화폐 거래소와의 차이점)

### 데이터 불연속성
한국 증권 시장은 정규시간(09:00 ~ 15:30)만 있는 것이 아니라 동시호가, 시간외 종가, 시간외 단일가같은 특징이 있어서 데이터가 시간에 따라 동등한 의미를 갖지 않을 수 있다.
데이터를 단지 인덱스로 슬라이싱 했을 경우 문제가 발생될 수 있다.

### 거래 수수료
거래소(코스피) + 코스닥 종목만을 대상으로 한다.

매수 시: 거래소 수수료 = 0.0015% (2018/04/09 키움증권 기준)
매도 시: 거래소 수수료 + 세금(0.3%) = 0.3015%
[!키움증권]: https://www.kiwoom.com/kws/html/useinfo/chargemoney.html

=> 매수 / 매도를 평균내어 매 거래 마다 수수료 **0.15%** 발생한다고 가정한다.

