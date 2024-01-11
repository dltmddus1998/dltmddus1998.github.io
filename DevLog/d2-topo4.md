# NodeJS로 d2 실행하여 API를 통해 브라우저로 토폴로지 구현하기

> 굉장히 거창해 보이지만 사실상 별거 없다. 말그대로 백엔드에서 구현한 d2를 통해 만들어진 토폴로지 svg를 front로 api를 통해 가져오기만 하면 된다.
> 

> d2lang자체가 생소하고 이걸 어떻게 백엔드에서 구현해서 추출된 svg를 가져오는지에 대해 막막함이 있었지만 생각해보니 그냥 d2관련 명령어를 실행하도록 코드로 구현해준 뒤, 이 때 추출된 svg를 api로 get하기만 하면 간단하다는 것을 깨달았다…!
> 

## 백엔드 코드 (NodeJS)

회사에서는 NestJS를 사용하긴 하는데 테스트용을 급하게 만든거라 미리 환경설정이 돼있는 node-test 레포지토리에 임의로 진행해서 NodeJS로 구현했다.

> 우선, generateSVG라는 메서드를 따로 제작해주었는데 이는 d2파일을 통해 svg를 생성해주는 메서드이다.
> 

```jsx
const getnerateSVG = async (filename) => {
  const __dirname = resolve();
  const parentFolderPath = join(__dirname, './d2');
  const d2FilePath = join(parentFolderPath, `${filename}.d2`);

  return new Promise((resolve, reject) => {
    const process = spawn('D2_LAYOUT=tala d2', ['--watch', d2FilePath, '--browser', '0'], { shell: true });

    process.stdout.on('data', (data) => {
      console.log(`stdout: ${data}`);
    });

    process.stderr.on('data', (data) => {
      console.error(`stderr: ${data}`);
    });

    process.on('close', (code) => {
      console.log(`child process exited with the code ${code}`);
      resolve();
    });

    setTimeout(() => {
      process.kill();
      resolve();
    }, 5000);
  });
};
```

> 우선 솔직히 말하면 위 코드는 chatGpt가 다 짜준거나 다름 없는데, 그런건 상관없고 (어차피 진정한 개발자는 코드를 잘 짜는 개발자가 아니라 코드 스니펫들을 어떻게 기가 막히게 활용하느냐에 따라 갈리는거다!! 아마도…?) 
✏️ 여기서 알게 된 사실 몇가지 대충 끄적여보겠다.
> 

### execAsync

> execAsync는 특정 명령어를 실행하기 위한 util 라이브러리에 있는 한 메서드인데 이 execAsync를 사용할 때 주의해야할 점은, 이 메서드는 비동기적으로 명령어를 실행하고 명령어가 완료될 때까지 기다린 후에 다음 코드를 실행하기 때문에 d2 --watch와 같은 계속 지켜보는 명령어를 쓰는 경우엔 맞지 않는 메서드라고 판단됐다.
> 

### spawn

> 그래서 위의 execAsync 대신 사용하게 된게 child_process 모듈의 spawn인데, 이는 프로세스를 더 세밀하게 관리할 수 있다. spawn은 출력 스트림을 실시간으로 읽을 수 있게 하고, 특정 조건에서 자식 프로세스를 종료시킬 수 있다.
> 

> 그래서 위 코드를 보면 알겠지만 process.stdout, process.stderr, process.on 과 같은 프로퍼티를 통해 프로세스를 세밀하게 관리했다.
> 

**그리고, 위에 선언한 메서드를 실제 request와 response가 이루어지는 곳에서 활용해보았다.**

```jsx
app.get('/d2-cons', async (req, res) => {
  try {
    await getnerateSVG('index2');

    const __dirname = resolve();

    const parentFolderPath = join(__dirname, './d2');
    const svgFilePath = join(parentFolderPath, 'index2.svg');
    const svgContent = readFileSync(svgFilePath, 'utf-8');

    res.send(svgContent);
  } catch (error) {
    console.error(error);
    res.status(500).send('Error in SVG generation');
  }
});
```

+) index2는 파일명이다.

postman에서 위 api를 테스트해보면 다음과 같이 svg element 형태의 문자열로 응답이 잘 되는 것을 알 수 있다.

![image](https://github.com/dltmddus1998/dltmddus1998.github.io/assets/73332608/9283bb76-5f08-44ce-ac1e-85fb6b5ae64a)

## 프론트단에서 API를 통해 해당 svg 브라우저에 띄우기

> 이 부분은 사실 여기에 안써도 될정도로 상당히 쉽다. 그냥 우리가 아는 axios로 api 연결해서 데이터 가져오는 방식이다.
> 

```jsx
const svgContainer = ref<any>("");

const displaySVG = (svgString: string) => {
  if (svgContainer.value) {
    svgContainer.value.innerHTML = svgString;
  }
};
onMounted(async () => {
  try {
    const response = await axios.get("http://localhost:3000/d2-cons");

    const svgString = response.data;
    displaySVG(svgString);
  } catch (error) {
    console.error(error);
  }
});
```

> 위 방식대로 렌더링한 후 ref=”svgContainer”라고 지정해놓은 div태그 내부에 api를 통해 가져온 svg이미지를 가져오도록 구현했다.
> 

결과 화면은 보안상 업로드는 불가하다
