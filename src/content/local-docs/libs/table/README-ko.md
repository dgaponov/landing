# @gravity-ui/table &middot; [![npm package](https://img.shields.io/npm/v/@gravity-ui/table)](https://www.npmjs.com/package/@gravity-ui/table) [![CI](https://img.shields.io/github/actions/workflow/status/gravity-ui/table/.github/workflows/ci.yml?label=CI&logo=github)](https://github.com/gravity-ui/table/actions/workflows/ci.yml?query=branch:main) [![storybook](https://img.shields.io/badge/Storybook-deployed-ff4685)](https://preview.gravity-ui.com/table/)

## 설치

```shell
npm install --save @gravity-ui/table
```

## 사용법

```tsx
import React from 'react';
import {Table, useTable} from '@gravity-ui/table';
import type {ColumnDef} from '@gravity-ui/table/tanstack';

interface Person {
  id: string;
  name: string;
  age: number;
}

const columns: ColumnDef<Person>[] = [
  {accessorKey: 'name', header: 'Name', size: 100},
  {accessorKey: 'age', header: 'Age', size: 100},
];

const data: Person[] = [
  {id: 'name', name: 'John', age: 23},
  {id: 'age', name: 'Michael', age: 27},
];

const BasicExample = () => {
  const table = useTable({
    columns,
    data,
  });

  return <Table table={table} />;
};
```

## 컴포넌트

사용할 수 있는 두 개의 Table 컴포넌트가 있습니다:

- `BaseTable` - 기본 스타일만 적용된 컴포넌트;
- `Table` - Gravity UI 기반 스타일이 적용된 컴포넌트.

### 행 선택

```tsx
import {selectionColumn} from '@gravity-ui/table';
import type {RowSelectionState} from '@gravity-ui/table/tanstack';

const columns: ColumnDef<Person>[] = [
  selectionColumn as ColumnDef<Person>,
  // ...other columns
];

const data: Person[] = [
  /* ... */
];

const RowSelectionExample = () => {
  const [rowSelection, setRowSelection] = React.useState<RowSelectionState>({});

  const table = useTable({
    columns,
    data,
    enableRowSelection: true,
    enableMultiRowSelection: true,
    onRowSelectionChange: setRowSelection,
    state: {
      rowSelection,
    },
  });

  return <Table table={table} />;
};
```

### 정렬

react-table [문서](https://tanstack.com/table/v8/docs/guide/sorting)에서 열 속성에 대해 알아보세요.

```tsx
import type {SortingState} from '@gravity-ui/table/tanstack';

const columns: ColumnDef<Person>[] = [
  /* ... */
];

const data: Person[] = [
  /* ... */
];

const SortingExample = () => {
  const [sorting, setSorting] = React.useState<SortingState>([]);

  // 정렬을 활성화하려면 열에 반드시 accessorFn이 있어야 합니다.

  const table = useTable({
    columns,
    data,
    enableSorting: true,
    getRowId: (item) => item.id,
    onSortingChange: setSorting,
    state: {
      sorting,
    },
  });

  return <Table table={table} />;
};
```

요소를 수동으로 정렬하려면 `manualSorting` 속성을 전달하세요:

```tsx
const table = useTable({
  // ...
  manualSorting: true,
});
```

### 그룹화

```tsx
import type {ExpandedState, Row} from '@gravity-ui/table/tanstack';

interface Person {
  id: string;
  name: string;
  age: number;
}

interface PersonGroup {
  id: string;
  name: string;
  items: Person[];
}

type Item = PersonGroup | Person;

const columns: ColumnDef<Item>[] = [
  {accessorKey: 'name', header: 'Name', size: 200},
  {accessorKey: 'age', header: 'Age', size: 100},
];

const data: Item[] = [
  {
    id: 'friends',
    name: 'Friends',
    items: [
      {id: 'nick', name: 'Nick', age: 25},
      {id: 'tom', name: 'Tom', age: 21},
    ],
  },
  {
    id: 'relatives',
    name: 'Relatives',
    items: [
      {id: 'john', name: 'John', age: 23},
      {id: 'michael', name: 'Michael', age: 27},
    ],
  },
];

const getGroupTitle = (row: Row<Item>) => row.getValue<string>('name');

const GroupingExample = () => {
  const [expanded, setExpanded] = React.useState<ExpandedState>({});

  const table = useTable({
    columns,
    data,
    enableExpanding: true,
    getSubRows: (item) => ('items' in item ? item.items : undefined),
    onExpandedChange: setExpanded,
    state: {
      expanded,
    },
  });

  return <Table table={table} getGroupTitle={getGroupTitle} />;
};
```

중첩 스타일을 활성화하려면 열 구성에서 `withNestingStyles = true`를 전달하세요.

중첩 표시기를 비활성화하려면 `showTreeDepthIndicators = false`를 전달하세요.

행을 확장/축소하는 컨트롤을 추가하려면, 셀 내용을 `TreeExpandableCell` 컴포넌트나 유사한 커스텀 컴포넌트로 감싸세요:

```tsx
import {TreeExpandableCell} from '@gravity-ui/table';

const columns: ColumnDef<Item>[] = [
  {
    accessorKey: 'name',
    header: 'Name',
    size: 200,
    showTreeDepthIndicators: false,
    withNestingStyles: true,
    cell: ({row, info}) => (
      <TreeExpandableCell row={row}>{info.getValue<string>()}</TreeExpandableCell>
    ),
  },
  // ...other columns
];
```

### 재정렬

```tsx
import type {ReorderingProviderProps} from '@gravity-ui/table';
import {dragHandleColumn, ReorderingProvider} from '@gravity-ui/table';

const columns: ColumnDef<Person>[] = [
  dragHandleColumn,
  // ...other columns
];

const data: Person[] = [
  /* ... */
];

const ReorderingExample = () => {
  const table = useTable({
    columns,
    data,
    getRowId: (item) => item.id,
  });

  const handleReorder = React.useCallback<
    NonNullable<ReorderingProviderProps<Person>['onReorder']>
  >(
    ({
      draggedItemKey,
      targetItemKey,
      baseItemKey,
      baseNextItemKey,
      enableNesting,
      nextChild,
      pullFromParent,
    }) => {
      // ...
    },
    [],
  );
```

### 가상화(Virtualization)

그리드 컨테이너를 스크롤 요소로 사용하려는 경우에 사용하세요(윈도우를 사용하려면 윈도우 가상화 섹션을 참조하세요). 컨테이너에 고정 높이를 설정해야 합니다. 그렇지 않으면 가상화가 작동하지 않습니다.

```tsx
import {useRowVirtualizer} from '@gravity-ui/table';

const columns: ColumnDef<Person>[] = [
  /* ... */
];

const data: Person[] = [
  /* ... */
];

const VirtualizationExample = () => {
  const table = useTable({
    columns,
    data,
    getRowId: (item) => item.id,
  });

  const containerRef = React.useRef<HTMLDivElement>(null);

  const rowVirtualizer = useRowVirtualizer({
    count: table.getRowModel().rows.length,
    estimateSize: () => 20,
    overscan: 5,
    getScrollElement: () => containerRef.current,
  });

  return (
    <div ref={containerRef} style={{height: '500px', overflow: 'auto'}}>
      <Table table={table} rowVirtualizer={rowVirtualizer} />
    </div>
  );
};
```

재정렬(reordering) 기능을 가상화와 함께 사용하는 경우 `rangeExtractor` 옵션을 추가로 전달해야 합니다:

```tsx
import {getVirtualRowRangeExtractor} from '@gravity-ui/table';

// ...

const tableRef = React.useRef<HTMLTableElement>(null);

const rowVirtualizer = useRowVirtualizer({
  // ...
  rangeExtractor: getVirtualRowRangeExtractor(tableRef.current),
});

return (
  <TableWithReordering
    ref={tableRef}
    table={table}
    rowVirtualizer={rowVirtualizer}
    onReorder={handleReorder}
  />
);
```

### 윈도우 가상화(Window virtualization)

윈도우를 스크롤 요소로 사용하려는 경우에 사용하세요.

```tsx
import {useWindowRowVirtualizer} from '@gravity-ui/table';

const columns: ColumnDef<Person>[] = [
  /* ... */
];

const data: Person[] = [
  /* ... */
];

const WindowVirtualizationExample = () => {
  const table = useTable({
    columns,
    data,
    getRowId: (item) => item.id,
  });

  const bodyRef = React.useRef<HTMLTableSectionElement>(null);

  const rowVirtualizer = useWindowRowVirtualizer({
    count: table.getRowModel().rows.length,
    estimateSize: () => 20,
    overscan: 5,
    scrollMargin: bodyRef.current?.offsetTop ?? 0,
  });

  return <Table table={table} rowVirtualizer={rowVirtualizer} bodyRef={bodyRef} />;
};
```

### 크기 조정(Resizing)

```tsx
const columns: ColumnDef<Person>[] = [
  /* ... */
];

const data: Person[] = [
  /* ... */
];

const ResizingDemo = () => {
  const table = useTable({
    columns,
    data,
    enableColumnResizing: true,
    columnResizeMode: 'onChange',
  });

  return <Table table={table} />;
};
```

### 열 설정(Column settings)

```tsx
const columns: ColumnDef<Person>[] = [
  // ...other columns
  {
    id: 'settings_column_id',
    header: ({table}) => <TableSettings table={table} />,
    meta: {
      hideInSettings: false, // Optional. Allows to hide this column from settings popover
      titleInSettings: 'ReactNode', // Optional. Overrides header field for settings popover (if you need different content for header and settings popover)
    },
  }, // or you can use function getSettingsColumn
];

const data: Person[] = [
  /* ... */
];

const TableSettingsDemo = () => {
  const [columnVisibility, onColumnVisibilityChange] = React.useState<VisibilityState>({
    // for outside control and initial state
    column_id: false, // for hidding by default
  });
  const [columnOrder, onColumnOrderChange] = React.useState<string[]>([
    /* leaf columns ids */
  ]); // for outside control and initial state

  // Alternative variant to get state, callbacks, and set on setting apply callbacks - using useTableSettings hook:
  // const {state, callbacks} = useTableSettings({initialVisibility: {}, initialOrder: []})

  const table = useTable({
    columns,
    data,
    state: {
      columnVisibility,
      columnOrder,
    },
    onColumnVisibilityChange,
    onColumnOrderChange,
  });

  return <Table table={table} />;
};
```

테이블과 열 크기 조정 속성에 대해 자세히 알아보려면 react-table [문서](https://tanstack.com/table/v8/docs/api/features/column-sizing)를 참조하세요.