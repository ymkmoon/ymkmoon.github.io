---
title:  "[React] DragAndDrop(DND) 사용하기 (Typescript)"
excerpt: "DND 사용해보기"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React DND]"
 
date: 2023-11-14
last_modified_at: 2023-11-14
---

### ``React DND``

- 참고사이트
    - [Medium.com](https://medium.com/litslink/react-dnd-in-examples-ce509b25839d){:target="_blank"}
    - [블로그](https://dsuumb.tistory.com/12){:target="_blank"}

<br>

### `DND 란?`

간단하다. 드래그와 드랍을 통해 특정 창의 위치를 변경한다는 의미이다. <br>

<br>

React 환경에서 DND 기능을 제공해주는 모듈은 대표적으로 3개가 존재한다.

<br>

- react-draggable
> 드래그로 아이템 간의 순서를 변경하는 기능보다, 아이템의 위치를 옮기능 기능에 강점이 있다.


- react-dnd
> DND 기능을 사용 할 수 있게 해주는 hook 을 제공해주는 모듈 이다. 


- react-beautiful-dnd

> react-dnd 에서 좀 더 확장된 느낌으로 UI/UX 퍼포먼스가 좋은 동작이 미리 정의되어 있다. 이처럼 편리한 기능을 제공하기 때문에 `react-dnd 보다 용량이 약 2배 많다.`

<br>

학습 후 우리가 얻게 될 결과물은 아래와 같다.

![React](/assets/image/react/React_DND_01.gif)

<br/>


### 라이브러리 다운로드

```
npm i react-dnd react-dnd-html5-backend
```

<br/>


### 프로젝트 구조

```
src
 ┣ constants
 ┃ ┣ dnd.ts
 ┣ components
 ┃ ┣ draganddrop
 ┃   ┣ Column.tsx
 ┃   ┣ DraggableComponent.tsx
 ┃   ┣ MovableItem.tsx
 ┃   ┣ tasks.ts
 ┣ types
 ┃ ┣ DragAndDrop.ts
```


<br/>


### CSS 추가

일단 각 컴포넌트에서 사용 될 css 를 추가하자.

```css
.dnd-container {
    display: flex;
    flex-direction: row;
    justify-content: space-around;
}

.dnd-column {
    height: 600px;
    width: 200px;
    margin: 20px;
    border-radius: 10px;
    box-shadow: 1px 1px 6px rgba(0, 0, 0, 0.5);
    justify-content: center;
    flex-wrap: wrap;
}

.dnd-first-column {
    background-color: #f5ffea;
}

.dnd-second-column {
    background-color: #fffbf5;
}

.dnd-movable-item {
    border-radius: 10px;
    background-color: #fff3f3;
    height: 100px;
    width: 170px;
    margin: 10px auto;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 0px 0px 3px rgba(0, 0, 0, 0.5);
}
```


<br/>


### DraggableComponent.tsx 

```ts
import { useState } from 'react';
import { DndProvider } from "react-dnd";
import { HTML5Backend } from "react-dnd-html5-backend";

import { tasks } from "components/draganddrop/tasks";
import { COLUMN_NAMES, ITEM_TYPE } from "constants/dnd";
import MovableItem from "components/draganddrop/MovableItem";
import Column from "components/draganddrop/Column";

const DraggableComponent: React.FC = () => {
    const [items, setItems] = useState(tasks);

    const moveCardHandler = (dragIndex: number, hoverIndex: number) => {
        const dragItem = items[dragIndex];

        // 1. 사용자가 item을 드래그하고 있다면
        if (dragItem) {
            setItems((prevState: any) => {
                // 2. 기존의 데이터(prevState)를 새로운 변수에 복사한다.
                const coppiedStateArray = [...prevState];

                // 3. splice로 hoverIndex 위치부터 1개의 데이터를 제거한 후,
                // 삭제한 index 위치에 현재 드래그하고 있는 item 데이터를 넣는다.
                // -> 삭제된 요소들의 배열은 prevItem 변수에 저장된다.
                const prevItem = coppiedStateArray.splice(hoverIndex, 1, dragItem);

                // 4. 3번과 마찬가지의 과정을 거친 후
                coppiedStateArray.splice(dragIndex, 1, prevItem[0]);

                // 5. coppiedStateArray 배열을 return
                return coppiedStateArray;
            });
        }
    };

    const returnItemsForColumn = (columnName: string) => {
        return items
            .filter((item: any) => item.column === columnName)
            .map((item, index) => (
                <MovableItem
                    key={item.id}
                    name={item.name}
                    setItems={setItems}
                    index={index}
                    moveCardHandler={moveCardHandler}
                />
            ));
    };

    const { DO_IT, IN_PROGRESS, AWAITING_REVIEW, DONE } = COLUMN_NAMES;

    return (
        <div className="dnd-container">
            <DndProvider backend={HTML5Backend}>
                <Column title={DO_IT} className="dnd-column dnd-first-column">
                    {returnItemsForColumn(DO_IT)}
                </Column>
                <Column title={IN_PROGRESS} className="dnd-column dnd-second-column">
                    {returnItemsForColumn(IN_PROGRESS)}
                </Column>
                <Column title={AWAITING_REVIEW} className="dnd-column dnd-first-column">
                    {returnItemsForColumn(AWAITING_REVIEW)}
                </Column>
                <Column title={DONE} className="dnd-column dnd-second-column">
                    {returnItemsForColumn(DONE)}
                </Column>
            </DndProvider>
        </div>
    );
};


export default DraggableComponent;
```


<br/>

### Column.tsx

```ts
import { FC } from "react";
import { useDrop } from "react-dnd";

import { COLUMN_NAMES, ITEM_TYPE } from "constants/dnd";
import { ColumnProps } from "types/DragAndDrop";

const Column: FC<ColumnProps> = ({ children, className, title }: ColumnProps) => {

    const [, drop] = useDrop({
        accept: ITEM_TYPE,
        drop: () => ( { name: title } ),
    });

    return (
        <div ref={drop} className={className}>
            {title}
            {children}
        </div>
    );
};

export default Column;
```

<br/>


### MovableItem.tsx

```ts
import { useRef } from "react";
import { useDrag, useDrop } from "react-dnd";

import { ItemState } from "types/DragAndDrop";
import { COLUMN_NAMES, ITEM_TYPE } from "constants/dnd";

const MovableItem = ({ name, index, moveCardHandler, setItems }: any) => {
    const changeItemColumn = (currentItem: any, columnName: string) => {
        setItems((prevState: ItemState[]) =>
            prevState.map((e: ItemState) => {
                return {
                    ...e,
                    column: e.name === currentItem.name ? columnName : e.column,
                };
            })
        );
    };

    const ref = useRef<HTMLDivElement>(null);

    // 마우스가 hover item 높이의 절반을 넘을 경우에만 이동을 수행하도록 설계
    // - 아래로 드래그할 때 커서가 50% 이하일 때만 이동
    // - 위로 드래그할 때 커서가 50% 이상일 때만 이동
    const [, drop] = useDrop({
        accept: ITEM_TYPE,
        hover(item: { index: number; name: string }, monitor: any) {
            if (!ref.current) {
                return;
            }
            const dragIndex = item.index;
            const hoverIndex = index;

            if (dragIndex === hoverIndex) {
                return;
            }

            const hoverBoundingRect = ref.current?.getBoundingClientRect();
            const hoverMiddleY =
                (hoverBoundingRect.bottom - hoverBoundingRect.top) / 2;
            const clientOffset = monitor.getClientOffset();
            const hoverClientY = clientOffset.y - hoverBoundingRect.top;

            // 아래로 드래깅
            if (dragIndex < hoverIndex && hoverClientY < hoverMiddleY) {
                return;
            }

            // 위로 드래깅
            if (dragIndex > hoverIndex && hoverClientY > hoverMiddleY) {
                return;
            }

            moveCardHandler(dragIndex, hoverIndex);
            item.index = hoverIndex;
        },
    });

    const [{ isDragging }, drag] = useDrag({
        type: ITEM_TYPE,
        item: { index, name },
        end: (item, monitor) => {
            const dropResult: any = monitor.getDropResult();
            if (dropResult) {
                const { name } = dropResult;
                const { DO_IT, IN_PROGRESS, AWAITING_REVIEW, DONE } = COLUMN_NAMES;
                switch (name) {
                    case DO_IT:
                        changeItemColumn(item, DO_IT);
                        break;
                    case IN_PROGRESS:
                        changeItemColumn(item, IN_PROGRESS);
                        break;
                    case AWAITING_REVIEW:
                        changeItemColumn(item, AWAITING_REVIEW);
                        break;
                    case DONE:
                        changeItemColumn(item, DONE);
                        break;
                }
            }
        },
        collect: (monitor) => ({
            isDragging: monitor.isDragging(),
        }),
    });

    const opacity = isDragging ? 0.4 : 1;

    drag(drop(ref));

    return (
        <>
            <div ref={ref} className="dnd-movable-item" style={{ opacity }}>
                {name}
            </div>
        </>
    );
};

export default MovableItem;
```


<br/>


### tasks.ts

```ts
import { COLUMN_NAMES } from "constants/dnd";

const { DO_IT } = COLUMN_NAMES;

export const tasks = [
    { id: 1, name: "Item 1", column: DO_IT },
    { id: 2, name: "Item 2", column: DO_IT },
    { id: 3, name: "Item 3", column: DO_IT },
    { id: 4, name: "Item 4", column: DO_IT },
];
```


<br/>


### dnd.ts

```ts
export const COLUMN_NAMES = {
    DO_IT: "Do it",
    IN_PROGRESS: "In Progress",
    AWAITING_REVIEW: "Awaiting review",
    DONE: "Done",
};

export const ITEM_TYPE = "BOARD_VIEW";
```

<br/>


### DragAndDrop.ts

```ts
export interface ItemState {
    id: number;
    name: string;
    column: string;
}

export interface ColumnProps {
    children: React.ReactNode;
    className: string;
    title: string;
}
```