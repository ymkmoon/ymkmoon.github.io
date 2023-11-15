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

동일한 컴포넌트는 제외하고 작성을 하겠다.


```
src
 ┣ constants
 ┃ ┣ dnd.ts
 ┣ components
 ┃ ┣ DragAndResizeMultiple
 ┃   ┣ Boundary.tsx ## 동일
 ┃   ┣ BoundaryLayout.tsx
 ┃   ┣ DragLayout.tsx
 ┃   ┣ DragLayoutSecond.tsx
 ┃   ┣ DragMenu.tsx
 ┃   ┣ DragMenuSecond.tsx
 ┃   ┣ ResizeModal.tsx ## 동일
 ┃   ┣ ResizeModalSecond.tsx 
 ┣ utils
 ┃ ┣ Resize.ts
 ┃ ┣ registDragEvent.ts
```

<br/>


### 바운더리 레아이웃 수정

> BoundaryLayout.tsx

```ts
import { useEffect, useRef, useState } from 'react';
import { inrange } from 'utils/resize';
import Boundary from './Boundary';
import registDragEvent from 'utils/registDragEvent';
import { BOUNDARY_MARGIN, DEFAULT_W, DEFAULT_H } from 'constants/dnd';
import DragMenu from './DragMenu';
import DragMenuSecond from './DragMenuSecond';

interface BoundaryLayoutProps {
    isShowFirstMenu: boolean;
    isShowSecondMenu: boolean;
}


const BoundaryLayout: React.FC<BoundaryLayoutProps> = ({
    isShowFirstMenu,
    isShowSecondMenu,
}: BoundaryLayoutProps) => {

    const boundaryRef = useRef<HTMLDivElement>(null);

    const [{ x, y, w, h }, setConfig] = useState({
        x: 0,
        y: 0,
        w: 0,
        h: 0,
    });

    useEffect(() => {
        const boundary = boundaryRef.current?.getBoundingClientRect();

        if (boundary) {
            setConfig({
                x: Math.floor(boundary.width / 2 - DEFAULT_W / 2),
                y: Math.floor(boundary.height / 2 - DEFAULT_H / 2),
                w: DEFAULT_W,
                h: DEFAULT_H,
            });

            setFirstMenuConfig({
                firstMenuX: Math.floor(boundary.width / 2 - DEFAULT_W / 2),
                firstMenuY: Math.floor(boundary.height / 2 - DEFAULT_H / 2),
                firstMenuW: DEFAULT_W,
                firstMenuH: DEFAULT_H,
            });

            setSecondMenuConfig({
                secondMenuX: Math.floor(boundary.width / 2 - DEFAULT_W / 2),
                secondMenuY: Math.floor(boundary.height / 2 - DEFAULT_H / 2),
                secondMenuW: DEFAULT_W,
                secondMenuH: DEFAULT_H,
            });
        }
    }, []);

    const [{ firstMenuX, firstMenuY, firstMenuW, firstMenuH }, setFirstMenuConfig] = useState({
        firstMenuX: 0,
        firstMenuY: 0,
        firstMenuW: 0,
        firstMenuH: 0,
    });
    const [firstMenuShow, setFirstMenuShow] = useState(true);
    const [isFirstMenuOpen, setFirstMenuIsOpen] = useState<boolean>(false);

    const [{ secondMenuX, secondMenuY, secondMenuW, secondMenuH }, setSecondMenuConfig] = useState({
        secondMenuX: 0,
        secondMenuY: 0,
        secondMenuW: 0,
        secondMenuH: 0,
    });
    const [secondMenuShow, setSecondMenuShow] = useState(true);
    const [isSecondMenuOpen, setSecondMenuIsOpen] = useState<boolean>(false);

    return (
        <>
            <div className="p-4">
                <Boundary ref={boundaryRef}>
                    <div
                        style={{ width: firstMenuW, height: firstMenuH, left: firstMenuX, top: firstMenuY }}
                        className="absolute"
                        {...registDragEvent((deltaX: number, deltaY: number) => {
                            if (!boundaryRef.current) return;

                            const boundary = boundaryRef.current.getBoundingClientRect();
                            setFirstMenuConfig({
                                firstMenuX: inrange(firstMenuX + deltaX, BOUNDARY_MARGIN, boundary.width - firstMenuW - BOUNDARY_MARGIN),
                                firstMenuY: inrange(firstMenuY + deltaY, BOUNDARY_MARGIN, boundary.height - firstMenuH - BOUNDARY_MARGIN),
                                firstMenuW,
                                firstMenuH,
                            });
                            console.log('first ' + firstMenuX + " / " + firstMenuY + " / " + firstMenuW + " / " + firstMenuH)
                        })}
                    >
                        {isShowFirstMenu && <DragMenu
                            isOpen={isFirstMenuOpen}
                            setConfig={setFirstMenuConfig}
                            x={firstMenuX}
                            y={firstMenuY}
                            w={firstMenuW}
                            h={firstMenuH}
                            boundaryRef={boundaryRef}
                            show={firstMenuShow}
                        />}
                    </div>

                    <div
                        style={{ width: secondMenuW, height: secondMenuH, left: secondMenuX, top: secondMenuY }}
                        className="absolute"
                        {...registDragEvent((deltaX: number, deltaY: number) => {
                            if (!boundaryRef.current) return;

                            const boundary = boundaryRef.current.getBoundingClientRect();
                            setSecondMenuConfig({
                                secondMenuX: inrange(secondMenuX + deltaX, BOUNDARY_MARGIN, boundary.width - secondMenuW - BOUNDARY_MARGIN),
                                secondMenuY: inrange(secondMenuY + deltaY, BOUNDARY_MARGIN, boundary.height - secondMenuH - BOUNDARY_MARGIN),
                                secondMenuW,
                                secondMenuH,
                            });
                            console.log('second ' + secondMenuX + " / " + secondMenuY + " / " + secondMenuW + " / " + secondMenuH)
                        })}
                    >
                        {isShowSecondMenu && <DragMenuSecond
                            isOpen={isSecondMenuOpen}
                            setConfig={setSecondMenuConfig}
                            x={secondMenuX}
                            y={secondMenuY}
                            w={secondMenuW}
                            h={secondMenuH}
                            boundaryRef={boundaryRef}
                            show={secondMenuShow}
                        />}
                    </div>
                </Boundary>
            </div>
        </>
    );
}

export default BoundaryLayout;
```



### Drag Layout 추가 및 추가

<details>
<summary>DragLayout.tsx</summary>
<div markdown="1">

```ts
import { inrange } from 'utils/resize';
import registDragEvent from 'utils/registDragEvent';
import { BOUNDARY_MARGIN, MIN_W, MIN_H } from 'constants/dnd';


interface DragLayoutProps {
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
    boundaryRef: React.RefObject<HTMLDivElement>;
    show: boolean;
}

const DragLayout: React.FC<DragLayoutProps> = ({
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
                        firstMenuX: inrange(x + deltaX, BOUNDARY_MARGIN, x + w - MIN_W),
                        firstMenuY: inrange(y + deltaY, BOUNDARY_MARGIN, y + h - MIN_H),
                        firstMenuW: inrange(w - deltaX, MIN_W, x + w - BOUNDARY_MARGIN),
                        firstMenuH: inrange(h - deltaY, MIN_H, y + h - BOUNDARY_MARGIN),
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
                        firstMenuX: x,
                        firstMenuY: inrange(y + deltaY, BOUNDARY_MARGIN, y + h - MIN_H),
                        firstMenuW: inrange(w + deltaX, MIN_W, boundary.width - x - BOUNDARY_MARGIN),
                        firstMenuH: inrange(h - deltaY, MIN_H, y + h - BOUNDARY_MARGIN),
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
                        firstMenuX: inrange(x + deltaX, BOUNDARY_MARGIN, x + w - MIN_W),
                        firstMenuY: y,
                        firstMenuW: inrange(w - deltaX, MIN_W, x + w - BOUNDARY_MARGIN),
                        firstMenuH: inrange(h + deltaY, MIN_H, boundary.height - y - BOUNDARY_MARGIN),
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
                        firstMenuX: x,
                        firstMenuY: y,
                        firstMenuW: inrange(w + deltaX, MIN_W, boundary.width - x - BOUNDARY_MARGIN),
                        firstMenuH: inrange(h + deltaY, MIN_H, boundary.height - y - BOUNDARY_MARGIN),
                    });
                }, true)}
            />
            {/* 상단 */}
            <div
                className="absolute -top-0.5 left-3 right-3 h-2 cursor-n-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((_, deltaY) => {
                    setConfig({
                        firstMenuX: x,
                        firstMenuY: inrange(y + deltaY, BOUNDARY_MARGIN, y + h - MIN_H),
                        firstMenuW: w,
                        firstMenuH: inrange(h - deltaY, MIN_H, y + h - BOUNDARY_MARGIN),
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
                        firstMenuX: x,
                        firstMenuY: y,
                        firstMenuW: w,
                        firstMenuH: inrange(h + deltaY, MIN_H, boundary.height - y - BOUNDARY_MARGIN),
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
                        firstMenuX: x,
                        firstMenuY: y,
                        firstMenuW: inrange(w + deltaX, MIN_W, boundary.width - x - BOUNDARY_MARGIN),
                        firstMenuH: h,
                    });
                }, true)}
            />
            {/* 좌측 */}
            <div
                className="absolute bottom-3 top-3 -left-0.5 w-2 cursor-w-resize"
                style={{ backgroundColor: show ? '#12121250' : 'transparent' }}
                {...registDragEvent((deltaX) => {
                    setConfig({
                        firstMenuX: inrange(x + deltaX, BOUNDARY_MARGIN, x + w - MIN_W),
                        firstMenuY: y,
                        firstMenuW: inrange(w - deltaX, MIN_W, x + w - BOUNDARY_MARGIN),
                        firstMenuH: h,
                    });
                }, true)}
            />
        </>

    );
}

export default DragLayout;
```

</div>
</details>

<details>
<summary>DragLayoutSecond.tsx</summary>
<div markdown="1">

```ts
import { inrange } from 'utils/resize';
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

</div>
</details>


<br/>


### 모달 추가


<details>
<summary>ResizeModal.tsx</summary>
<div markdown="1">

```ts
const ResizeModal = () => {
    return (<>
        <>
            <div className="absolute h-full w-full cursor-move rounded-xl bg-white shadow-xl ring-1 ring-gray-100 transition-[shadow,transform] active:scale-[0.97] active:shadow-lg" >
                <div className="modal-header dark">
                    <h5 className="modal-title dark f14">테스트 모달 1</h5>
                </div>
            </div>
            <br />
        </>
    </>)
}

export default ResizeModal;
```

</div>
</details>


<details>
<summary>ResizeModalSecond.tsx</summary>
<div markdown="1">

```ts
const ResizeModalSecond = () => {

    return (<>
        <>
            <div className="absolute h-full w-full cursor-move rounded-xl bg-white shadow-xl ring-1 ring-gray-100 transition-[shadow,transform] active:scale-[0.97] active:shadow-lg" >
                <div className="modal-header dark">
                    <h5 className="modal-title dark f14">테스트 모달 2</h5>
                </div>
            </div>
            <br />
        </>
    </>)
}

export default ResizeModalSecond;
```

</div>
</details>

<br/>


### 모달 추가

<details>
<summary>DragMenu.tsx</summary>
<div markdown="1">

```ts
import ResizeModal from "components/DragAndResizeMultiple/ResizeModal";
import DragLayout from "components/DragAndResizeMultiple/DragLayout";

interface DragMenuProps {
    isOpen: boolean;
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

const DragMenu: React.FC<DragMenuProps> = ({
    isOpen,
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
                        <ResizeModal />
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

export default DragMenu;
```

</div>
</details>

<details>
<summary>DragMenuSecond.tsx</summary>
<div markdown="1">

```ts
import ResizeModalSecond from "components/DragAndResizeMultiple/ResizeModalSecond";
import DragLayoutSecond from "components/DragAndResizeMultiple/DragLayoutSecond";

interface DragMenuSecondProps {
    isOpen: boolean;
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
    boundaryRef: React.RefObject<HTMLDivElement>
    show: boolean;
}

const DragMenuSecond: React.FC<DragMenuSecondProps> = ({
    isOpen,
    setConfig,
    x, y, w, h,
    boundaryRef,
    show,
}: DragMenuSecondProps) => {

    return (
        <>
            {isOpen &&
                <>
                    <div className="absolute h-full w-full cursor-move rounded-xl bg-white shadow-xl ring-1 ring-gray-100 transition-[shadow,transform] active:scale-[0.97] active:shadow-lg">
                        <ResizeModalSecond />
                    </div>

                    <DragLayoutSecond
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

</div>
</details>




![React](/assets/image/react/React_DND_02.PNG)