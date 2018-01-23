---
title: Javascript Chart Source For Dashboard
category: Server
excerpt: |
  Javascript Chart Source For Dashboard
feature_text: |
  ## Javascript Chart Source For Dashboard
feature_image: "https://www.elastic.co/kr/assets/bltc02a4b0eadbc0ed1/kibana-timeseries.jpg"
image: "https://www.elastic.co/kr/assets/bltc02a4b0eadbc0ed1/kibana-timeseries.jpg"
---

### 잡소리...

새해를 맞아... 프로젝트 준비중이다..
정확히는 아직 연구 및 스터디 단계인듯...

이 업무를 아니..
회사를.. 계속 다닐 수 있을런지.. 모르겠지만..;;

요즘 Dashboard가 유행인듯하다.

검색 서비스가 활발해 짐에 따라 다양한 소프트웨어들이 많이 생겨났는데,
그 중 Elastic Search를 논하지 않을 수 없겠다.

이후 Elastic Search와 더불어 패키지 제품처럼 Logstash, Kibana라는 툴도 등장했다.

합쳐서..ELK<br/>
뭐..ELK는 누구나 다 검색해보면 정보를 알 수 있기 때문에 잡소리는 여기까지..

한동안은 어떻게 될지 모르겠지만,
맡아줄 업무를 요청받은 내용은 Kibana형태의 Dashboard를 만드는 일이다.

Server 개발만 주 업무로 해오다가,,
다시 Client 개발하려니... 머리가 아프다..
Javascript라니.....

여튼, 기술이 많이 발전되서..Javascript 영역의 기술도 많이 좋아진듯 하다.

### Javascript For Dashboard

Dashboard를 만드려면..우선 기반자료먼저 찾아보는게 좋다.

[Kibana](https://github.com/elastic/kibana)라는 녀석이 어떻게 구성되어있는지.
대략 어떤 기능이 있는지..

기능을 살펴보자..[Kibana Demo](https://demo.elastic.co/app/kibana#/dashboard/b7be4700-6837-11e7-bd1c-eb5e5ad48f8b)

2년 전에 봐왔던 화면과는 사뭇 다르다..
역시..외국인들이 덕후비중에는 더 많은 것 같다..
뭐 Dashboard이기 때문에 느린점도 있지만 예전보단..빠르다 빨라..

Kibana Dashboard에서 어떤 부분을 참고하면 좋을까..
[Kibana Feature](https://www.elastic.co/products/kibana) 주요 기능들도 많이 발전되서 좋아졌다.
그만큼 만들어질게 많아진듯...ㅠㅠ

괜히..IoT를 선택했다..
몇 년간은 국내에서 뜨지도 못할텐데...그냥..생각...

Dashboard 관련해서 찾아보니..
Github에서 누군가 Markdown으로 정리해놓은 것이 있었다.

[Awesome dashboard (and visualization)](https://github.com/obazoud/awesome-dashboard) 훈늉하다..

기본적으로 보았던 것들 위주로 정리해본다.

Time-Series Data 기반으로 Visualization을 할 수 있는 Grafana는 Kibana와 경쟁자인듯 싶다.
Kibana가 분석과 검색에 특화된 거라면..
Grafana는.. Visualization에 좀더 신경쓴 것 같다.
[Grafana Demo](http://play.grafana.org)

상세한 내용은 조금더 살펴봐야겠지만..
[InfluxDB, Telegraf, Grafana 를 활용한 Monitoring System 만들기(1)](http://www.popit.kr/influxdb_telegraf_grafana_1/)

이 글을 포스팅 하신분에게는 상당히 신선한 충격이었다고 말한다..
아름다움과 유연함.. 아마도 Editor의 기능때문일까..

[Freeboard](https://github.com/Freeboard/freeboard)는 IoT와 웹 기반의 Mashup 플랫폼을 위한 실시간 Dashboard이다.
Github에서 보면 알겠지만,, Demo나 Screen Shot에 표현된 것은 말 그대로 무엇을 위한 플랫폼인지를 명확히 보여준다..
하지만 너무 정형화되있다고 해야할까..

NodeJS, React, D3, Reflux 등을 같이 사용하여 만든 [Mozaik](https://github.com/plouc/mozaik)는 개발자스럽지만..
뭔가 귀여운 느낌도 넣은듯 하다..

기능이라고 하기엔.. 좀 많이 부족하지만 이런 Dashboard도 충분히 사용성이 있다는 것.
  - Scalable layout
  - Themes support
  - Extendable by modules
  - Grid positioning
  - Optimized backend communication
  - Rotation support (with smooth transition)

이외의 나머지.. Dashboard는 아직은 언급하기에는 기능이나 시각적으로 뚜렷하게 볼 수 있는 부분은 없었던 것 같다.

Dashboard의 마지막은 [Akveo](https://www.akveo.com/)를 소개한다.
Framework 형태로 제공하는 Akveo는 두가지의 Dashboard형태가 있다.

[Ng2-Admin](https://www.akveo.com/ng2-admin/)과 [Blur Admin](https://www.akveo.com/blur-admin/)
위의 것들과 소개한 것에 조금 차이를 두자면 Angular2 기반으로 만들어져있다.

UI도 Kibana에서 보았던 비슷한 몇몇 차트도 있는듯 하다.

Visualization 관련 Library는 크게 두 종류를 눈여겨 두고 있다.

[D3](https://d3js.org/) Chart쪽에서는 거의 모두 쓰다시피 할 정도로 이미 그 위력이 어마어마하다.
[D3 Demo](https://github.com/d3/d3/wiki/Gallery)를 보면 알겠지만 Chart이 외에도 다양한 부분에 활용할 수 있는 라이브러리다.

D3의 독보(?)적인 발전에 반기를 내세운 걸까..
Baidu에서 팀을 꾸렸나..

어디서 만들었나 싶었는데..Baidu였다.
[Echarts](https://github.com/ecomfe/echarts)도 다른 Visualization Library와 다르지 않을것 같았는데..
D3와 비슷하게 Interactive와 Visualization에 많은 신경을 쓴 것 같다.

[Echart Demo](https://ecomfe.github.io/echarts-examples/public/)

일단 조사하고 알아본 것은 여기까지..
정리라고 하기엔...그냥 개인적인 생각위주이기에.. 보시는 분은 그냥 이상한 글이구나..라고 생각해주시길..
