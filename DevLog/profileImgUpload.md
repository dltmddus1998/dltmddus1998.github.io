# NestJS와 Vue를 이용하여 사용자의 프로필 이미지 업데이트하기 (+ AWS S3 Bucket) 

## 백엔드 (NestJS)

## 프론트엔드 (VueJS)

### 진행 순서

1. input에서 파일이 업로드될 때 @change 이벤트가 실행되어 이미지 파일을 onPreviewImage()에 넘겨준다.
2. onPreviewImage()에서는 Web API의 readAsDatURL()을 사용한다. 이후 reader.result에 이미지의 토큰 값이 나온다.

### 미리보기

> FileReader API를 활용하여 이미지 업로드 후 미리보기를 구현했다.
> 

```tsx
const onPreviewImage = async (event: any) => {
  if (event.target.files && event.target.files[0]) {
    const reader = new FileReader();

    reader.onload = (e: any) => {
      const previewImage: any = document.getElementById('img');
      previewImage.src = e.target.result;
    };

    reader.readAsDataURL(event.target.files[0]);
  }
};
```

![image](https://user-images.githubusercontent.com/73332608/204468258-a44a9b92-2389-4efd-b32d-34866851bd32.png)

![image](https://user-images.githubusercontent.com/73332608/204468295-95c06d62-c40b-403c-8115-6992b33842b8.png)

### “S3 Bucket 업로드 → Profile 전체 업데이트” 비동기적으로 일어나도록 구현

> 
> 

```tsx

```
