# RPA 툴, UiPath를 이용한 네이버 기사 크롤링 실습

🧐 **조코딩의 UiPath 실습 영상을 보고 진행했다.**

> UiPath Studio 앱을 활용하여 “네이버에서 삼성전자 관련 기사들을 크롤링”하는 실습을 진행했다.
> 

💡 **영상이 나왔을 때보다 업데이트 된 부분이 있어 영상과 조금 다를 수 있다.  
     (classic version을 사용하면 영상과 동일한 실습이 가능하다.)**

## FlowChart를 통해 워크 플로우를 형성하라

![image](https://user-images.githubusercontent.com/73332608/200778434-841160cb-7012-4903-8f69-3e5fbacbeaf6.png)

## Open Browser을 통해 새 브라우저를 열어라

![image](https://user-images.githubusercontent.com/73332608/200778467-d97e6790-321f-4581-9fdc-c688e017783c.png)

## 레코딩 기능을 통해 다수의 액티비티를 녹화해서 등록해라

![image](https://user-images.githubusercontent.com/73332608/200778524-c22b9c6a-08a6-4b7c-846d-2d5e46e8d33b.png)

![image](https://user-images.githubusercontent.com/73332608/200778567-0e2d7312-1819-4e15-8818-f12ece5e979a.png)

## 데이터 스크래핑 기능을 통해 데이터를 추출해라

> 데이터 스크래핑은 데이터를 추출하는 기능이 들어있다. 
특정 페이지에서 특정 값(Text, URL)을 선택하여 추출해낼 수 있다.
> 

✔️ **해당 실습의 경우 “삼성전자”라고 검색한 네이버 뉴스 페이지에서 기사들의 제목과 해당 URL을 추출해냈다.**

![image](https://user-images.githubusercontent.com/73332608/200778698-dead2dfc-8f10-4827-afbc-6d97b3b58374.png)

![image](https://user-images.githubusercontent.com/73332608/200778746-8b674e1c-0587-451d-a53d-5b66281b7864.png)

## Write Csv를 이용해 추출한 데이터를 csv 파일로 내보내자

![image](https://user-images.githubusercontent.com/73332608/200778832-bec1e9e9-9f40-4fe2-868f-202475b9e6fb.png)

## 실행

> 위 모두 단계를 마무리한 뒤 실행하면 “**네이버 브라우저 열기 → 삼성전자 검색 → 뉴스 탭 이동 → ‘삼성전자’ 키워드를 포함한 기사들 크롤링하여 데이터 추출 → 해당 데이터 csv로 내보내기**” 의 과정이 자동으로 이루어진다.
>
