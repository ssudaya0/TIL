## Mongoose .exec()

.exec()의 유무와 상관없이 기능적은 동일하다.

#### exec()을 미사용시.

- 몽고는 promise가 아니지만, 편의를 위해 co라이브러리의 async/await를 사용한다.

#### exec() 사용시

- fully-fledged promise를 제공한다.
- 호출 코라인이 Stack Trace에 포함되어 디버깅이 편하다.
