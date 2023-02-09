# 📑 AG GRID

## 😌 Smart Html Elements의 Grid에서 AG GRID로 업데이트한 후 장점

> 배포하기 전까지는 모르겠지만 확실히 덜 무거워진 것 같다. 실행했을 때 로딩 시간이 눈에 띌 정도로 빨라졌고 구현에 있어서는 smart html elements보다 복잡한 부분은 전혀 없었다.
> 

## ✏️ 구현 과정

> 이전에 구현해 놓은 데이터 테이블 중에 하나를 먼저 업데이트 해보았는데 코드는 다음과 같이 나왔다. 
(Vue3을 이용해 개발, function 관련 코드만 올려놓을 예정)
> 

### 👾 코드

```tsx
// processColumns.ts

const columnType = (data: any) => {
  if (
    typeof data === 'string' ||
    typeof data === 'boolean' ||
    typeof data === 'number' ||
    data === null
  )
    return 'column';
  else if (typeof data === 'object' && Object.keys(data)[0] === '0') return 'array';
  else if (Array.isArray(data)) return 'array';
  else return 'group';
};

export const gridMethods = {
  makeSources: async (datas: any) => {
    if (Array.isArray(datas.value)) {
      return await Promise.all(
        datas.value.map((item: any) => {
          const keys = Object.keys(item);
          const source = {} as { [column: string]: string };
          keys.forEach((key) => {
            const type = columnType(item[key]);
            if (type === 'column') source[key] = item[key];
            if (type === 'group') {
              const fields = Object.keys(item[key]);
              fields.forEach((field) => {
                source[field] = item[key][field];
              });
            }
          });
          return { ...source };
        })
      );
    }
  },
  setColumns: async (datas: any, columnList: any) => {
    const processedDatas = await gridMethods.makeSources(datas);
    if (processedDatas) {
      processedDatas.forEach((data) => {
        const columns = data;
        columnList.value = [];
        const fields = Object.keys(columns);

        fields.forEach((field: any) => {
          if (columns[field]) {
            columnList.value.push({
              headerName: field,
            });
          }
        });
      });
    }
  },
};
```

```tsx
<AgGrid
  v-model:datas="propDatas"
  :column-list="fieldList"
  :settings="gridSettings"
  :style="`width: 1000px; height: 100%`"
  @row-selected="onRowSelected"
/>

// vue 파일의 script 내부 코드 중 일부분
import { AgGrid } from '[AgGrid Component 생성한 파일]'

const getData = async () => {
  const query = JSON.stringify({});
  const res = await getTenants({ query });

  if (res.data.value.statusCode === 200) datas.value = res.data.value.data;

  columns.value = [
    'companyName',
    'tenantName',
    'companyContactEmail',
    'companyContactName',
    'companyContactPhone',
    'companyBusinessNumber',
    'startDate',
    'dueDate',
  ];

  columns.value.forEach((column: any) => {
    if (column) fieldList.value.push({ field: column });
  });
};

const setTableDatas = async () => {
  await getData();
  await gridMethods.setColumns(datas, columnList);
  propDatas.value = await gridMethods.makeSources(datas);
};

onMounted(async () => {
  await setTableDatas();
});
```

### 💡 상세 설명

우선, processColumns.ts에서 columnType은 DB에서 받아오는 데이터들의 데이터 타입이 각각 다르기 때문에 (어떤 데이터는 문자열, 어떤 문자열은 객체… 등등) 각각 다르게 가공하기 위해 타입을 구분하기 위한 메서드이다. 

그리고 gridMethods의 makeSources는 위에서 만든 columnType을 활용하여 타입별로 데이터를 가공하는 방법을 나눠놓은 메서드이다.

gridMethods의 setColumns는 makeSources를 통해 타입별로 데이터를 가공한 뒤, Ag Grid에서 사용되는 타입인 ColDef타입의 ColumnList에 최종 필드명들을 삽입하는 메서드이다.

Container.vue에서 getData를 통해 백엔드에 구현해놓은 API를 호출하여 DB에서 데이터를 받아온다. 여기서 columns라는 ref 값에 미리 컬럼을 설정해주어야 컬럼 순서를 지정해줄 수 있다.

setTableDatas에는 processColumns에서 만들어놓은 메서드들을 호출하여 최종 데이터인 propDatas에 최종 데이터를 할당하여 AgGrid에 props 데이터에 이를 보내면 된다.

### 📺 최종 화면
![image](https://user-images.githubusercontent.com/73332608/217750158-c18c7e3c-df1f-4643-be81-7056c8c8940f.png)
