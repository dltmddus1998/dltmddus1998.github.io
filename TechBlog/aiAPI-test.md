# AI API 테스트

## 네이버 파파고 테스트 (node.js) - translate

```jsx
import request from 'request';

export const papagoTranslate = (req, res) => {
  const client_id = 'xxxxxxxxxx-xxx--xxxx';
  const client_secret = 'xxxxxxxx';
  const query = 'Hi, how do you do?';

  const api_url = 'https://openapi.naver.com/v1/papago/n2mt';
  let options = {
    url: api_url,
    form: { source: 'en', target: 'ko', text: query },
    headers: { 'X-Naver-Client-Id': client_id, 'X-Naver-Client-Secret': client_secret },
  };

  request.post(options, (err, response, body) => {
    if (!err && response.statusCode === 200) {
      res.writeHead(200, { 'Content-Type': 'text/json;charset=utf-8' });
      res.end(body);
    } else {
      res.status(response.statusCode).end();
      console.log(`error = `, response.statusCode);
    }
  });
};
```

![스크린샷 2023-03-06 오후 2 08 44](https://user-images.githubusercontent.com/73332608/223047497-49775148-df07-4bba-893d-087e19bfc89c.png)

## dall-e of openAI (node.js) - text to image

```jsx
import { Configuration, OpenAIApi } from 'openai';

export const dalle = async (req, res) => {
  const { prompt } = req.body;
  const configuration = new Configuration({
    apiKey: process.env.OPENAI_API_KEY,
  });
  const openai = new OpenAIApi(configuration);
  const response = await openai.createImage({
    prompt,
    n: 1,
    size: '1024x1024',
  });
  const image_url = response['data']['data'][0]['url'];
  res.send(image_url);
};
```

<img width="878" alt="스크린샷 2023-03-06 오후 3 17 26" src="https://user-images.githubusercontent.com/73332608/223047462-73155cab-400b-4d02-81af-8b9b1828dee6.png">

![laugh](https://user-images.githubusercontent.com/73332608/223047563-d44baaa9-0785-4365-b8cb-30aa47c9f9a1.png)

## Stable Diffusion (python3) - text to image

```python
import os
import replicate

# Set the REPLICATE_API_TOKEN environment variable
os.environ["REPLICATE_API_TOKEN"]="xxxx_xxxxxx--x---xxx";

model = replicate.models.get("stability-ai/stable-diffusion")
version = model.versions.get("db21e45d3f7023abc2a46ee38a23973f6dce16bb082a930b0c49861f96d1e5bf")

# https://replicate.com/stability-ai/stable-diffusion/versions/db21e45d3f7023abc2a46ee38a23973f6dce16bb082a930b0c49861f96d1e5bf#input
inputs = {
    # Input prompt
    'prompt': "running dogs",

    # pixel dimensions of output image
    'image_dimensions': "768x768",

    # Specify things to not see in the output
    # 'negative_prompt': ...,

    # Number of images to output.
    # Range: 1 to 4
    'num_outputs': 1,

    # Number of denoising steps
    # Range: 1 to 500
    'num_inference_steps': 50,

    # Scale for classifier-free guidance
    # Range: 1 to 20
    'guidance_scale': 7.5,

    # Choose a scheduler.
    'scheduler': "DPMSolverMultistep",

    # Random seed. Leave blank to randomize the seed
    # 'seed': ...,
}

# https://replicate.com/stability-ai/stable-diffusion/versions/db21e45d3f7023abc2a46ee38a23973f6dce16bb082a930b0c49861f96d1e5bf#output-schema
output = version.predict(**inputs)
print(output)
```

```
➜  stable-diffusion git:(main) ✗ python3 stable-diffusion.py

['https://replicate.delivery/pbxt/QfT9gYT7842dUKAgSXJBG6fmggfJLfz83e5K92nnfxFd7BNJE/out-0.png']
```

![running dogs](https://user-images.githubusercontent.com/73332608/223047614-2eb20dd5-eed9-436f-815e-0ffb43879d81.png)
