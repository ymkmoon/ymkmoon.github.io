---
title:  "[React] DragAndDrop(DND) 와 Resizing 사용하기 #2 (Typescript)"
excerpt: "DND 와 Resizing 사용하기"

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
    - [블로그](https://dnd-playground.vercel.app/resize){:target="_blank"}
    - [블로그](https://velog.io/@bepyan/Resize-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%BD%80%EA%B0%9C%EA%B8%B0){:target="_blank"}

<br>

### `시작하기 전에`

[이전 챕터](https://ymkmoon.github.io/React-18-Dnd-And-Resizing/)에서는 하나의 모달만을 사용했지만, 이번에는 복수개의 모달을 DND 와 Resizing 하는 방법을 알아보자.


### 프로젝트 구조

```
src
 ┣ constants
 ┃ ┣ dnd.ts
 ┣ components
 ┃ ┣ dragandresize
 ┃   ┣ Boundary.tsx
 ┃   ┣ BoundaryLayout.tsx
 ┃   ┣ DragLayout.tsx
 ┃   ┣ DragLayoutSecond.tsx ## 추가
 ┃   ┣ DragMenu.tsx
 ┃   ┣ DragMenuSecond.tsx ## 추가
 ┃   ┣ ResizeModal.tsx
 ┃   ┣ ResizeModalSecond.tsx ## 추가
 ┣ utils
 ┃ ┣ Resize.ts
 ┃ ┣ registDragEvent.ts
```

<br/>



### Drag Layout 추가

> DragLayoutSecond.tsx

```ts
import { inrange } from 'utils/Resize';
import registDragEvent from 'utils/registDragEvent';
import { BOUNDARY_MARGIN, MIN_W, MIN_H } from 'constants/dnd';


interface DragLayoutProps {
    setConfig: React.Dispatch<React.SetStateAction<{
        secondMenuX: number;
        secondMenuY: number;
        secondMenuW: number;
        secondMenuH: number;
    }>>;
    x: number;
    y: number;
    w: number;
    h: number;
    boundaryRef: React.RefObject<HTMLDivElement>;
    show: boolean;
}

const DragLayoutSecond: React.FC<DragLayoutProps> = ({
    setConfig,
    x, y, w, h,
    boundaryRef,
    show,
}: DragLayoutProps) => {

    return (
        <>

            {/* 좌상단  */}
            <div
                className="absolute -top-1 -left-1 h-4 w-4 cursor-nw-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((deltaX, deltaY) => {
                    setConfig({
                        secondMenuX: inrange(x + deltaX, BOUNDARY_MARGIN, x + w - MIN_W),
                        secondMenuY: inrange(y + deltaY, BOUNDARY_MARGIN, y + h - MIN_H),
                        secondMenuW: inrange(w - deltaX, MIN_W, x + w - BOUNDARY_MARGIN),
                        secondMenuH: inrange(h - deltaY, MIN_H, y + h - BOUNDARY_MARGIN),
                    });
                }, true)}
            />
            {/* 우상단 */}
            <div
                className="absolute -top-1 -right-1 h-4 w-4 cursor-ne-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((deltaX, deltaY) => {
                    if (!boundaryRef.current) return;

                    const boundary = boundaryRef.current.getBoundingClientRect();

                    setConfig({
                        secondMenuX: x,
                        secondMenuY: inrange(y + deltaY, BOUNDARY_MARGIN, y + h - MIN_H),
                        secondMenuW: inrange(w + deltaX, MIN_W, boundary.width - x - BOUNDARY_MARGIN),
                        secondMenuH: inrange(h - deltaY, MIN_H, y + h - BOUNDARY_MARGIN),
                    });
                }, true)}
            />
            {/* 좌하단 */}
            <div
                className="absolute -bottom-1 -left-1 h-4 w-4 cursor-ne-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((deltaX, deltaY) => {
                    if (!boundaryRef.current) return;

                    const boundary = boundaryRef.current.getBoundingClientRect();

                    setConfig({
                        secondMenuX: inrange(x + deltaX, BOUNDARY_MARGIN, x + w - MIN_W),
                        secondMenuY: y,
                        secondMenuW: inrange(w - deltaX, MIN_W, x + w - BOUNDARY_MARGIN),
                        secondMenuH: inrange(h + deltaY, MIN_H, boundary.height - y - BOUNDARY_MARGIN),
                    });
                }, true)}
            />
            {/* 우하단 */}
            <div
                className="absolute -bottom-1 -right-1 h-4 w-4 cursor-se-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((deltaX, deltaY) => {
                    if (!boundaryRef.current) return;

                    const boundary = boundaryRef.current.getBoundingClientRect();

                    setConfig({
                        secondMenuX: x,
                        secondMenuY: y,
                        secondMenuW: inrange(w + deltaX, MIN_W, boundary.width - x - BOUNDARY_MARGIN),
                        secondMenuH: inrange(h + deltaY, MIN_H, boundary.height - y - BOUNDARY_MARGIN),
                    });
                }, true)}
            />
            {/* 상단 */}
            <div
                className="absolute -top-0.5 left-3 right-3 h-2 cursor-n-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((_, deltaY) => {
                    setConfig({
                        secondMenuX: x,
                        secondMenuY: inrange(y + deltaY, BOUNDARY_MARGIN, y + h - MIN_H),
                        secondMenuW: w,
                        secondMenuH: inrange(h - deltaY, MIN_H, y + h - BOUNDARY_MARGIN),
                    });
                }, true)}
            />
            {/* 하단 */}
            <div
                className="absolute -bottom-0.5 left-3 right-3 h-2 cursor-s-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((_, deltaY) => {
                    if (!boundaryRef.current) return;

                    const boundary = boundaryRef.current.getBoundingClientRect();

                    setConfig({
                        secondMenuX: x,
                        secondMenuY: y,
                        secondMenuW: w,
                        secondMenuH: inrange(h + deltaY, MIN_H, boundary.height - y - BOUNDARY_MARGIN),
                    });
                }, true)}
            />
            {/* 우측 */}
            <div
                className="absolute bottom-3 top-3 -right-0.5 w-2 cursor-e-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((deltaX) => {
                    if (!boundaryRef.current) return;

                    const boundary = boundaryRef.current.getBoundingClientRect();

                    setConfig({
                        secondMenuX: x,
                        secondMenuY: y,
                        secondMenuW: inrange(w + deltaX, MIN_W, boundary.width - x - BOUNDARY_MARGIN),
                        secondMenuH: h,
                    });
                }, true)}
            />
            {/* 좌측 */}
            <div
                className="absolute bottom-3 top-3 -left-0.5 w-2 cursor-w-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((deltaX) => {
                    setConfig({
                        secondMenuX: inrange(x + deltaX, BOUNDARY_MARGIN, x + w - MIN_W),
                        secondMenuY: y,
                        secondMenuW: inrange(w - deltaX, MIN_W, x + w - BOUNDARY_MARGIN),
                        secondMenuH: h,
                    });
                }, true)}
            />
        </>

    );
}

export default DragLayoutSecond;
```


<br/>


### 모달 추가

> ResizeModalSecond.tsx

```ts
const ResizeModalSecond: React.FC = () => {
    return (<>
        <div className="absolute h-full w-full cursor-move rounded-xl bg-white shadow-xl ring-1 ring-gray-100 transition-[shadow,transform] active:scale-[0.97] active:shadow-lg" >
            <div className="modal-header dark">
                <h5 className="modal-title dark f14">테스트 모달 1</h5>
            </div>
        </div>
        <br />
    </>)
}

export default ResizeModalSecond;
```

<br/>


### 모달 등록

> DragMenu.tsx

```ts
import ResizeModal from "components/dragandresize/ResizeModal";
import DragLayout from "components/dragandresize/DragLayout";

interface DragMenuProps {
    isOpen: boolean;
    openModal: () => void;
    closeModal: () => void;
    setConfig: React.Dispatch<React.SetStateAction<{
        firstMenuX: number;
        firstMenuY: number;
        firstMenuW: number;
        firstMenuH: number;
    }>>;
    x: number;
    y: number;
    w: number;
    h: number;
    boundaryRef: React.RefObject<HTMLDivElement>
    show: boolean;
}

const DragMenuSecond: React.FC<DragMenuProps> = ({
    isOpen, openModal, closeModal,
    setConfig,
    x, y, w, h,
    boundaryRef,
    show,
}: DragMenuProps) => {

    return (
        <>
            {isOpen &&
                <>
                    <div className="absolute h-full w-full cursor-move rounded-xl bg-white shadow-xl ring-1 ring-gray-100 transition-[shadow,transform] active:scale-[0.97] active:shadow-lg">
                        <ResizeModal isOpen={isOpen} close={closeModal} />
                    </div>

                    <DragLayout
                        setConfig={setConfig}
                        x={x}
                        y={y}
                        w={w}
                        h={h}
                        boundaryRef={boundaryRef}
                        show={show}
                    />
                </>
            }

        </>

    );
}

export default DragMenuSecond;
```

<br/>

### 컴포넌트 호출


```ts
import { FC, useRef, useState, useMemo } from 'react';
import "./css/resize.css";
import BoundaryLayout from "components/dragandresize/BoundaryLayout";

const [isShowFirstMenu, setIsShowFirstMenu] = useState(false);
const [isShowSecondMenu, setIsShowSecondMenu] = useState(false);

const openFirstMenu = () => {
    setIsShowFirstMenu(!isShowFirstMenu);
}

const openSecondMenu = () => {
    setIsShowSecondMenu(!isShowSecondMenu);
}

const bundaryLayout = useMemo(
    () => {
        return (<BoundaryLayout
            isShowFirstMenu={isShowFirstMenu}
            isShowSecondMenu={isShowSecondMenu}
        />)
    },
    [isShowFirstMenu, isShowSecondMenu]
);

return (
    <>
        <div className="wrapper-home-custom">
            {bundaryLayout}


            <button
                type="button"
                className="btn btn-type-04 sm-btn"
                onClick={openFirstMenu}
            >
                first menu
            </button>

            <button
                type="button"
                className="btn btn-type-04 sm-btn"
                onClick={openSecondMenu}
            >
                second menu
            </button>
        </div>
    </>
);




![React](/assets/image/react/React_DND_02.PNG)