# D3로 도형 그래그 구현하기
## 구현 예정

> 추후에 토폴로지를 d3를 활용하여 구현할 예정인데 자유자재로 구현할 수 있기 위한 스킬업을 위해 간단한 도형과 도형의 드래그를 통한 위치 이동, 도형간 연결 등을 구현해볼 예정이다.
> 

## 구현 방법 (코드)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>D3 circles</title>
    <link rel="stylesheet" href="text/css" href="style.css" />
    <script src="https://cdn.jsdelivr.net/npm/d3@6.3.0/dist/d3.min.js"></script>
  </head>
  <body>
    <div id="chart">
      <div id="tooltip"></div>
    </div>
    <script src="https://d3js.org/d3.v6.min.js"></script>
    <script src="app.js"></script>
  </body>
</html>
```

```jsx
function draw() {
  const svg = d3.select("svg");
  const data = {
    nodes: [
      { id: "A", x: 100, y: 100 },
      { id: "B", x: 200, y: 200 },
      { id: "C", x: 300, y: 100 },
    ],
    links: [
      { source: "A", target: "B" },
      { source: "B", target: "C" },
    ],
  };

  const link = svg
    .append("g")
    .attr("class", "links")
    .selectAll("line")
    .data(data.links)
    .enter()
    .append("line")
    .attr("x1", (d) => d.source.x)
    .attr("y1", (d) => d.source.y)
    .attr("x2", (d) => d.target.x)
    .attr("y2", (d) => d.target.y);

  const node = svg
    .selectAll(".node")
    .data(data.nodes)
    .enter()
    .append("circle")
    .attr("cx", (d) => {
      console.log(d);
      return d.x;
    })
    .attr("cy", (d) => d.y)
    .attr("r", 20)
    .attr("class", "node")
    .call(d3.drag().on("start", dragstarted).on("drag", dragged).on("end", dragended));

  function dragstarted(d) {
    d3.select(this).raise().classed("active", true);
  }

  function dragended(d) {
    d3.select(this).classed("active", false);
  }

  function dragged(d) {
    d3.select(this)
      .attr("cx", (d.x = d3.event.x))
      .attr("cy", (d.y = d3.event.y));

    link
      .filter((l) => {
        // console.log(l);
        return l.source === d || l.target === d;
      })
      .attr("x1", (d) => {
        console.log("asdjfklsajf;as");
        console.log(d);
        return d.x;
      })
      .attr("y1", (d) => d.y)
      .attr("x2", (l) => {
        console.log(l);
        return l.source === d ? d.x : l.target.x;
      })
      .attr("y2", (l) => (l.source === d ? d.y : l.target.y));
  }
}

draw();
```

## Issue & Error-Handling

### 😭 도형간 node 연결 관련 이슈 (at line element)

> 진짜 이거 해결하다가 돌아버리는 줄 알았다. d3가 워낙 대중적인 기술이다 보니 chatgpt를 활용해서 구현중이었는데, 얘가 중요한걸 안알려주고 에러났다고 하면 그제서야 해당 주요 코드나 정보를 알려준다. 일부러 그러는건가? 🤪
> 

⁉️ 그래서 결국, 원인은 뭐였냐… 위 코드에서 link를 선언해주는 부분에 attr 메서드를 이용하여 특정 element에 데이터값을 적용하는 과정에서 일어난 이슈인데, 애초에 선언한 data의 link 부분을 보면 d.source.x와 같은 코드가 나올 수가 없다. x가 포함이 안되어있지 않나. 예상대로 콘솔에 찍어보니 `undefined`라고 뜨더라.

⁉️ 이걸 chatgpt에 다음과 같이 물어보니까 해결책을 내놓더라.

![image](https://user-images.githubusercontent.com/73332608/216917118-c1e7ef7b-8bc0-4583-b631-e24acc5e6e20.png)

👉🏻 이걸 보고, 아 그럼 `d.source.x = d3.event.x` `d.target.x = d3.event.y` 와 같이 변경해주면 되지 않을까? 라는 생각이 들었다. 그래서 다음 코드를 추가했다. 

```jsx
const link = svg
  .selectAll(".link")
  .data(data.links)
  .enter()
  .append("line")
  .attr("x1", d => d.source.x)
  .attr("y1", d => d.source.y)
  .attr("x2", d => d.target.x)
  .attr("y2", d => d.target.y)
  .attr("class", "link");

function dragged(d) {
  d3.select(this)
    .attr("cx", d.x = d3.event.x)
    .attr("cy", d.y = d3.event.y);

  link.filter(l => l.source === d || l.target === d)
    .attr("x1", d => d.x)
    .attr("y1", d => d.y)
    .attr("x2", l => (l.source === d ? d.x : l.target.x))
    .attr("y2", l => (l.source === d ? d.y : l.target.y));
}
```

🤪 그런데도 끝끝내 도형을 연결해주는 선이 안보이더라. d3.event.x를 따로 할당해주었는데도 그러면 뭐가 문제일까… 계속 부분적으로 코드만 고치다가는 해결을 못할 것 같아서 아예 새로 코드를 작성해봤다. (HTML&CSS 파일은 중요한 부분이 아니니 생략하겠다.)


### 최종 코드

```jsx
// Set up the data for the nodes and links
const nodes = [{ id: 1 }, { id: 2 }, { id: 3 }, { id: 4 }, { id: 5 }];
const links = [
  { source: 1, target: 2 },
  { source: 2, target: 3 },
  { source: 3, target: 4 },
  { source: 4, target: 5 },
];

const updatedNodes = [
  { id: 1, x: 100, y: 200 },
  { id: 2, x: 150, y: 150 },
  { id: 3, x: 200, y: 100 },
  { id: 4, x: 250, y: 150 },
  { id: 5, x: 300, y: 200 },
];

const updatedLinks = [
  { source: updatedNodes[0], target: updatedNodes[1] },
  { source: updatedNodes[1], target: updatedNodes[2] },
  { source: updatedNodes[2], target: updatedNodes[3] },
  { source: updatedNodes[3], target: updatedNodes[4] },
];

// Create the SVG element
const svg = d3.select("body").append("svg").attr("width", 1000).attr("height", 1000);

const node = svg.selectAll("circle").data(updatedNodes);

node
  .enter()
  .append("circle")
  .merge(node)
  .attr("r", 20)
  .style("fill", "lightblue")
  .attr("cx", (d) => d.x)
  .attr("cy", (d) => d.y)
  .call(d3.drag().on("start", dragStarted).on("drag", dragging).on("end", dragEnded));

node.exit().remove();

const link = svg.selectAll("line").data(updatedLinks);

link
  .enter()
  .append("line")
  .merge(link)
  .style("stroke", "black")
  .style("stroke-width", 2)
  .attr("x1", (d) => d.source.x)
  .attr("y1", (d) => d.source.y)
  .attr("x2", (d) => d.target.x)
  .attr("y2", (d) => d.target.y);

link.exit().remove();

// Define the drag behavior for the nodes
function dragStarted(d) {
  d3.select(this).raise();
}

function dragging(d) {
  d.x = d3.event.x;
  d.y = d3.event.y;

  d3.select(this).attr("cx", d.x).attr("cy", d.y);

  link
    .filter((l) => l.source === d || l.target === d)
    .attr("x1", (l) => l.source.x)
    .attr("y1", (l) => l.source.y)
    .attr("x2", (l) => l.target.x)
    .attr("y2", (l) => l.target.y);
}

function dragEnded(d) {
  d.x = d3.event.x;
  d.y = d3.event.y;

  d3.select(this).attr("cx", d.x).attr("cy", d.y);

  link
    .filter((l) => l.source === d || l.target === d)
    .attr("x1", (l) => l.source.x)
    .attr("y1", (l) => l.source.y)
    .attr("x2", (l) => l.target.x)
    .attr("y2", (l) => l.target.y);
}
```
