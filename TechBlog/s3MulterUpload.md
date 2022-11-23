# NestJS에서 AWS Multer S3 를 활용하여 이미지 업로드하기

## package 설치

*$ pnpm add -wED @types/multer*

> 해당 패키지를 설치하면 Express.Multer.File 유형을 사용할 수 있다.
> 

## 사용법

> 단일 파일을 업로드하려면 `FileIntercepter()` 를 경고 핸들러로 연결하고 `@UploadFile()` 데코레이터를 사용하여 요청으로부터 파일을 추출한다.
> 

```tsx
@Post('upload')
@Useinterceptors(FileIntercepter('file', [options]))
uploadFile(@UploadFile() file: Express.Multer.File) {
	console.log(file);
}
```

## AWS S3 multer 관련 설정

```tsx
import { Controller, Post, UploadedFile, UseInterceptors } from "@nestjs/common";
import { FileInterceptor } from "@nestjs/platform-express";

@Controller('uploads')
export class UploadsController {
  @Post('')
  @UseInterceptors(FileInterceptor('file'))
  uploadFile(@UploadedFile() file) {
    console.log(file)
  }
}
```

> 위에서 찍은 콘솔에는 fieldname, originalname, encoding, mimetype, buffer, size 이렇게 파일에 대한 정보가 나온다.
> 

> 이제 위의 정보를 AWS S3에 저장하기 위해 aws-sdk 관련 패키지를 설치해주면 된다.
> 

✔️ **aws-sdk와 @aws-sdk/client-s3를 설치해주면 된다.**

## AWS S3 multer upload 구현

> service 파일에서 다음과 같이 사진 업로드 코드를 작성할 수 있다.
> 

> 아래 코드는 사용자 전체 프로필 업데이트와 함께 결합하여 구현했기 때문에 다른 코드가 섞여있다.
> 

```tsx
async update(
  updateAccountRequestDto: UpdateAccountRequestDto
): Promise<DefaultResponse> {
  const validateAccount = await this.validateAccountByUrn(
    ...
  );
  if (!validateAccount) {
    ...
  }

  const updateData = updateAccountRequestDto.updateData();

  const { profileImg } = updateAccountRequestDto;
  this.uploadFile(profileImg);

  const updateAccount = await this.db
    .collection('...')
    .updateOne({ ... }, { $set: updateData });
  return {
    statusCode: ...,
    data: JSON.stringify(updateAccount),
  };
}

async uploadFile(@UploadedFile() file) {
  AWS.config.update({
    region: [region],
    credentials: {
      accessKeyId: process.env.[Access key ID],
      secretAccessKey: process.env.[Secret Access Key ID],
    },
  });
  try {
    const upload = await new AWS.S3()
      .putObject({
        Key: `${Date.now() + file.originalname}`,
        Body: file.buffer,
        Bucket: process.env.[Bucket Name],
      })
      .promise();
    console.log(upload);
    return upload;
  } catch (err) {
    console.log(err);
  }
}
```

> s3Config는 @aws-sdk/client-s3를 통해 S3의 region 및 access-key 설정에 관한 변수이다.
> 

> 이제 위 코드를 통해 s3 bucket에 이미지를 업로드할 수 있다. (controller)
> 

```tsx
@GrpcMethod(ACCOUNT_SERVICE_NAME, 'update')
@UseInterceptors(FileInterceptor('image'))
private async update(
  @Body() payload: UpdateAccountRequestDto
): Promise<DefaultResponse> {
  return this.accountService.update(payload);
}
```

> `@UseInterceptors(FileInterceptor(’image’))` 를 통해 이미지 검증 과정을 거친다.
>
