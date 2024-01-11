# D2로 토폴로지 구현하기

> d2를 활용하여 region 표시하는 세계지도를 구현해보았는데 한 가지 이슈가 있었다.
> 

## 엔진 문제 및 해결

기본적으로 사용하는 엔진은 dagre인데 해당 엔진을 사용하게 되면, top과 right 설정이 불가하다.

[공식문서](https://d2lang.com/tour/layouts)에 따르면 해당 엔진은 locked position 설정이 불가능하게 돼있고 가능한 엔진은 TALA가 유일하다.

→ 그래서 엔진을 TALA로 바꿨는데 여기서 살짝 애먹었다. 원래 깔아둔 엔진을 삭제하고 TALA를 삭제해야 잘 돌아가는 걸 몰라서 위에 덮어서 설치했더니 계속 에러가 났다.

→ 결국 원래 깔려 있던 걸 uninstall하고 `curl -fsSL https://d2lang.com/install.sh | sh -s -- --uninstall` TALA로 install했다. `curl -fsSL https://d2lang.com/install.sh | sh -s -- --tala --dry-run`

아래 코드와 같이 구현했더니,

```yaml
# Define a container

# Target something inside a container with "."

a: {
  shape: image
  icon: https://email-form-images.s3.ap-northeast-2.amazonaws.com/WorldMap.svg
  width: 613
  height: 320
  top: 0
  left: 0
  label: ""
}

b 2: b {
  icon: https://email-form-images.s3.ap-northeast-2.amazonaws.com/icon_DirectConnect.svg
  shape: image
  width: 26
  height: 43
  top: 128
  left: 387
  label: ""
}
b: {
  icon: https://email-form-images.s3.ap-northeast-2.amazonaws.com/icon_DirectConnect.svg
  shape: image
  width: 26
  height: 43
  top: 120
  left: 163
  label: ""
}
b 3: b {
  icon: https://email-form-images.s3.ap-northeast-2.amazonaws.com/icon_DirectConnect.svg
  shape: image
  width: 26
  height: 43
  top: 190
  left: 311
  label: ""
}
b 4: b {
  icon: https://email-form-images.s3.ap-northeast-2.amazonaws.com/icon_DirectConnect.svg
  shape: image
  width: 26
  height: 43
  top: 8
  left: 229
  label: ""
}
```

`D2_LAYOUT=tala d2 input.d2 out.svg` 라고 실행하면,

아래와 같이 나온다.

![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/2fa77e21-521e-400f-b068-290112ef1434)

out.svg

+) 워터마크는 pricing 문제때문에 뜬 것 같다. (유료 서비스이다)

### From now on…

- 현재 상태는 d2 코드를 실행시키면 svg가 파일형태로 export되는 형태인데, 이를 실제 코드에 적용하려면 d2로 구현된게 바로 코드에 적용돼야 한다.