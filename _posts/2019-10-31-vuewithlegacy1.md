---
layout: post
title:  "[Vue.js] #1.레거시 위에 vue.js 올리기(IIS웹서버에 vue 빌드 올리기)"
categories: front 
comments: true
---





개인적으로 나의 가장 큰 고민은 vue를 만드는게 아닌, 기존에 가지고 있던 코드들(레거시)과 vue를 어떻게 짬뽕시킬것인지에 대한 것이었다.

처음부터 끝까지 vue로 사이트를 만들면... vue에 대한것만 하면 될텐데...

50%은 레거시, 50%은 뷰로 만들고싶다면 

1. 이 둘을 어떻게 합칠것인지, 

2. 이 빌드파일을 현재의 웹서버에 어떻게 연동할 것인지, (난 IIS를 쓴다)

3. API를 어떻게 만들것인지,(왜냐면 기효님이 프론트는 db와 바로 연결할수없어서 보통은 api로 받아와서 사용한다고 했다.)

4. 현재 레거시에서 vue로 어떻게 넘길것인지,

5. 로그인처리(세션처리)는 어떻게 할것인지...가 아마 가장 크지 않을까 싶다.

내가 이거 고민한것만해도 굉장히 오랜 기간동안 고민하고 실험해 본거라 리마인드 && 정리를 위해 시리즈로 적어볼 생각이다.



내 레거시 스택은 classic ASP이고, 로그인은 asp에서, 팝업창(window.open)은 vue로 띄우는 걸 기획했다.



우선 내가 다시 vue로 뭔가를 만들어봐야겠다, 생각했을때는 이미 API를 만들어놓은 상태였다. 

그러니까.. 3번은 이미 해결이 된것 

*참고글은 [classic ASP JSON으로 만들기](https://soraji.github.io/back/2019/09/25/classASPjson/)*

---

전체적인 이 구조를 만들다보면 처음부터 큰 그림을 잡고 착착착 진행헀으면 좋겠지만

보통은 그렇지 못하다. 

기능하나만 목표로 잡고 그 기능을 구현하고, 또 기능을 구현하고 그 기능들을 결론적으로는 어떻게 연결시킬것인지 고민한다. 



내가 목표로 둔 기능은 이러했다.

1. api를 만들어 어떻게 json형식으로 데이터를 가져올것인가
2. vue의 코드들을 어떻게 웹서버에서 올릴것인가
3. 노드서버에서(localhost:8080) 돌리는 vue를 어떻게 IIS와 연동할것인가
4. 현재 레거시에서 window.open을 했을때 어떻게 vue를 부르는가
5. 레거시에서 로그인한 상태(세션)을 어떻게 vue로 가져올것인가

이 단계만 끝내면 그 이후로는 vue만 작업하면 되는거라 그때는 찾는게 더 쉬워지지 않을까..?(나만의 착각인가)



그래서 이 다섯개를 시리즈로 해서 레거시와 합치는 방법을 기록하려한다.



우선 이번글은 1번은 완료되었으니 2번을 해야겠지

2번.vue의 코드들을 어떻게 웹서버에 올릴것인가

asp를 하다보면 npm run build개념이나, npm run serve개념이나(난 cli3쓴다) 처음에 이게 뭐지 싶다.

왜냐면 asp는 그냥 asp만 올리면 알아서 html파일이라 css js다 직접가지고 오는데 

vue는 .vue파일을 어떻게 IIS에서 렌더링을 해주는지도 모르겠고..



node를 웹서버로 사용하다가 (localhost) 실제로 IIS에다가 올리는걸 어떻게 해야하는지.. 

누군가에게는 쉽겠지만 나한테는 물어볼 상황도 아니었다.... 흑흑

build라는걸 왜 해주는지에 대해서도 몰랐는데 그냥 하다보니까 이게 '아.. 이래서 해주는구나' 싶었다.

(빌드하게 되면 src에 가지고 있던 assets들과 js, css, routes, vuex등등을 다 하나의 css와 js로 묶어서 빌드를 해준다

결론적으로 말하면 index.html, css, js만 있다는 소리다. 아, favicon도 있네ㅋㅋ 온갖거 다 가지고 오는 html보다 훨씬 직관적이고 편하다)





여하튼,

본인이 local에서 돌린 vue가 완성이 되어서 배포를 하고 싶으면 npm run build를 한다. 

그러면 프로젝트 폴더에 dist라는 디렉토리가 새로 생성이 된다.

그 dist폴더에 들어있는게 핵심인데 IIS웹서버에 올리는거는 이 dist폴더만 올리면 된다.

새로운 사이트를 만들던, 기존에 레거시에 추가를 하던, 그 dist폴더만 넣어주면 된다.

~~~
root
    -project
        -dir1
        -dir2
        -dir3
        -dist
~~~

이런식으로 트리구조가 되어있다고 생각하면 된다.



그리고 경로는 그 vue에 접속을 하고싶다면 

~~~
웹사이트/dist/index.html(혹은 라우트)
~~~

 로 해주면 된다.





그치만 아마 IIS에서 세팅을 따로 해주지 않아 브라우저에서 불러오지는 못할것이다. 

왜냐면 3번, 노드서버에서(localhost:8080) 돌리는 vue를 어떻게 IIS와 연동할것인가 을 해결해야지 node를 IIS에서도 돌릴수있기 때문이다.

(근데 이 상태로 열리는지 안열리는지 모르겠네.. 시간이 좀 지나서 기억이 안난다 아마 500에러가 떴을텐데)

다음은 3번에 대한 글로 올려야지!





