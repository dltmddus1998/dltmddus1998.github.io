# Smart Html Elements - Grid [특정 Row 데이터 가져오기]

## Vue에서 Smart Html Elements의 Grid를 활용하던 중 생긴 Issue

### 💡 원래 사용한 방식

> event.target.textContent를 해보니까 각 row마다 다른 데이터가 나오길래 이걸 경우의 수로 나눠서 아래와 같이 말도 안되게 코드를 짰다.
> 

```tsx
if (event) {
	accountsList.value.map((accounts: any) => {
	  if (accounts.email === value) {
	    email.value = value;
	    displayName.value = accounts.displayName;
	    profileImg.value = accounts.profileImg;
	    provider.value = accounts.provider;
	    createDate.value = accounts.createDate;
	    urn.value = accounts.urn.urn;
	  } else if (accounts.displayName === value) {
		  .
			.
			. (위, 아래 코드 많이 반복...)
	  } else if (accounts.createDate === value) {
	    email.value = accounts.email;
	    displayName.value = accounts.displayName;
	    profileImg.value = accounts.profileImg;
	    provider.value = accounts.provider;
	    createDate.value = value;
	    urn.value = accounts.urn.urn;
	  }
	});
}
```

🫥 **이렇게 짜보니 데이터가 가져와지긴 했지만 지금은 row가 몇 개 없어서 가능한 코드였다. 만약 열개가 넘어가면 이 쓸데없는 코드가 얼마나 길어졌을까.**

### 🧐 개선한 방식

**smart html elements**의 `grid`의 속성 중 `getSelectedRowIds`와 `getRowData`를 활용하면 훨씬 간단하게 특정 row의 데이터를 추출할 수 있다는 것을 알게 되었고 다음과 같이 코드를 짰다.

```tsx
if (event) {
  const grid = <Grid>document.querySelector('#grid');
  const rowId = grid.getSelectedRowIds()[0]; // 현재 누른 row의 번호
  const rowData = grid.getRowData(rowId);

  // data 객체에 grid로부터 클릭하여 가져온 rowData 할당
  data.value = rowData;
}
```

위처럼 데이터를 뽑아와서 다음과 같이 데이터를 가공하여 활용했다.

![image](https://user-images.githubusercontent.com/73332608/203239865-14341d49-0108-4a4e-ad4d-fc8eed21a285.png)
