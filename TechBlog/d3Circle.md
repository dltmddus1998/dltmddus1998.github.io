> d3는 사용하기에 까다로운 라이브러리이기 때문에 간단한 실습부터 시작해보겠다.
> 

```html
<!-- index.html -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <script src="example.js"></script>
    <title>Document</title>
  </head>
  <body onload="main()"></body>
</html>
```

```jsx
// example.js

function main() {
  let width = 500;
  let height = 500;
  let data = [10, 15, 20, 25, 30];
  let colors = ['#00005C', '#3B185F', '#C060A1', '#F0CAA3', '#3F3B6C'];
  let svg = d3.select('body').append('svg').attr('width', width).attr('height', height);
  let group = svg
    .selectAll('g')
    .data(data)
    .enter()
    .append('g')
    .attr('transform', function (d, i) {
      return 'translate(0,0)';
    });
  group
    .append('circle')
    .attr('cx', function (d, i) {
      // x축 위치
      return i * 100 + 50;
    })
    .attr('cy', function (d, i) {
      // y축 위치
      return 100;
    })
    .attr('r', function (d) {
      // 원 크
      return d * 1.5;
    })
    .attr('fill', function (d, i) {
      // 색 설정
      return colors[i];
    });
  group
    .append('text') // 텍스트 설정
    .attr('x', function (d, i) {
      return i * 100 + 45;
    })
    .attr('y', 105)
    .attr('stroke', 'white')
    .attr('font-size', '20px')
    .attr('font-family', 'snas-serif')
    .text(function (d) {
      return d;
    });
}
```

> 우선 변수들을 통해 width, height, data, colors 속성들을 설정해뒀다.
> 

> group이라는 변수에 d3에서 body태그에 svg 속성을 더한 부분에서 g태그에 이제 만들고자하는 데이터를 더한 것을 할당한다.
> 

> 이후 cx, cy, r 차례대로 x축, y축, 원의 반지름을 append, attr 메서드를 통해 설정해준 후 텍스트 또한 설정이 가능했다.
> 

💡 **결과는 다음과 같이 나온다.**

![스크린샷 2022-12-05 오전 11 00 58](https://user-images.githubusercontent.com/73332608/205533452-04531084-17c4-4e20-a1e0-7a44c97fea51.png)
