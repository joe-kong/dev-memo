# React コード規約とベストプラクティス

## リンティングと自動フォーマッターの使用

React の主要なツールはすべてリンティングルールを提供しています。必要に応じて自分のスタイルに合わせて編集しても構いませんが、必ず何らかのルールを使用し、リンティングとフォーマットのプロセスを自動化してください。推奨ツールは eslint と prettier です。

## インポート順序

eslint 設定にインポート順序のルールを追加してください。これにより、インポートが常に同じ順序で、事前定義されたルールによってグループ化されることが保証されます。

例：

```javascript
{
  'import/order': [
    2,
    {
      'newlines-between': 'always',
      groups: [
        'builtin',
        'external',
        'internal',
        'parent',
        'sibling',
        'index',
        'unknown',
        'object',
        'type',
      ],
      alphabetize: {
        order: 'asc',
        caseInsensitive: true,
      },
      pathGroups: [
        {
          pattern: 'react*',
          group: 'external',
          position: 'before',
        },
      ],
    },
  ],
}
```

## 命名規則

- **コンポーネント、インターフェース、型エイリアスには PascalCase を使用**
- **変数、配列、オブジェクト、関数などの JavaScript データ型には camelCase を使用**
- **フォルダ名と非コンポーネントファイル名には camelCase、コンポーネントファイル名には PascalCase を使用**

```
src/utils/form.ts
src/hooks/useForm.ts
src/components/banners/edit/Form.tsx
```

## TypeScript バレルの使用を推奨

バレルは、複数のモジュールからのエクスポートを単一の便利なモジュールにまとめる方法です。

このアプローチは、共有コンポーネント、ユーティリティ、ヘルパー関数などのエンティティに適用できます。作業を楽にするために、barrelsby を使用してバレルを自動生成できます。

## デフォルトエクスポートを避ける

デフォルトエクスポートは、エクスポートされるアイテムに名前を関連付けないため、インポート時に任意の名前を適用できます。これは柔軟性を提供するかもしれませんが、複数の開発者が同じモジュールを異なる名前や記述的でない名前でインポートした場合、問題が発生します。

名前付きエクスポートは明示的で、コンシューマーに元の作者が意図した名前でのインポートを強制し、曖昧さを取り除きます。

## コンポーネント構造

すべてのコンポーネントファイルの一貫性を保つため、以下のパターンに従ってください：

```typescript
// 1. インポート - 記述量を最小化するために分割代入インポートを推奨
import React, { PropsWithChildren, useState, useEffect } from "react";

// 2. 型定義
type ComponentProps = {
  someProperty: string;
};

// 3. スタイル - @mui を使用する場合は styled API またはコンポーネントの sx プロップを使用
const Wrapper = styled("div")(({ theme }) => ({
  color: theme.palette.white
}));

// 4. 追加変数
const SOME_CONSTANT = "something";

// 5. コンポーネント
function Component({ someProperty }: PropsWithChildren<ComponentProps>) {
  // 5.1 定義
  const [state, setState] = useState(true);
  const { something } = useSomething();

  // 5.2 関数
  function handleToggleState() {
    setState(!state);
  }

  // 5.3 エフェクト
  useEffect(() => {
    // ...
  }, []);

  // 5.5 追加の分割代入
  const { property } = something;

  return (
    <Wrapper>
      <div>
        <p>Lorem ipsum</p>
        <p>Pellentesque arcu</p>
      </div>

      <p>Lorem ipsum</p>
      <p>Pellentesque arcu</p>
    </Wrapper>
  );
}

// 6. エクスポート
export { Component };
export type { ComponentProps };
```

### PropsWithChildren の使用

```typescript
import React, { PropsWithChildren } from "react";

type ComponentProps = {
  someProperty: string;
};

// ✅
function Component({ someProperty, children }: PropsWithChildren<ComponentProps>) {
  // ...
}
```

### 1行を超える場合は関数を JSX から分離

```typescript
// ❌
<button
  onClick={() => {
    setState(!state);
    resetForm();
    reloadData();
  }}
/>

// ✅
<button onClick={() => setState(!state)} />

// ✅
const handleButtonClick = () => {
  setState(!state);
  resetForm();
  reloadData();
}

<button onClick={handleButtonClick} />
```

### インデックスをキープロップとして使用することを避ける

```typescript
// ❌
const List = () => {
  const list = ["item1", "item2", "item3"];

  return (
    <ul>
      {list.map((value, index) => {
        return <li key={index}>{value}</li>;
      })}
    </ul>
  );
};

// ✅
const List = () => {
  const list = [
    { id: "111", value: "item1" },
    { id: "222", value: "item2" },
    { id: "333", value: "item3" }
  ];

  return (
    <ul>
      {list.map((item) => {
        return <li key={item.id}>{item.value}</li>;
      })}
    </ul>
  );
};
```

### フラグメントを使用

```typescript
// ❌
const ActionButtons = ({ text1, text2 }) => {
  return (
    <div>
      <button>{text1}</button>
      <button>{text2}</button>
    </div>
  );
};

// ✅
const Button = ({ text1, text2 }) => {
  return (
    <>
      <button>{text1}</button>
      <button>{text2}</button>
    </>
  );
};
```

### プロパティの分割代入を推奨

コンポーネントで使用されるプロパティが明確になるように、プロパティの分割代入を推奨します。

```typescript
// ❌
const Button = (props) => {
  return <button>{props.text}</button>;
};

// ✅
const Button = (props) => {
  const { text } = props;

  return <button>{text}</button>;
};

// ✅
const Button = ({ text }) => {
  return <button>{text}</button>;
};
```

## 関心の分離

ビジネスロジックをプレゼンテーション（レンダリング）から分離することで、コンポーネントのコードがより読みやすくなります。これは、複数のフックや useEffect を使用するページ/スクリーン/コンテナコンポーネントに最も適用されます。そうすると、最終的なコードが読みにくくなるほど巨大になり始めます。

## カスタムフック

責任を分離するために、まず useEffect や複数の useState をコンポーネントに直接配置するのではなく、カスタムフックを作成する必要があります。

```typescript
// ❌
const ScreenDimensions = () => {
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined
  });

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }

    window.addEventListener("resize", handleResize);
    handleResize();
    
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return (
    <>
      <p>Current screen width: {windowSize.width}</p>
      <p>Current screen height: {windowSize.height}</p>
    </>
  );
};

// ✅
const useWindowSize = () => {
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined
  });

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }

    window.addEventListener("resize", handleResize);
    handleResize();

    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return windowSize;
};

const ScreenDimensions = () => {
  const windowSize = useWindowSize();

  return (
    <>
      <p>Current screen width: {windowSize.width}</p>
      <p>Current screen height: {windowSize.height}</p>
    </>
  );
};
```

*参照: https://usehooks.com/useWindowSize/*

## 高階関数としてのコンポーネントコントローラー

コンポーネントが状態やカスタムフックで過負荷になったら、コントローラーを作成する時です。

**ヘルパー関数 — utils/wrap.ts**

```typescript
import { FC, createElement } from 'react';

export const wrap = <Props extends object, ViewProps extends object>(
  View: FC<Partial<ViewProps>>,
  controllers: Array<(props: Props) => Partial<ViewProps> | null | void>
) => (args: Props) =>
  createElement(
    View,
    ...controllers
      .map((useController) => useController(args) as Partial<ViewProps> | null)
      .filter(Boolean)
  );
```

**ページコントローラー — useTodoController.ts**

```typescript
import { useMemo } from 'react';

import { useTodoList } from '../../adapters/todoAdapter';
import { useUserData } from '../../adapters/userAdapter';

export const useTodoController = () => {
  const { user } = useUserData();

  const {
    todos: { isLoading, data },
  } = useTodoList(user.id);

  return useMemo(() => ({ isLoading, todos: data }), [isLoading, data]);
};
```

**ページ — TodoList.tsx**

```typescript
import { useTodoController } from './useTodoController';
import { TodoItem } from './components/TodoItem';

import { wrap } from '../../utils/wrap';

import { Todo, TodoList as TodoListType } from '../../../domain/struct/todo';

type TodoListProps = {
  todos: TodoListType;
  isLoading: boolean;
};

const TodoListComponent = ({ todos, isLoading }: TodoListProps) => {
  return isLoading ? (
    <>Loading...</>
  ) : (
    <>
      <ul className="tasks-list">
        {todos.map((todo: Todo) => (
          <TodoItem key={todo.id} todo={todo} />
        ))}
      </ul>
    </>
  );
};

export const TodoList = wrap(TodoListComponent, [useTodoController]);
```

## 巨大なコンポーネントを避ける

可能な限り、コンポーネントをより小さなチャンクに分割してください。これは、条件付きレンダリングを使用している場合やデータグリッドの列を定義している場合などによく適用されます。また、多くのカスタムフックが使用されている場合にも適用されます。

```typescript
// ❌
const SomeSection = ({ isEditable, value }) => {
  if (isEditable) {
    return (
      <Section>
        <Title>Edit this content</Title>
        <Content>{value}</Content>
        <Button>Clear content</Button>
      </Section>
    );
  }

  return (
    <Section>
      <Title>Read this content</Title>
      <Content>{value}</Content>
    </Section>
  );
};

// ✅
const EditableSection = ({ value }) => {
  return (
    <Section>
      <Title>Edit this content</Title>
      <Content>{value}</Content>
      <Button>Clear content</Button>
    </Section>
  );
};

const DetailSection = ({ value }) => {
  return (
    <Section>
      <Title>Read this content</Title>
      <Content>{value}</Content>
    </Section>
  );
};

const SomeSection = ({ isEditable, value }) => {
  return isEditable ? (
    <EditableSection value={value} />
  ) : (
    <DetailSection value={value} />
  );
};
```

## 可能な限り状態をグループ化

```typescript
// ❌
const [username, setUsername] = useState('')
const [password, setPassword] = useState('')

// ✅
const [user, setUser] = useState({})
```

## ブール値プロップには短縮形を使用

```typescript
// ❌
<Form hasPadding={true} withError={true} />

// ✅
<Form hasPadding withError />
```

## 文字列プロップには中括弧を避ける

```typescript
// ❌
<Title variant={"h1"} value={"Home page"} />

// ✅
<Title variant="h1" value="Home page" />
```

## インラインスタイルの使用を避ける

理想的には、CSS モジュール、JSS、styled-components などのスタイル処理用のサードパーティを使用するか、カスタムフックを作成してください。

```typescript
// ❌
const Title = (props) => {
  return (
    <h1 style={{ fontWeight: 600, fontSize: '24px' }} {...props}>
      {children}
    </h1>
  );
};

// ✅
const useStyles = (props) => {
  return useMemo(
    () => ({
      header: { fontWeight: props.isBold ? 700 : 400, fontSize: "24px" }
    }),
    [props]
  );
};

const Title = (props) => {
  const styles = useStyles(props);

  return (
    <h1 style={styles.header} {...props}>
      {children}
    </h1>
  );
};
```

## 三項演算子を使用した条件付きレンダリングを推奨

```typescript
const { role } = user;

// ❌
if (role === ADMIN) {
  return <AdminUser />;
} else {
  return <NormalUser />;
}

// ✅
return role === ADMIN ? <AdminUser /> : <NormalUser />;
```

## 文字列値には定数や列挙型を使用

```typescript
// ❌
if (role === 'admin') {
  return <AdminUser />;
}

// ✅
enum Roles {
  admin = 'admin',
  basic = 'basic',
}

if (role === Roles.admin) {
  return <AdminUser />;
}
```

## 型エイリアスを使用

```typescript
export type TodoId = number;
export type UserId = number;

export interface Todo {
  id: TodoId;
  name: string;
  completed: boolean;
  userId: UserId;
}

export type TodoList = Todo[];
```

## サードパーティライブラリを直接使用しない

サードパーティライブラリを直接インポートする代わりに、一元化された場所で再エクスポートしてください。

```typescript
// src/lib/store.ts
export { useDispatch, useSelector } from 'react-redux';

// src/lib/query.ts
export { useQuery, useMutation, useQueryClient } from 'react-query';
```

## 実装ではなく抽象化に依存

```typescript
// ❌ moment を直接使用
import moment from 'moment';

const updateProduct = (product) => {
  const payload = {
    ...product,
    // ❌ moment インターフェースの実装に依存している
    updatedAt: moment().toDate(),
  };

  return await fetch(`/product/${product.id}`, {
    method: 'PUT',
    body: JSON.stringify(payload),
  });
};

// ✅ 抽象化（ヘルパー関数）を作成して機能をラップ

// utils/createDate.ts
import moment from 'moment';

export const createDate = (): Date => moment().toDate();

// updateProduct.ts
import { createDate } from './utils/createDate';

const updateProduct = (product) => {
  const payload = {
    ...product,
    // ✅ 抽象化されたヘルパー関数を使用
    updatedAt: createDate(),
  };

  return await fetch(`/product/${product.id}`, {
    method: 'PUT',
    body: JSON.stringify(payload),
  });
};
```

## 宣言的プログラミングを推奨

```typescript
// ❌ 命令的：配列反復の内部処理に対処
const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
let sum = 0;

for (let i = 0; i < arr.length; i++) {
  sum += arr[i];
}

// ✅ 宣言的：反復の内部処理に対処しない
const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
const sum = arr.reduce((acc, v) => acc + v, 0);
```

## 説明的な変数名を使用

```typescript
// ❌ 単一文字の名前を避ける
const n = 'Max';
// ✅
const name = 'Max';

// ❌ 略語を避ける
const sof = 'Sunday';
// ✅
const startOfWeek = 'Sunday';

// ❌ 無意味な名前を避ける
const foo = false;
// ✅
const appInit = false;
```

## 長い関数引数リストを避ける

```typescript
// ❌
function createPerson(firstName, lastName, height, weight, gender) {
  // ...
}

// ✅
function createPerson({ firstName, lastName, height, weight, gender }) {
  // ...
}

// ✅
function createPerson(person) {
  const { firstName, lastName, height, weight, gender } = person;
  // ...
}
```

## オブジェクトの分割代入を使用

```typescript
// ❌
return (
  <>
    <div> {user.name} </div>
    <div> {user.age} </div>
    <div> {user.profession} </div>
  </> 
)

// ✅
const { name, age, profession } = user;

return (
  <>
    <div> {name} </div>
    <div> {age} </div>
    <div> {profession} </div>
  </> 
)
```

## テンプレートリテラルの使用を推奨

```typescript
// ❌
const userName = user.firstName + " " + user.lastName;

// ✅
const userDetails = `${user.firstName} ${user.lastName}`;
```

## 小さな関数では暗黙的な戻り値を使用

```typescript
// ❌
const add = (a, b) => {
  return a + b;
}

// ✅
const add = (a, b) => a + b;
```

---

*コンサルテーションに感謝：Jakub Kuřitka*