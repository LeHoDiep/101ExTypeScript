# Awaited và ReturnType

- RetyrnType\<Type> giúp ta lấy ra kiểu dữ liệu trả về của một hàm

  - vị dụ hàm đó có rất nhiều overload

  ```ts
  //vd1:
  //stringOrNum có rất nhiều overload
  declare function stringOrNum(x: string): number;
  declare function stringOrNum(x: number): string;
  declare function stringOrNum(x: string | number): string | number;

  type T1 = ReturnType<typeof stringOrNum>;
  //   ^? type T1 = string | number
  ```

  - nhưng nếu

  ```ts
  //stringOrNum có rất nhiều overload
  declare function stringOrNum(x: string): number;
  declare function stringOrNum(x: string | number): string | number;
  declare function stringOrNum(x: number): string;

  type T1 = ReturnType<typeof stringOrNum>;
  //   ^? type T1 = string
  //vì inferring(suy luận) từ nhiều overload nó sẽ trả về
  // overload cuối cùng
  ```

  - ví dụ nâng cao

  ```ts
  type T3 = ReturnType<<T extends U, U extends number[]>() => T>;

  type T3 = number[];
  ```

- nếu ta có một hàm promise và chưa biết hắn sẽ trả về gì, nhưng ta cần định nghĩa nó để định nghĩa có biến sẽ nhận kết quả trả về của promise đó ta có thể dùng Awaited kết hợp với ReturnType và typeOf

  ```ts
  //dùng typeof để lấy type của hàm fetchGuestInfo
  //ReturnType để lấy giá trị trả ra
  type GuestInfoPromise = ReturnType<typeof fetchGuestInfo>;
  type CheckoutPromise = ReturnType<typeof processCheckout>;

  async function checkoutGuest(guestId: number): Promise<void> {
    //định nghĩa rằng guestInfo sẽ nhận giá trị trả ra của fetchGuestInfo
    const guestInfo: Awaited<GuestInfoPromise> = await fetchGuestInfo(guestId);
    console.log("Guest information fetched:", guestInfo);

    await processCheckout(guestId);

    console.log("Checkout completed for guest:", guestId);
  }

  async function fetchGuestInfo(guestId: number): Promise<string> {
    return new Promise<string>((resolve) => {
      setTimeout(() => {
        resolve(`Guest ${guestId}: John Doe`);
      }, 2000);
    });
  }

  async function processCheckout(guestId: number): Promise<void> {
    return new Promise<void>((resolve) => {
      setTimeout(() => {
        resolve();
      }, 1500);
    });
  }

  const guestId: number = 123;
  checkoutGuest(guestId)
    .then(() => {
      console.log("Guest has successfully checked out!");
    })
    .catch((error) => {
      console.error("An error occurred during checkout:", error);
    });
  ```

- trong ví dụ trên, tôi lấy kiểu của promise GuestInfoPromise bằng ReturnType<typeof fetchGuestInfo>
- và dùng Awaited để định nghĩa giá trị trả ra nếu await fetchGuestInfo, một điều thú vị là dù có promise nhiều lớp thì
  Awaited<GuestInfoPromise> vẫn định nghĩa là **string**

  ```ts
  type A = Awaited<Promise<string>>;
  //   ^? type A = string

  type B = Awaited<Promise<Promise<number>>>;
  //   ^? type B = number; dù có lồng nhiều lớp

  type C = Awaited<boolean | Promise<number>>;
  //   ^? type C = boolean | number;
  ```

# Partial\<Type>

- Partial\<Type> giúp ta tạo ra một type mới từ type đã có, nhưng tất cả các thuộc tính của type đó đều là optional

- ứng dụng trong việc update prop của một object, nhưng chỉ
  cho phép update một số prop đã định nghĩa trước đó

      ```ts
      //định nghĩa 1 Todo
      interface Todo {
      title: string;
      description: string;
      }
      //tạo ra 1 object có dạng Todo
      const todo1: Todo = {
      title: "organize desk",
      description: "clear clutter",
      };

      //viết hàm updateTodo nhận vào 1 object dạng Todo và 1 object dạng Todo nhưng các prop là optional
      function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
      return { ...todo, ...fieldsToUpdate };
      }
      //hàm trả về 1 object mới có các prop đã được update

      const todo2 = updateTodo(todo1, {
      description: "throw out trash",
      fname: "Hùng", //prop này không tồn tại trong Todo
      }); //nên sẽ báo lỗi
      ```

# Required\<Type>

- Ngược với Partial, Required\<Type> giúp ta tạo ra một type mới từ type đã có, nhưng tất cả các thuộc tính của type đó đều là bắt buộc

- ứng dụng trong việc nếu như user đó đầy đủ thông tin thì mới cho phép tạo mới user đó

```ts
interface User {
  id: number;
  name?: string;
  email?: string;
}

// Tạo một phiên bản mới của User với tất cả các thuộc tính là bắt buộc
type RequiredUser = Required<User>;
//RequiredUser{
//    id: number;
//    name: string;
//    email: string;
//}

// Hàm createUser chỉ chấp nhận đối tượng user có đủ thông tin
function createUser(user: RequiredUser): void {
  console.log("User ID:", user.id);
  console.log("User Name:", user.name);
  console.log("User Email:", user.email);
}

//**********use nè********

//ta có thể dùng type mới để tạo ra 1 object mới
const newUser: RequiredUser = {
  id: 1, //thiếu dòng nào cũng lỗi
  name: "John Doe", //thiếu dòng nào cũng lỗi
  email: "john@example.com", //thiếu dòng nào cũng lỗi
};

// Gọi hàm createUser với đối tượng newUser
createUser(newUser);
```

# Readonly\<Type>

- Readonly<Type> giúp ta tạo ra một type mới từ type đã có, nhưng tất cả các thuộc tính của type đó đều là chỉ đọc

- ứng dụng trong việc tạo ra một object mà không cho phép thay đổi giá trị của các thuộc tính

- có ích để tạo ra các prop hằng số, tránh việc người dùng thay đổi giá trị của prop

```ts
interface Todo {
  title: string;
  jobDs: {
    name: string;
    address: string;
  };
}

//todo là object của type Todo những đã bị readonly
const todo: Readonly<Todo> = {
  title: "Delete inactive users",
  jobDs: {
    name: "Dev",
    address: "115/13",
  },
};

todo.jobDs.name = "worker"; //vẫn đổi được
todo.title = "Hello"; //bug
```

trong js ta có flag Object.freeze() và Readonly\<Type> để đóng băng object, nhưng nó chỉ đóng băng ở mức độ 1 cấp

# Record<Keys, Type>

- Record<Keys, Type> giúp ta tạo ra một type mới có các
  thuộc tính được định nghĩa bởi Keys và Type với số lượng không xác định

- ứng dụng trong việc tạo ra một object với các key và value có kiểu dữ liệu cố định, và rất nhiều prop như vậy

  ```ts
  interface DogInfor {
    type: string;
    color: string;
  }

  type DogName = string;

  const cats: Record<DogName, DogInfor> = {
    miffy: { type: "Beggie", color: "while" },
    boris: { type: "Chiquaqua", color: "Brown" },
    mordred: { type: "Dalmation", color: "While" },
    cody: { type: 123, color: "While" }, //bug
  };
  ```

# Pick<Type, Keys>

- Pick<Type, Keys> giúp ta tạo ra một type mới từ type đã có, nhưng chỉ chứa các thuộc tính được định nghĩa bởi Keys

```ts
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// UserSubset là User nhưng chỉ chứa id và name
type UserSubset = Pick<User, "id" | "name">;

// Sử dụng type UserSubset
const userSubset: UserSubset = {
  id: 1,
  name: "John Doe",
  age: 30, //bug
};
```

# Omit<Type, Keys>

- ngược lại với Pick, Omit<Type, Keys> giúp ta tạo ra một type mới từ type đã có, nhưng loại bỏ các thuộc tính được định nghĩa bởi Keys

  ```ts
  interface Todo {
    title: string;
    description: string;
    completed: boolean;
    createdAt: number;
  }

  type TodoInfo = Omit<Todo, "completed" | "createdAt">;

  const todoInfo: TodoInfo = {
    title: "Pick up kids",
    description: "Kindergarten closes at 5pm",
    completed: false, //bug
    createdAt: 1615544252770, //bug
  };
  ```

# Exclude<UnionType, ExcludedMembers>

- khá giống Omit nhưng dùng cho union type
  - Omit dùng để loại bỏ prop của object
  - Exclude dùng để loại bỏ **union type**

```ts
//ta có một union type
type basic = "a" | "b" | "c" | "d";

type basic = "a" | "b" | "c" | "d";
type basicC = Exclude<basic, "a" | "b">;
//vậy thì type basicC  =  | "c" | "d"

//vd2:
type T2 = Exclude<string | number | (() => void), Function>;
//T2 = string | number

//vd3: loại bỏ bằng một mô tả cụ thể
type Shape =
  | { kind: "circle"; radius: number } //bị loại
  | { kind: "square"; x: number }
  | { kind: "triangle"; x: number; y: number };

type T3 = Exclude<Shape, { kind: "circle" }>;
```

# Extract<Type, Union>

- Extract<Type, Union> giúp ta tạo ra một type mới từ type đã có, nhưng chỉ chứa các thuộc tính được định nghĩa bởi Union
- **cũng dùng với union type**
- có thể sử dụng như filter lọc ra các phần tử giống nhau
  hoặc thỏa mãn điều kiện

```ts
type T0 = Extract<"a" | "b" | "c", "a" | "b" | "f">;
//ta có 2 union type, Extract sẽ lấy ra các phần tử giống nhau
//T0 = "a" | "b"

type Shape =
  | { kind: "circle"; radius: number } //bị loại
  | { kind: "square"; x: number }
  | { kind: "triangle"; x: number; y: number };

type T3 = Extract<Shape, { kind: "square" } | { y: number }>;

/*
type T3 = {
    kind: "square";
    x: number;
} | {
    kind: "triangle";
    x: number;
    y: number;
}
*/
```

# NonNullable<Type>

- NonNullable<Type> sẽ loại bỏ các giá trị null từ type
- nếu type là union type thì sẽ loại bỏ null từ union type

```ts
type T0 = NonNullable<string | number | undefined>;
//  type T0 =  string | number
type T1 = NonNullable<string[] | null | undefined>;
//   type T1 = string[]

let possiblyNull: string | null = "Hello";
possiblyNull.length; //not good
// Error: Object is possibly 'null'.
```

- nếu mình biết chắc giá trị trả ra k thể null thì nên dùng NonNullable

```ts
let definitelyNotNull: NonNullable<typeof possiblyNull> = possiblyNull;
//possiblyNull: string = "Hello"
possiblyNull.length; //good
```

# Parameters\<Type>

- Parameters\<Type> giúp ta tạo ra một type mới từ type đã có, nhưng chỉ chứa các tham số của hàm
  - ReturnType\<Type> giúp ta tạo ra một type mới từ type đã có, nhưng chỉ chứa giá trị trả về của hàm

```ts
type T0 = Parameters<() => string>;
//   type T0 = []

type T1 = Parameters<(s: string) => void>;
//   type T1 = [s: string]

type T2 = Parameters<<T>(arg: T) => T>;
//   type T2 = [arg: unknown]
// vì T là kiểu dữ liệu chưa biết

declare function f1(arg: { a: number; b: string }): void;
type T3 = Parameters<typeof f>;
//   type T3 = [{ a: number; b: string }]

//any thì xem paramater là 1 mảng các unknown
type T4 = Parameters<any>;
//type T4 = unknown[];

type T5 = Parameters<never>;
//   type T5 = never

type T6 = Parameters<string>; // sai

type T7 = Parameters<Function>; // sai
```

## Nâng cao với Parameters\<Type>

(https://www.typescriptlang.org/docs/handbook/2/conditional-types)

- Inferring Within Conditional Types

# InstanceType\<Type>

- instance là một object được tạo ra từ class

- InstanceType\<Type> giúp ta tạo ra một type mới từ constructor của class, hoặc type của class

- ứng dụng trong việc tạo ra type để định
  nghĩa 1 instance được tạo ra từ class

```ts
//v1
class C {
  x = 0;
  y = 0;
}

type T0 = InstanceType<typeof C>;

//    ^?T0 = {
//          x = 0;
//          y = 0;
//     }
type T1 = InstanceType<any>;
//    ^?T1 = any
type T2 = InstanceType<never>;
//    ^?T2 = never
type T3 = InstanceType<string>;
//    ^? bug
type T4 = InstanceType<Function>;
//    ^? bug

//ứng dụng
// ta có 1 class chuyên tạo ra các
//instance(object) giúp kết nối mongoDB theo url
class MongoDBConnection {
  constructor(private url: string) {}

  connect() {
    //giả bộ có code
  }

  close() {
    //giả bộ có code
  }
}

//tạo type cho instance của class MongoDBConnection
type MongoDBConnectionInstance = InstanceType<typeof MongoDBConnection>;

//khi dùng sẽ là
const connectionA: MongoDBConnectionInstance = new MongoDBConnection(
  "mongodb://localhost:27017/mydb"
);
connection.connect();

// connectionA đã đc định nghĩa
```

# ThisParameterType\<Type>
