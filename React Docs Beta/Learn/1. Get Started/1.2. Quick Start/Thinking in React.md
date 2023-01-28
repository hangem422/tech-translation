# React에서 사고하기

React는 디자인을 바라보고 앱을 구현하는 것에 대한 당신의 사고를 바꿀 수 있습니다. React로 유저 인터페이스를 구현할 때, 우선적으로 이것을 컴포넌트 조각으로 분해할 겁니다. 그리고, 컴포넌트들의 각기 다른 외관 상태를 표현할 겁니다. 마지막으로, 데이터의 흐름에 따라 컴포너트들을 연결할 것입니다. 이번 튜토리얼에서, React로 검색 가능한 제품 데이터 테이블을 구현하는 사고의 순서에 대해 알려드리도록 하겠습니다.

## 목업으로 시작하기

이미 JSON API를 가지고 있고, 디자이너로부터 목업을 받았다고 상상하세요. JSON API는 다음과 같은 형테의 데이터를 반환합니다.

```json
[
  { "category": "Fruits", "price": "$1", "stocked": true, "name": "Apple" },
  { "category": "Fruits", "price": "$1", "stocked": true, "name": "Dragonfruit" },
  { "category": "Fruits", "price": "$2", "stocked": false, "name": "Passionfruit" },
  { "category": "Vegetables", "price": "$2", "stocked": true, "name": "Spinach" },
  { "category": "Vegetables", "price": "$4", "stocked": false, "name": "Pumpkin" },
  { "category": "Vegetables", "price": "$1", "stocked": true, "name": "Peas" }
]
```

디자인 목업은 다음과 같습니다.

![Design Mockup](https://beta.reactjs.org/images/docs/s_thinking-in-react_ui.png)

## Step 1: 컴포넌트 계층 구조로 UI 분해하기

목업의 모든 컴포넌트와 서브 컴포넌트 주위에 박스를 그리고 이름을 붙이며 시작하세요. 디자이너와 함께 일한다면, 이미 디자인 툴에 이 컴포넌트들의 이름을 지어놨을 수도 있습니다. 함께 확인해보세요!

환경에 따라서, 디자인을 컴포넌트로 쪼개는 방법을 다향하게 생각해 볼 수 있습니다.

- Programming - 새로운 함수 혹은 객체를 만들어야 하는지 결정하는데 동일한 기술을 사용하세요. 그 그술 중 하나는 [단일 책임 원칙](https://en.wikipedia.org/wiki/Single_responsibility_principle)이며, 이상적으로 컴포넌트는 단 한가지 일만 한다는 것을 의미합니다. 만약 컴포넌트가 커진다면, 더 작은 사이즈의 서브 컴포넌트로 분해돼야 합니다.
- CSS - 어떤 것을 위해서 클래스 선택자를 만드는지 고민하세요. (반면, 컴포넌트는 CSS보다 다소 덜 세분화 돼있습니다.)
- Design - 디자인 계층을 어떻게 조직화할지 고민하세요.

만약 JSON이 잘 만들어져있다면, UI의 컴포넌트 구조와 자연스럽게 맵핑되는 경우가 많습니다. UI와 데이터 모델은 동일할 정보 아키텍처를 갖는 경우가 많이 때문입니다. 즉, 모양이 같습니다. UI를 데이터 모델의 일부와 부합하는 컴포넌트로 분해하세요.

다음 화면에는 5개의 컴포넌트가 있습니다.

![FiveComponent](https://beta.reactjs.org/images/docs/s_thinking-in-react_ui_outline.png)

1. `FilterableProductTable`(회색)은 전체적인 앱을 담고 있습니다.
2. `SearchBar`(파랑)은 유저의 입력을 받습니다.
3. `ProductTable`(라벤더)는 유저의 입력에 따라 리스트를 필터링하고 출력합니다.
4. `ProductCategoryRow`(초록)은 각 카테고리의 제목을 출력합니다.
5. `ProductRow`(노랑)은 각 제품을 한 행으로 출력합니다.

`ProductTable`를 보면, 테이블의 헤더("Name"과 "Price" 라벨을 담고 있는)가 독립된 컴포넌트를 갖지 않는 것을 알 수 있습니다. 이건 취향의 문제이며, 어떻게하던 상관 없습닙다. 이번 예제에서, 이건 `ProductTabl`의 리스트 내부에 출력되기 때문에 `ProductTable`의 일부입니다. 그러나, 추후에 이 헤더의 복잡도가 커진다면, `ProductTableHeader` 컴포넌트로 독립 시키는 것이 좋습니다.

이제 목업 내부의 컴포넌트들을 정의했으므로, 이들의 계층을 정리하세요. 목업에서 다른 컴포넌트 내부에 들어가는 컴포넌트는 계층 구조에서 자식으로 표현돼야 합니다.

- `FilterableProductTable`
  - `SearchBar`
  - `ProductTable`
    - `ProductCategoryRow`
    - `ProductRow`

## Step 2: React로 정적인 버전 구현하기

컴포넌트 계층을 만들었으니, 이제 앱을 구현할 차례입니다. 가장 정석적인 접근은 일단 데이터 모델을 기반으로 어떤 상호작용도 추가되지 않은 UI를 렌더하는 버전을 구현하는 겁니다. 일반적으로 처음에는 정적인 버전을 처음으로 구현하고 부분적으로 상호작용을 추가하는게 쉽습니다. 정적 버전 구현은 많은 양의 타이핑이 필요하고 생각은 필요하지 않지만, 상호작용을 추가하는 것은 타이핑은 적고 생각은 많이 필요합니다.

데이터 모델을 렌더하는 정적인 버전의 앱을 구현하기 위해서는, 다른 컴포넌트들을 재활용하고 props를 통해 데이터를 넘겨주는 컴포넌트를 구현할 필요가 있습니다. props는 부모에서 자식으로 데이터를 전달하는 방법입니다. (만약 state 개념에 익숙하다면, 정적 버전 구현에서는 state를 일체 사용하지 마세요. State는 상호 작용, 즉 시간에 따라 변하는 데터를 위해 남겨둡니다. 이번에는 앱의 정적인 버전이므로, 이게 필요하지 않습니다.)

계층 구조의 상층부(`FilterableProductTable` 같은) 컴포넌트부터 구현을 시작하는 "탑 다운"과 하층부(`ProductRow` 같은) 컴포넌트부터 시작하는 "바텀 업"중 어떤 것을 사용해도 무관합니다. 단순한 예시에서는 탑 다운 방식이 쉽고, 큰 프로젝트에서는 바텀 업 방식이 쉽습니다.

```tsx
interface Product {
  category: string;
  price: string;
  stocked: boolean;
  name: string;
}

function ProductCategoryRow({ category }: Pick<Product, "category">) {
  return (
    <tr>
      <th colSpan="2">{category}</th>
    </tr>
  );
}

function ProductRow({ product }: { product: Product }) {
  const name = product.stocked ? product.name : <span style={{ color: "red" }}>{product.name}</span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products }: { products: Array<Product> }) {
  const rows: Array<ReactNode> = [];
  let lastCategory: string | null = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(<ProductCategoryRow category={product.category} key={product.category} />);
    }
    rows.push(<ProductRow product={product} key={product.name} />);
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label>
        <input type="checkbox" /> Only show products in stock
      </label>
    </form>
  );
}

function FilterableProductTable({ products }: { products: Array<Product> }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}

const PRODUCTS: Array<Product> = [
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" },
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

컴포넌트를 구현한하고 나면, 데이터 모델을 랜더하는 재사용 가능한 컴포넌트 라이브러리를 갖게 됩니다. 이건 정적인 앱이기 떄문에, 컴포넌트는 단순히 JSX를 반환하기만 합니다. 계층의 최상부 컴포넌트(`FilterableProductTable`)는 데이터 모델을 props로 전달 받을 겁니다. 데이터가 트리의 상단 컴포넌트에서 하단 컴포넌트중 하나로 흘러 내려가기 때문에 이것을 _단방향 데이터 흐름_(one-way data flow)라 부릅니다.

> ### 주의 사항
>
> 이 시점에서, state 값을 사용하면 안됩니다. state는 다음 단계를 위한 것 입니다.

## Step 3: 작지만 완변한 UI 상태 표현 찾기

UI를 상호 작용 가능하게 만들기 위해서 내제된 데이터 모델을 유저가 변경할 수 있도록 해야 합니다. 이걸 위해 *state*를 사용할 겁니다.

state를 앱이 기억해야하는 최소한의 가변 데이터 모음이라고 생각하세요. 가장 중요한 원칙은 state 구조가 [DRY(Don’t Repeat Yourself)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)하게 유지돼야 한다는 것 입니다. 애플리케이션에서 필요로하는 state의 가장 작은 표현을 찾고, 다른 모든 것들은 온디맨드(on-demand)로 계산해야 합니다. 예를 들어, 쇼핑 리스트를 구현한다면, 아이템들을 state에 배열로 저장할 수 있습니다. 리스트의 아이템 갯수를 출력하고 싶다면, 아이템의 갯수를 별도의 값으로 state에 저장하지 마세요. 대신, 배열의 길이를 읽으세요.

이제 예제 애플리케이션의 모든 데이터를 생각해 봅니다.

- 원본 상품 리스트
- 유저가 입력한 검색문
- 체크 박스의 값
- 필터링 된 상품 리스트

어떤걸 state로 만들어야 할까요? 그렇지 않은 것을 알아봅시다.

- 시간이 지나도 변하지 않는 데이터인가요? 그럼 state가 아닙니다.
- 부모로부터 props로 전달받나요? 그럼 state가 아닙니다.
- 컴포넌트에 존재하는 state나 props를 기반으로 계산할 수 있는 건가요? 그렇다면 절대로 state가 아닙니다.

남은 것들은 아마 state일 겁니다.

하나씩 다시 살펴봅시다.

- 원본 상품 리스트는 **props로 전달받으므로, state가 아닙니다.**
- 검색문은 시간이 흐르면서 변할 수 있고 다른 것으로 부터 계산될 수 없기 떄문에 state로 보입니다.
- 체크 박스의 값은 시간이 흐르면서 변할 수 있고 다른 것으로 부터 계산될 수 없기 떄문에 state로 보입니다.
- 필터링된 리스트는 원본 상품 리스트를 가져와 검색문과 체크 박스 값을 고려하여 **계산할 수 있으므로 state가 아닙니다.**

즉, 검색문과 체크 박스 값만이 state입니다! 잘하셨습니다!

> ### Props vs State
>
> React에서 데이터의 모델은 props와 state 두 가지가 존재합니다. 두가지는 매우 다릅니다.
>
> - Props는 함수에 넘기는 인자와 같습니다. 부모 컴포넌트에서 자식 컴포넌트로 데이터를 전달하고 자식 컴포넌트의 외형을 조작할 수 있게 해줍니다. 예를 들어, `Form`은 `Button`으로 `color` prop을 넘길 수 있습니다.
> - State는 컴포넌트의 메모리와 같습니다. State를 사용하여 컴포넌트는 특정 정보를 관리하고 상화 작용을 통해 변경할 수 있습니다. 예를 들어, `Button`은 `isHovered` state를 관리할 수 있습니다.
>
> Prop과 State는 다르지만, 함께 동작합니다. 부모 컴포넌트는 정보를 state로 저장(때문에 변경될 수 있습니다)하고, 자식 컴포넌트의 props로 이를 내려주는 경우가 많습니다. 처음 읽을 떄 차이점이 잘 이해가지 않아도 괜찮습니다. 완벽하게 이해하기 위해서는 조금의 연습이 필요합니다!

## Step 4: state가 들어있을 위치 찾기

앱의 최소한의 state 데이터를 확인한 후에, 어떤 컴포넌트가 이 state에 대한 변경의 주체인지, state를 소유해야 하는지에 대해 알아볼 필요가 있습니다. 리엑트는 컴포넌트 계층 구조의 부모 컴포넌트에서 자식 컴포넌트로 데이터를 내려주는, 단방향 데이터 흐름(one-way data flow)을 사용해야 하는 것을 기억하세요. 어떤 컴포넌트가 어떤 state를 가져야 하는지는 당장 명확하지 않아도 됩니다. 이런 컨셉을 처음 경험해본다면 이것이 어려울 수 있지만, 다음 과저을 따라가면 알아낼 수 있습니다.

애플리케이션의 각 state에 대해

- 해당 state를 기반으로 무언가를 랜더하는 컴포넌트를 모두 찾습니다.
- 이 컴포넌트들의 가장 가까운 공통 부모 컴포넌트를 찾습니다 - 계층 구조에서 위에서 찾아낸 모든 컴포넌트들의 상위에 존재하는 컴포넌트 입니다.
- state가 존재할 장소를 결정합니다.
  - 일반적으로, 이들의 공통 부모에 state를 직접 넣어줄 수 있습니다.
  - 이들의 공통 부모의 상위에 있는 컴포넌트에 state를 위치시킬 수도 있습니다.
  - state를 소유할 알맞은 컴포넌트를 찾지 못했다면, 오로지 해당 state를 소유하기 위한 새로운 컴포넌트를 만들고 계층상에서 공통 부모 컴포넌트 상위에 추가할 수 있습니다.

이전 스탭에서, 이 애플리케이션의 state를 두개 찾았습니다. 검색문꽈 체크 박스 값입니다. 이 예제에서, 이 둘은 항상 함께 출력되므로, 이들을 단일 state로 생각하기 쉽습니다.

이제 이 state를 위한 전략을 살펴봅니다.

1. **state를 사용하는 컴포넌트를 찾습니다**
   - `ProductTable`는 state(검색문과 체크 박스 값)를 기반으로 상품 리스트를 필터해야 합니다.
   - `SearchBar`는 state(검색문과 체크 박스 값)를 출력해야 합니다.
2. **이들의 공통 부모를 찾습니다**: 두 컴포넌트가 공유하는 첫 번째 부모 컴포넌트는 `FilterableProductTable`입니다.
3. **state의 위치를 결정합니다**: 우리는 검색문과 체크 박스 값 state를 `FilterableProductTable`에 위치시킬 겁니다.

그러므로 state 값들은 `FilterableProductTable` 내부에 위치할 겁니다.

[`useState` 훅](https://beta.reactjs.org/reference/react/useState)을 사용해서 컴포넌트에 state를 추가합니다. 훅을 사용하면 컴포넌트의 [랜더 사이클](https://beta.reactjs.org/learn/render-and-commit)을 관리(hook into)할 수 있습니다. `FilterableProductTable` 상단에 두개의 state 변수를 추가하고, 애플리케이션의 초기 state 지정합니다. 그리고 `filterText`와 `inStockOnly`를 `ProductTable`와 `SearchBar`의 props로 넘깁니다.

```tsx
function FilterableProductTable({ products }: { products: Array<Product> }) {
  const [filterText, setFilterText] = useState("");
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar filterText={filterText} inStockOnly={inStockOnly} />
      <ProductTable products={products} filterText={filterText} inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductTable({
  products,
  filterText,
  inStockOnly,
}: {
  products: Array<Product>;
  filterText: string;
  inStockOnly: string;
}) {
  const rows: Array<ReactNode> = [];
  let lastCategory: string | null = null;

  products.forEach((product) => {
    if (product.name.toLowerCase().indexOf(filterText.toLowerCase()) === -1) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(<ProductCategoryRow category={product.category} key={product.category} />);
    }
    rows.push(<ProductRow product={product} key={product.name} />);
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly }: { filterText: string; inStockOnly: boolean }) {
  return (
    <form>
      <input type="text" value={filterText} placeholder="Search..." />
      <label>
        <input type="checkbox" checked={inStockOnly} /> Only show products in stock
      </label>
    </form>
  );
}
```

애플리케이션이 어떻게 동작하는지 확인할 수 있습니다. `useState('')`를 `useState('fruit')`로 수정하여 `filterText` 초기 값을 수정해 봅니다.

```tsx
function FilterableProductTable({ products }: { products: Array<Product> }) {
  const [filterText, setFilterText] = useState("fruit");
  const [inStockOnly, setInStockOnly] = useState(false);

  // ...
}
```

아직 form 수정은 동작하지 않습니다. 다음은 이유를 설명해주는 콘솔 에러입니다.

```
You provided a `value` prop to a form field without an `onChange` handler. This will render a read-only field.
```

`ProductTable`와 `SearchBar`는 테이블, 체크박스 그리고 인풋을 렌더라기 위해서 `filterText`와 `inStockOnly` props를 읽습니다. 예를 들어, 다음은 `SearchBar`가 입력 값을 채우는 방법입니다.

```tsx
function SearchBar({ filterText, inStockOnly }: { filterText: string; inStockOnly: boolean }) {
  return (
    <form>
      <input type="text" value={filterText} placeholder="Search..." />
      {/* ... */}
    </form>
  );
}
```

하지만, 아직 타이핑과 같은 유저 액션에 응답하는 어떤 코드도 추가하지 않았습니다. 이것이 마지막 스탭이 될겁니다.

## Step 5: 역 데이터 흐름 추가하기

현재 앱은 계층의 아래로 흐르는 props와 state를 정확하게 렌더합니다. 그러나 유저의 입력에 따라 state를 변경하기 위해서, 다른 방식의 데이터 흐름을 지원해야 합니다. 계층 안의 form 컴포넌트 깊숙한 곳에서 `FilterableProductTable`의 state를 변경할 필요가 있습니다.

React는 이 데이터 흐름을 명시적으로 만들지만, 양방향 데이터 바인딩(two-way data binding)보다 좀더 많은 타이필이 필요합니다. 위의 예시에서 타이핑을 치거나 체크 박스를 체크하면, React가 입력을 무시하는 것을 볼 수 있습니다. 이건 계회된 것 입니다. `<input value={filterText} />`에 입력을 하더라도, `input`의 `value` prop에 항상 `FilterableProductTable`로부터 전달되는 동일한 `filterText` state가 할당됩니다. `filterText`가 바뀌지 않는 한, input 또한 바뀌지 않습니다.

유저가 form의 input을 변경할 때 마다, 변경 사항을 반영하여 state가 업데이트 되게 만들고 싶습니다. `FilterableProductTable`가 소유한 state는 `setFilterText`와 `setInStockOnly`로만 변경할 수 있습니다. `SearchBar`가 `FilterableProductTable`의 state를 변경하기 위해서는, 이 함수들을 `SearchBar`로 내려줘야 합니다.

```tsx
function FilterableProductTable({ products }: { products: Array<Product> }) {
  const [filterText, setFilterText] = useState("");
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly}
      />
      <ProductTable products={products} filterText={filterText} inStockOnly={inStockOnly} />
    </div>
  );
}
```

`SearchBar` 내부에서는 `onChange`이벤트 핸들러를 추가할 수 있으며, 이를 통해 부모의 state를 변경할 수 있습니다.

```tsx
function SearchBar({ filterText, inStockOnly, onFilterTextChange, onInStockOnlyChange }) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Search..."
        onChange={(e) => onFilterTextChange(e.target.value)}
      />
      <label>
        <input type="checkbox" checked={inStockOnly} onChange={(e) => onInStockOnlyChange(e.target.checked)} /> Only
        show products in stock
      </label>
    </form>
  );
}
```

이제 애플리케이션이 완벽하게 동작합니다!
