# Awaited và ReturnType

- nếu ta có một hàm promise và chưa biết hắn sẽ trả về gì, nhưng ta cần định nghĩa nó để định nghĩa có biến sẽ nhận kết quả trả về của promise đó ta có thể dùng Awaited kết hợp với ReturnType và typeOf

  ```ts
  type GuestInfoPromise = ReturnType<typeof fetchGuestInfo>;
  type CheckoutPromise = ReturnType<typeof processCheckout>;

  async function checkoutGuest(guestId: number): Promise<void> {
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
