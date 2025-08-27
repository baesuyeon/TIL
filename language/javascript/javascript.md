* 참고: [코딩앙마 자바스크립트 강좌](https://youtu.be/3Ao3OroqQLQ?si=qiX45oqYQqZq7ZnC)

# 자바스크립트에서 비동기 작업을 다루는 방법
비동기 작업이 중첩되면서 콜백 안에 콜백이 계속 들어가는 콜백 지옥 문제를 해결하기 위해 Promise가 등장했고 
더 나아가 async/await 문법으로 비동기 코드를 동기 코드처럼 직관적으로 작성할 수 있게 되었다.

## Promise란?
- 자바스크립트의 비동기 작업을 표현하는 객체이다.
- 비동기 작업이 성공(resolve) 또는 실패(reject)했다는 상태와 그 결과 값을 다룬다.
- 지금은 값이 없지만 나중에 생길 수도 있는 값을 약속(Promise)한다.

```javascript
const promise = new Promise((resolve, reject) => {
});
```
- promise는 `new Promise`로 생성하며 resolve, reject를 인수로 받는 함수를 전달받는다.
- resolve는 성공, reject는 실패했을 때 실행되는 함수이다.

```
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    // resolve("OK");
    reject(new Error('error 발생'));
  }, 3000);
});

console.log("시작");
promise.then((result) => {
  console.log(result);
}).catch((err) => {
  console.log(err);
}).finally(() => {
  console.log("끝");
})
```
- then: 성공 시 호출되는 콜백을 등록한다
- catch: 실패 시 호출되는 콜백을 등록한다.
- finally: 성공/실패와 상관없이 마지막에 실행되는 콜백을 등록한다.

- Promise에서 resolve가 호출되면 성공으로 간주하여 `then`에 있는 동작을 실행하고
- reject가 호출되면 실패로 간주하여 `catch`에 있는 동작이 실행된다.

- .then().then().catch() 형태로 여러 작업을 체이닝 연결이 가능하여 콜백 지옥을 피하게해주는 구조적 장점이 있다.
- Promise 객체를 생성하는 순간 바로 실행된다. (then을 붙였을 때 실행되는 게 아님)

## async
- async는 함수를 선언할 때 사용할 수 있다.
- `new Promise(…)` 를 리턴하는 함수를 `async` 함수로 표현할 수 있다.
- resolve(value); 부분을 return value; 로 변경한다.
- reject(new Error(…)); 부분을 throw new Error(…); 로 수정한다.
- async 함수의 리턴 값은 무조건 Promise이다.

new Promise로 직접 정의한 함수
```
function checkAgePromise(age) {
  return new Promise((resolve, reject) => {
    if (age > 20) {
      resolve(`${age} success`);          // 성공 시 resolve
    } else {
      reject(new Error(`${age} is not over 20`)); // 실패 시 reject
    }
  });
}

// 사용 예시
checkAgePromise(25)
  .then(result => console.log(result))   // "25 success"
  .catch(error => console.error(error)); // error 출력 없음
  
checkAgePromise(15)
  .then(result => console.log(result))   // 출력 없음
  .catch(error => console.error(error)); // "15 is not over 20"
```

async 함수로 표현한 동일 로직
```
async function checkAgeAsync(age) {
  if (age > 20) {
    return `${age} success`;              // resolve 대신 return
  } else {
    throw new Error(`${age} is not over 20`); // reject 대신 throw
  }
}

// 사용 예시
checkAgePromise(25)
  .then(result => console.log(result))   // "25 success"
  .catch(error => console.error(error)); // error 출력 없음
  
checkAgePromise(15)
  .then(result => console.log(result))   // 출력 없음
  .catch(error => console.error(error)); // "15 is not over 20"
```

### await
- async 함수 내부에서 await을 사용할 수 있다.
- await은 Promise가 끝날 때까지 기다렸다가 값을 반환한다.
- reject의 경우 에러를 throw하며 이는 try-catch로 잡아야 한다.

await을 사용한 방식
```
async function checkAgeAsync(age) {
  if (age > 20) {
    return `${age} success`;              // resolve 대신 return
  } else {
    throw new Error(`${age} is not over 20`); // reject 대신 throw
  }
}

// 사용 예시 (await 방식)
async function main() {
  try {
    const result = await checkAgeAsync(25);
    console.log(result); // "25 success"
  } catch (error) {
    console.error(error.message);
  }

  try {
    const result = await checkAgeAsync(15);
    console.log(result);
  } catch (error) {
    console.error(error.message); // "15 is not over 20"
  }
}

main();
```

### 콜백 지옥 vs Promise/async-await으로 개선한 코드
비동기 작업이 중첩되면서 콜백 안에 콜백이 계속 들어가는 콜백 지옥 예시
```
console.log("시작");

setTimeout(() => {
  console.log("1단계 완료");

  setTimeout(() => {
    console.log("2단계 완료");

    setTimeout(() => {
      console.log("3단계 완료");

      setTimeout(() => {
        console.log("4단계 완료");
      }, 1000);

    }, 1000);

  }, 1000);

}, 1000);
```

실행결과
```
시작
(1초 후) 1단계 완료
(2초 후) 2단계 완료
(3초 후) 3단계 완료
(4초 후) 4단계 완료
```

Promise/async-await으로 개선한 코드
```
// Promise로 변환
function delay(ms, msg) {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log(msg);
      resolve();
    }, ms);
  });
}

// then-catch 버전
delay(1000, "1단계 완료")
  .then(() => delay(1000, "2단계 완료"))
  .then(() => delay(1000, "3단계 완료"))
  .then(() => delay(1000, "4단계 완료"));

// async/await 버전
async function run() {
  await delay(1000, "1단계 완료");
  await delay(1000, "2단계 완료");
  await delay(1000, "3단계 완료");
  await delay(1000, "4단계 완료");
}
run();

```