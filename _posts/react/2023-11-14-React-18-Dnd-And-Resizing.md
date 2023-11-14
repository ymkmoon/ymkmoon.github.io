---
title:  "[React] DragAndDrop(DND) 와 Resizing 사용하기 (Typescript)"
excerpt: "Fragment 의 사용이유와 사용법"

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

[이전 챕터](https://ymkmoon.github.io/React-17-Drag-And-Drop/)에서는 DND 를 이용해 모달을 이동시키는 방법을 알어봤으며, <br>
이번에는 DND 와 함께 모달의 크기도 자유자재로 변경하는 방법을 알아보자.


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
 ┃   ┣ DragMenu.tsx
 ┃   ┣ ResizeModal.tsx
 ┣ utils
 ┃ ┣ Resize.ts
 ┃ ┣ registDragEvent.ts
```

<br/>


### 드래그 이벤트 등록

> registDragEvent.ts

```ts
const isTouchScreen =
    typeof window !== 'undefined' && window.matchMedia('(hover: none) and (pointer: coarse)').matches;

export default function registDragEvent(
    onDragChange: (deltaX: number, deltaY: number) => void,
    stopPropagation?: boolean,
) {
    if (isTouchScreen) {
        return {
            onTouchStart: (touchEvent: React.TouchEvent<HTMLDivElement>) => {

                // element의 클릭 이벤트가 부모로 전파되지 않길 원할 경우 stopPropagation을 호출
                if (stopPropagation) touchEvent.stopPropagation();

                // 클릭된 상태에서 마우스를 움직일시 mousemove에 등록 된 함수가 계속 호출
                const touchMoveHandler = (moveEvent: TouchEvent) => {
                    if (moveEvent.cancelable) moveEvent.preventDefault();
                    // 클릭(mousedown) 이벤트 발생시의 마우스 위치를 기준으로,
                    // 이동(mousemove) 이벤트에서 상대적으로 이동한 거리(deltaX, deltaY)를 계산하고
                    // 콜벡으로 받은 onDragChange에게 전달
                    const deltaX = moveEvent.touches[0].screenX - touchEvent.touches[0].screenX;
                    const deltaY = moveEvent.touches[0].screenY - touchEvent.touches[0].screenY;
                    onDragChange(deltaX, deltaY);
                };

                // mouseup 이벤트에서 mousemove 이벤트를 제거
                const touchEndHandler = () => {
                    document.removeEventListener('touchmove', touchMoveHandler);
                };

                // 클릭시(onMouseDown) document에 mousemove mouseup 이벤트를 등록
                document.addEventListener('touchmove', touchMoveHandler, { passive: false });
                document.addEventListener('touchend', touchEndHandler, { once: true });
            },
        };
    }

    return {
        onMouseDown: (clickEvent: React.MouseEvent<Element, MouseEvent>) => {
            if (stopPropagation) clickEvent.stopPropagation();

            const mouseMoveHandler = (moveEvent: MouseEvent) => {
                const deltaX = moveEvent.screenX - clickEvent.screenX;
                const deltaY = moveEvent.screenY - clickEvent.screenY;
                onDragChange(deltaX, deltaY);
            };

            const mouseUpHandler = () => {
                document.removeEventListener('mousemove', mouseMoveHandler);
            };

            document.addEventListener('mousemove', mouseMoveHandler);
            document.addEventListener('mouseup', mouseUpHandler, { once: true });
        },
    };
}

```

<br/>


### 거리계산 함수 추가

> Resize.ts

```ts
export const $ = (...classnames: any[]) => {
    return classnames.filter((v) => !!v).join(' ');
};

export const inrange = (v: number, min: number, max: number) => {
    if (v < min) return min;
    if (v > max) return max;
    return v;
};

```


### 기본 넓이&높이, 최소 넓이&높이 설정

> dnd.ts

```ts
export const BOUNDARY_MARGIN = 12;
export const MIN_W = 80;
export const MIN_H = 80;
export const DEFAULT_W = 240;
export const DEFAULT_H = 120;
```


<br/>


### 바운더리 설정

> Boundary.ysx

```ts
import React from 'react';

export default React.forwardRef<HTMLDivElement, React.ComponentProps<'div'>>(function Boundary(
  props,
  ref,
) {
  return (
    <div
      {...props}
      ref={ref}
      className={
        'relative h-64 overflow-hidden rounded-xl bg-gray-200 dark:bg-[#121212] ' + props.className
      }
    />
  );
});

```

<br/>


### 바운더리의 레이아웃 설정

> BoundaryLayout.tsx

```ts
import { useEffect, useRef, useState } from 'react';
import { inrange } from 'utils/Resize';
import Boundary from './Boundary';
import registDragEvent from 'utils/registDragEvent';
import { BOUNDARY_MARGIN, DEFAULT_W, DEFAULT_H } from 'constants/dnd';
import DragMenu from './DragMenu';

interface BoundaryLayoutProps {
    isShowFirstMenu: boolean;
}


const BoundaryLayout: React.FC<BoundaryLayoutProps> = ({
    isShowFirstMenu,
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
    const openFirstModal = () => {
        setFirstMenuIsOpen((prevState) => {
            if (!prevState) return true;
            return prevState;
        });
    };
    const closeFirstModal = () => {
        setFirstMenuIsOpen((prevState) => {
            if (prevState) return false;
            return prevState;
        });
    };


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
                        })}
                    >
                        {isShowFirstMenu && <DragMenu
                            isOpen={isFirstMenuOpen} openModal={openFirstModal} closeModal={closeFirstModal}
                            setConfig={setFirstMenuConfig}
                            x={firstMenuX}
                            y={firstMenuY}
                            w={firstMenuW}
                            h={firstMenuH}
                            boundaryRef={boundaryRef}
                            show={firstMenuShow}
                        />}
                    </div>
                </Boundary>
            </div>
        </>
    );
}

export default BoundaryLayout;
```

<br/>


### Drag Layout 설정

> DragLayout.tsx

```ts
import { inrange } from 'utils/Resize';
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


<br/>


### 모달 생성

> ResizeModal.tsx

```ts
const ResizeModal: React.FC = () => {
    return (<>
        <div className="absolute h-full w-full cursor-move rounded-xl bg-white shadow-xl ring-1 ring-gray-100 transition-[shadow,transform] active:scale-[0.97] active:shadow-lg" >
            <div className="modal-header dark">
                <h5 className="modal-title dark f14">테스트 모달 1</h5>
            </div>
        </div>
        <br />
    </>)
}

export default ResizeModal;
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

const DragMenu: React.FC<DragMenuProps> = ({
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

export default DragMenu;
```

<br/>

