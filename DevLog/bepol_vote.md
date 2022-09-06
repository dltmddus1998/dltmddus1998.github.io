# Bepol 프로젝트 중 찬반투표 및 투표 취소 기능 서버 개발

## 찬반투표 API

⚠️ 아직 소셜 로그인 부분이 git에 머지되지 않았으므로 유저 관련 부분은 임의의 ID를 통해 구현했다. 이 부분은 추후에 추가 구현할 예정이다.

⚠️ model에 있는 데이터 파일에서 데이터 조작을 진행하다보니 파일이 너무 길어졌다. 좀더 계층 분리를 확실히 하기 위해 services 폴더를 추가했다.

☑️ 먼저 컨트롤러 부분에는 최대한 DDL에 대한 내용이 노출되지 않도록 하기 위해 models폴더에 있는 파일에 DDL에 대한 함수들을 선언하여 컨트롤러에서는 이를 import해와서 썼다.

💡 찬반투표의 경우, 두 가지의 데이터 변화가 필요하다. 

1. **post_answer** 컬렉션에 해당 유저의 찬반 투표 결과 데이터를 추가
2. **post** 컬렉션에 해당 게시글의 찬성 혹은 반대 수 ++

이 두 가지는 model에서 구현한 `addAnswer`와 `addAgrees`, `addDisagrees` 메서드를 통해 기능을 구현했다.

💡 트랜잭션 처리를 위해 mongoose의 트랜잭션 처리 관련 문법을 활용했다. (`addAnswerTransaction` 메서드)

*(참고 사이트: [https://mongoosejs.com/docs/transactions.html](https://mongoosejs.com/docs/transactions.html))*

### Controller

```jsx
export const voteToPost = async (req, res, next) => {
  const { agree } = req.body;
  const { accesstoken } = req.headers;
  const { postId } = req.params;
  const userId = "62e1eb6f6cc8d5e6d3bfac2d"; // 소셜로그인 구현되면 변경
  const votedUser = await postAnswerRepository.getUserIdAnswered(userId);

  if (!accesstoken) {
    // user 정보 불일치시 error
    return res.status(401).json({
      message: "Unauthorized user",
    });
  } else if (votedUser) {
    // 이미 투표한 경우
    return res.status(403).json({
      message: "Already voted user!",
    });
  } else {
    postAnswerRepository.addAnswerTransaction(postId, userId, agree);

    return res.status(201).json({
      message: "Voted successfully",
    });
  }
};
```

### DDL

```jsx
export const getUserIdAnswered = async (userId) => {
  return Post_answers.findOne({ userId });
};

export const addAnswer = async (postId, userId, agree) => {
  return Post_answers.create({
    id: postId + userId,
    postId,
    userId,
    answer: agree,
  });
};

export const addAgrees = async (postId) => {
  return Post.findOne({ id: postId }, { userId: false }).then((post) => {
    post.agrees++;
    return post.save();
  });
};

export const addDisagrees = async (postId) => {
  return Post.findOne({ id: postId }, { userId: false }).then((post) => {
    post.disagrees++;
    return post.save();
  });
};
```

### 트랜잭션 처리

```jsx
export const addAnswerTransaction = async (postId, userId, agree) => {
  /**
   * 투표 반영 -> Post, Post_answer
   */
  const session = await mongoose.startSession();
  try {
    session.startTransaction(); // 트랜잭션 시작

    await addAnswer(postId, userId, agree, session); // Post_answer 컬렉션에 도큐먼트 추가
    await (agree === true ? addAgrees(postId, session) : addDisagrees(postId, session)); // 찬성, 반대 여부에 맞게 Post 컬렉션 수정
    await pushVoteStatistics(postId, userId, agree, session); // 통계 결과 입력

    await session.commitTransaction();
    session.endSession();

    console.log("Transaction success!!");
    return "success";
  } catch (err) {
    console.error(err, "Transaction error!!!");
    await session.abortTransaction();
    session.endSession();
  }
};
```

## 찬반투표 취소 API

이 부분은 찬반투표 부분과 반대되는 로직이므로 거의 유사하다. (설명 생략)

### Controller

```jsx
export const voteDeleteToPost = async (req, res, next) => {
  const { accesstoken } = req.headers;
  const { postId } = req.params;
  const userId = "62e1eb6f6cc8d5e6d3bfac2d"; // 소셜로그인 구현되면 변경
  const userPostAnswer = await postAnswerRepository.findUserAnswer(userId);
  const { answer } = userPostAnswer;
  const votedUser = await postAnswerRepository.getUserIdAnswered(userId);

  if (!accesstoken) {
    // user 정보 불일치시 error
    res.status(401).json({
      message: "Unauthorized user",
    });
  } else if (!votedUser) {
    // 투표 안한 경우
    return res.status(403).json({
      message: "No vote record of this user!!",
    });
  } else {
    postAnswerRepository.deleteAnswerTransaction(postId, userId, answer);

    return res.status(200).json({
      message: "Vote is deleted!!",
    });
  }
};
```

### DDL

```jsx
export const findUserAnswer = async (userId) => {
  return Post_answers.findOne({ userId });
};

export const deleteAnswer = async (postId, userId) => {
  return Post_answers.deleteOne({ id: postId + userId });
};

export const substractAgrees = async (postId) => {
  return Post.findOne({ id: postId }, { userId: false }).then((post) => {
    post.agrees--;
    return post.save();
  });
};

export const substractDisagrees = async (postId) => {
  return Post.findOne({ id: postId }, { userId: false }).then((post) => {
    post.disagrees--;
    return post.save();
  });
};
```

### 트랜잭션 처리

```jsx
export const deleteAnswerTransaction = async (postId, userId, answer) => {
  /**
   * 투표 취소 반영 -> Post, Post_answer
   */
  const session = await mongoose.startSession();
  try {
    session.startTransaction(); // 트랜잭션 시작

    await deleteAnswer(postId, userId, session); // Post_answer 컬렉션에서 해당 도큐먼트 삭제
    await (answer === true ? substractAgrees(postId, session) : substractDisagrees(postId, session)); // Post 컬렉션에서 찬성 반대 여부에 맞게 수정
    await popVoteStatistics(postId, userId, answer, session); // 통계 결과 입력

    await session.commitTransaction();
    session.endSession();

    console.log("Transaction success!!");
    return "success";
  } catch (err) {
    console.error(err, "Transaction error!!");
    await session.abortTransaction();
    session.endSession();
  }
};

export const getPostStatistics = async (postId) => {
  // 통계 조회
  return Post_statistics.findOne({ postId });
};
```