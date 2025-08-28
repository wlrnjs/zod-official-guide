## 스키마 정의하기 (Defining schemas)

데이터를 검증하려면 먼저 스키마(schema)를 정의해야 합니다.
스키마는 단순한 원시 값(primitive)부터, 복잡한 중첩 객체와 배열까지 타입을 표현합니다.

---

### 원시 타입 (Primitives)

```ts
import * as z from "zod";

// primitive types
z.string();
z.number();
z.bigint();
z.boolean();
z.symbol();
z.undefined();
z.null();
```

---

### 강제 변환 (Coercion)

입력값을 원하는 타입으로 강제 변환하려면 `z.coerce`를 사용합니다.

```ts
z.coerce.string(); // String(input)
z.coerce.number(); // Number(input)
z.coerce.boolean(); // Boolean(input)
z.coerce.bigint(); // BigInt(input)
```

강제 변환된 스키마는 입력값을 해당 타입으로 변환하려 시도합니다.

```ts
const schema = z.coerce.string();

schema.parse("tuna"); // => "tuna"
schema.parse(42); // => "42"
schema.parse(true); // => "true"
schema.parse(null); // => "null"
```

기본적으로 입력 타입은 `unknown`으로 간주됩니다.
더 구체적인 입력 타입을 지정하려면 제네릭 파라미터를 넘겨줍니다.

```ts
const A = z.coerce.number();
type AInput = z.infer<typeof A>; // => number

const B = z.coerce.number<number>();
type BInput = z.infer<typeof B>; // => number
```

---

### 리터럴 (Literals)

리터럴 스키마는 `"hello world"` 또는 `5` 같은 특정 값만 허용하는 타입을 표현합니다.

```ts
const tuna = z.literal("tuna");
const twelve = z.literal(12);
const twobig = z.literal(2n);
const tru = z.literal(true);
```

자바스크립트의 `null`, `undefined` 리터럴은 이렇게 표현할 수 있습니다.

```ts
z.null();
z.undefined();
z.void(); // z.undefined() 와 동일
```

여러 리터럴 값을 허용하려면:

```ts
const colors = z.literal(["red", "green", "blue"]);

colors.parse("green"); // ✅
colors.parse("yellow"); // ❌
```

허용된 값들의 집합을 추출할 수도 있습니다:

```ts
colors.values; // => Set<"red" | "green" | "blue">
```

---

### 문자열 (Strings)

Zod는 문자열에 대해 다양한 검증 및 변환 API를 제공합니다.

문자열 검증 예시:

```ts
z.string().max(5);
z.string().min(5);
z.string().length(5);
z.string().regex(/^[a-z]+$/);
z.string().startsWith("aaa");
z.string().endsWith("zzz");
z.string().includes("---");
z.string().uppercase();
z.string().lowercase();
```

문자열 변환 예시

```ts
z.string().trim(); // 공백 제거
z.string().toLowerCase(); // 소문자 변환
z.string().toUpperCase(); // 대문자 변환
z.string().normalize(); // 유니코드 정규화
```

---

### 문자열 포맷 (String formats)

Zod는 여러 문자열 포맷에 대한 검증기를 제공합니다.

```ts
z.email();
z.uuid();
z.url();
z.httpUrl(); // http/https만 허용
z.hostname();
z.emoji(); // 단일 이모지
z.base64();
z.base64url();
z.hex();
z.jwt();
z.nanoid();
z.cuid();
z.cuid2();
z.ulid();
z.ipv4();
z.ipv6();
z.cidrv4(); // ipv4 CIDR 블록
z.cidrv6(); // ipv6 CIDR 블록
z.hash("sha256"); // ("sha1", "sha384", "sha512", "md5" 도 가능)
z.iso.date();
z.iso.time();
z.iso.datetime();
z.iso.duration();
```

---

### 이메일 (Emails)

```ts
z.email();
```

Zod의 기본 이메일 검증은 Gmail과 유사한 엄격한 정규식을 사용합니다.

기본 regex:

```ts
/^(?!\.)(?!.*\.\.)([a-z0-9_'+\-\.]*)[a-z0-9_+-]@([a-z0-9][a-z0-9\-]*\.)+[a-z]{2,}$/i;
```

원하는 경우 커스텀 정규식을 넘겨줄 수 있습니다.

```ts
z.email({ pattern: /your regex here/ });
```

Zod는 유용한 기본 regex도 내보냅니다:

```ts
z.email({ pattern: z.regexes.email }); // 기본
z.email({ pattern: z.regexes.html5Email }); // HTML5 input[type=email] 기준
z.email({ pattern: z.regexes.rfc5322Email });
z.email({ pattern: z.regexes.unicodeEmail }); // 유니코드 허용
```

---

### UUID (UUIDs)

```ts
z.uuid();
z.uuid({ version: "v4" }); // 특정 버전 지정 가능
z.uuidv4();
z.uuidv6();
z.uuidv7();
```

UUID 형식만 검사하고 싶다면:

```ts
z.guid();
```

---

### URL (URLs)

WHATWG 호환 URL 검증:

```ts
const schema = z.url();

schema.parse("https://example.com"); // ✅
schema.parse("http://localhost"); // ✅
schema.parse("mailto:noreply@zod.dev"); // ✅
```

호스트네임, 프로토콜 조건을 추가할 수도 있습니다.

```ts
z.url({ hostname: /^example\.com$/ });
z.url({ protocol: /^https$/ });
```

웹 URL 전용 스키마 예시:

```ts
const httpUrl = z.url({
  protocol: /^https?$/,
  hostname: z.regexes.domain,
});
```

---

### ISO 날짜/시간 (ISO datetime/date/time)

- z.iso.datetime(): ISO 8601 datetime 문자열 검증
- z.iso.date(): YYYY-MM-DD 형식 검증
- z.iso.time(): HH:MM[:SS[.s+]] 형식 검증

(세부 옵션: offset 허용, precision 제약 등 문서 그대로 사용 가능)

---

### 숫자 (Numbers)

```ts
const schema = z.number();

schema.parse(3.14); // ✅
schema.parse(NaN); // ❌
schema.parse(Infinity); // ❌
```

숫자 관련 검증:

```ts
z.number().gt(5);
z.number().gte(5); // .min(5) 와 동일
z.number().lt(5);
z.number().lte(5); // .max(5) 와 동일
z.number().positive();
z.number().nonnegative();
z.number().negative();
z.number().nonpositive();
z.number().multipleOf(5); // .step(5) 와 동일
```

---

### BigInt (BigInts)

BigInt 값을 검증하려면:

```ts
z.bigint();
```

BigInt 전용 검증 메서드:

```ts
z.bigint().gt(5n);
z.bigint().gte(5n); // .min(5n) 과 동일
z.bigint().lt(5n);
z.bigint().lte(5n); // .max(5n) 과 동일
z.bigint().positive();
z.bigint().nonnegative();
z.bigint().negative();
z.bigint().nonpositive();
z.bigint().multipleOf(5n); // .step(5n) 과 동일
```

---

### 불리언 (Booleans)

```ts
z.boolean().parse(true); // => true
z.boolean().parse(false); // => false
```

---

### 날짜 (Dates)

Date 인스턴스 검증:

```ts
z.date().safeParse(new Date()); // success: true
z.date().safeParse("2022-01-12T06:15:00.000Z"); // success: false
```

커스텀 에러 메시지:

```ts
z.date({
  error: (issue) => (issue.input === undefined ? "Required" : "Invalid date"),
});
```

날짜 전용 검증:

```ts
z.date().min(new Date("1900-01-01"), { error: "Too old!" });
z.date().max(new Date(), { error: "Too young!" });
```

---

### 열거형 (Enums)

고정된 문자열 집합에 대해 입력값 검증:

```ts
const FishEnum = z.enum(["Salmon", "Tuna", "Trout"]);

FishEnum.parse("Salmon"); // => "Salmon"
FishEnum.parse("Swordfish"); // ❌
```

> 주의: 배열을 변수로 선언하면 Zod가 정확한 리터럴 타입을 추론하지 못합니다.

```ts
const fish = ["Salmon", "Tuna", "Trout"];
const FishEnum = z.enum(fish);
type FishEnum = z.infer<typeof FishEnum>; // string
```

해결 방법:

```ts
const fish = ["Salmon", "Tuna", "Trout"] as const;
const FishEnum = z.enum(fish);
type FishEnum = z.infer<typeof FishEnum>; // "Salmon" | "Tuna" | "Trout"
```

TypeScript enum도 지원합니다. (Zod 4에서는 z.enum이 z.nativeEnum을 대체)

```ts
enum Fish {
  Salmon = "Salmon",
  Tuna = "Tuna",
  Trout = "Trout",
}

const FishEnum = z.enum(Fish);
```

추가 API:

```ts
FishEnum.enum;              // 값 객체 반환
FishEnum.exclude([...]);    // 특정 값 제외
FishEnum.extract([...]);    // 특정 값 추출
```

---

### 옵션 / 널 허용 / 널리시 (Optionals / Nullables / Nullish)

```ts
z.optional(z.literal("yoda")); // undefined 허용
z.nullable(z.literal("yoda")); // null 허용
z.nullish(z.literal("yoda")); // null + undefined 모두 허용
```

---

### 객체 (Objects)

객체 스키마 정의:

```ts
const Person = z.object({
  name: z.string(),
  age: z.number(),
});
```

옵션 필드:

```ts
const Dog = z.object({
  name: z.string(),
  age: z.number().optional(),
});
```

- 기본: 알 수 없는 키는 제거됨
- z.strictObject: 알 수 없는 키 있으면 에러 발생
- z.looseObject: 알 수 없는 키 그대로 유지
- .catchall(): 알 수 없는 키도 특정 스키마로 검증

내부 API:

- .shape -> 내부 스키마 접근
- .keyof() -> 키 이름 enum 스키마 생성
- .extend() / .safeExtend() -> 필드 확장
- .pick() / .omit() -> 특정 키 선택/제외
- .partial() / .required() -> 필드 옵셔널/필수 변환

재귀 객체도 가능 (getter 사용).

---

### 배열 (Arrays)

배열 스키마 정의:

```ts
const stringArray = z.array(z.string()); // or z.string().array()
```

배열 전용 검증:

```ts
z.array(z.string()).min(5);
z.array(z.string()).max(5);
z.array(z.string()).length(5);
```

튜플:

```ts
const MyTuple = z.tuple([z.string(), z.number(), z.boolean()]);
```

가변 길이(rest) 지원:

```ts
const variadicTuple = z.tuple([z.string()], z.number());
// => [string, ...number[]]
```

---

### 유니언 & 인터섹션 (Unions & Intersections)

Union (A | B):

```ts
const stringOrNumber = z.union([z.string(), z.number()]);
```

Discriminated Union:

```ts
const MyResult = z.discriminatedUnion("status", [
  z.object({ status: z.literal("success"), data: z.string() }),
  z.object({ status: z.literal("failed"), error: z.string() }),
]);
```

Intersection (A & B):

```ts
const EmployedPerson = z.intersection(Person, Employee);
```

---

### 레코드 / 맵 / 셋 (Records / Maps / Sets)

Record:

```ts
const IdCache = z.record(z.string(), z.string());
```

Map:

```ts
const StringNumberMap = z.map(z.string(), z.number());
```

Set:

```ts
const NumberSet = z.set(z.number());
z.set(z.string()).min(5);
```

---

### 함수 (Functions)

Zod 함수 스키마:

```ts
const MyFunction = z.function({
  input: [z.string()],
  output: z.number(),
});
```

`.implement()` 사용:

```ts
const computeTrimmedLength = MyFunction.implement(
  (input) => input.trim().length
);
```

---

### 세분화 검증 (Refinements)

사용자 정의 검증:

```ts
z.string().refine((val) => val.length <= 255);
```

옵션:

- error -> 에러 메시지
- abort -> 실패 시 즉시 중단
- path -> 에러 경로 지정

비동기 검증도 가능 (parseAsync 필수).
고급: .superRefine(), .check()

---

### 변환 (Transforms)

단방향 변환:

```ts
z.string().transform((val) => val.length);
```

비동기 변환도 가능:

```ts
z.string().transform(async (id) => db.getUserById(id));
```

사전처리:

```ts
z.preprocess((val) => Number.parseInt(String(val)), z.int());
```

---

### 기본값 / 프리폴트 / 캐치 (Defaults / Prefaults / Catch)

기본값:

```ts
z.string().default("default");
```

사전 기본값 (prefault):

```ts
z.string().trim().toUpperCase().prefault(" tuna ");
```

에러 시 대체값:

```ts
z.number().catch(42);
```

---

### 브랜드 타입 (Branded types)

명목 타입 시뮬레이션:

```ts
const Cat = z.object({ name: z.string() }).brand<"Cat">();
```

---

### 읽기 전용 (Readonly)

불변 스키마:

```ts
z.object({ name: z.string() }).readonly();
z.array(z.string()).readonly();
```

결과는 `Object.freeze()`로 처리됨.

---

### JSON (JSON)

JSON 전체 값 검증:

```ts
const jsonSchema = z.json();
```

---

### 사용자 정의 타입 (Custom)

사용자 정의 타입 스키마:

```ts
const px = z.custom<`${number}px`>((val) => /^\d+px$/.test(val));
```

---

### 템플릿 리터털 (Template literals)

```ts
z.templateLiteral(["hello, ", z.string(), "!"]);
```

---

### 코덱 (Codecs)

양방향 변환 스키마:

```ts
const stringToDate = z.codec(z.iso.datetime(), z.date(), {
  decode: (iso) => new Date(iso),
  encode: (date) => date.toISOString(),
});
```

---

### 파이프 (Pipes)

스키마 체이닝:

```ts
const stringToLength = z.string().pipe(z.transform((val) => val.length));
```
