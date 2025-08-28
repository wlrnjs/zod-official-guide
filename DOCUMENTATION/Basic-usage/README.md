## 기본 사용법 (Basic usage)

이 페이지에서는 스키마(schema) 생성, 데이터 파싱(parsing), 타입 추론(inferred types)의 기본을 다룹니다. Zod의 스키마 API 전체 문서는 https://zod.dev/api 를 참고하세요.

---

### 스키마 정의하기 (Defining a schema)

무엇을 하기 전에는 먼저 **스키마(schema)**를 정의해야 합니다.
이 가이드에서는 간단한 객체(object) 스키마를 예시로 사용하겠습니다.

```ts
import * as { z } from "zod";

const Player = z.object({
  username: z.string(),
  xp: z.number(),
});
```

---

### 데이터 파싱하기 (Parsing data)

어떤 Zod 스키마든 `.parse` 메서드를 사용해 입력값을 검증할 수 있습니다.
입력이 유효하면, Zod는 타입이 보장된(deep clone) 값을 반환합니다.

```ts
Player.parse({ username: "billie", xp: 100 });
// => returns { username: "billie", xp: 100 }
```

> 참고 - 스키마에서 비동기 API(예: async refinements, transforms)를 사용한다면 `.parseAsync()` 메서드를 사용해야 합니다.

```ts
await Player.parseAsync({ username: "billie", xp: 100 });
```

---

### 에러 처리 (Handling errors)

검증(validation)이 실패하면 `.parse()` 메서드는 ZodError 인스턴스를 던지며,
여기에는 구체적인(validation issues) 정보가 포함되어 있습니다.

```ts
try {
  Player.parse({ username: 42, xp: "100" });
} catch (error) {
  if (error instanceof z.ZodError) {
    error.issues;
    /* [
      {
        expected: 'string',
        code: 'invalid_type',
        path: [ 'username' ],
        message: 'Invalid input: expected string'
      },
      {
        expected: 'number',
        code: 'invalid_type',
        path: [ 'xp' ],
        message: 'Invalid input: expected number'
      }
    ] */
  }
}
```

`try/catch` 블록을 피하고 싶다면 `.safeParse()`를 사용할 수 있습니다.
이 메서드는 성공 여부(success)와 결과(data 또는 error)가 들어 있는 plain object를 반환합니다.

```ts
const result = Player.safeParse({ username: 42, xp: "100" });
if (!result.success) {
  result.error; // ZodError instance
} else {
  result.data; // { username: string; xp: number }
}
```

> 참고 - 비동기 API를 사용하는 경우 `.safeParseAsync()`를 사용해야 합니다.

```ts
await schema.safeParseAsync("hello");
```

---

### 타입 추론 (Inferring types)

Zod는 정의한 스키마로부터 정적 타입을 추론합니다.
이 타입은 `z.infer<>` 유틸리티를 사용해 추출할 수 있습니다.

```ts
const Player = z.object({
  username: z.string(),
  xp: z.number(),
});

// 추론된 타입 추출
type Player = z.infer<typeof Player>;

// 코드에서 사용
const player: Player = { username: "billie", xp: 100 };
```

일부 경우에는 입력 타입과 출력 타입이 다를 수 있습니다.
예를 들어, `.transform()` API는 입력값을 다른 타입으로 변환할 수 있습니다.
이런 경우에는 입력 타입과 출력 타입을 각각 추출할 수 있습니다.

```ts
const mySchema = z.string().transform((val) => val.length);

type MySchemaIn = z.input<typeof mySchema>;
// => string

type MySchemaOut = z.output<typeof mySchema>; // z.infer<typeof mySchema> 와 동일
// number
```
