---
title:  "[React] DragAndDrop(DND) 와 Resizing 사용하기 #1 (Typescript)"
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

[이전 챕터](https://ymkmoon.github.io/React-17-Drag-And-Drop/)에서는 DND 를 이용해 모달을 이동시키는 방법을 알어봤으며, <br>
이번에는 DND 와 함께 모달의 크기도 자유자재로 변경하는 방법을 알아보자.


### 프로젝트 구조

```
src
 ┣ constants
 ┃ ┣ dnd.ts
 ┣ components
 ┃ ┣ DragAndResize
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

### css 추가



<details>
<summary>resize.css</summary>
<div markdown="1">

```css
/*
! tailwindcss v3.1.8 | MIT License | https://tailwindcss.com
*/
*,


:-moz-focusring {
  outline: auto
}

:-moz-ui-invalid {
  box-shadow: none
}

progress {
  vertical-align: baseline
}


.visible {
  visibility: visible
}

.fixed {
  position: fixed
}

.absolute {
  position: absolute
}

.relative {
  position: relative
}

.top-16 {
  top: 4rem
}

.-top-1 {
  top: -.25rem
}

.-left-1 {
  left: -.25rem
}

.-right-1 {
  right: -.25rem
}

.-bottom-1 {
  bottom: -.25rem
}

.-top-0\.5 {
  top: -.125rem
}

.left-3 {
  left: .75rem
}

.right-3 {
  right: .75rem
}

.-top-0 {
  top: 0
}

.-bottom-0\.5 {
  bottom: -.125rem
}

.-bottom-0 {
  bottom: 0
}

.bottom-3 {
  bottom: .75rem
}

.top-3 {
  top: .75rem
}

.-right-0\.5 {
  right: -.125rem
}

.-right-0 {
  right: 0
}

.-left-0\.5 {
  left: -.125rem
}

.-left-0 {
  left: 0
}

.mx-auto {
  margin-left: auto;
  margin-right: auto
}

.mb-2 {
  margin-bottom: .5rem
}

.ml-4 {
  margin-left: 1rem
}

.mt-16 {
  margin-top: 4rem
}

.mt-4 {
  margin-top: 1rem
}

.flex {
  display: flex
}

.grid {
  display: grid
}

.hidden {
  display: none
}

.h-12 {
  height: 3rem
}

.h-64 {
  height: 16rem
}

.h-full {
  height: 100%
}

.h-24 {
  height: 6rem
}

.h-4 {
  height: 1rem
}

.h-2 {
  height: .5rem
}

.min-h-\[700px\] {
  min-height: 700px
}

.w-full {
  width: 100%
}

.w-24 {
  width: 6rem
}

.w-4 {
  width: 1rem
}

.w-2 {
  width: .5rem
}

.max-w-3xl {
  max-width: 48rem
}

.max-w-lg {
  max-width: 32rem
}

.flex-1 {
  flex: 1 1 0%
}

.flex-shrink-0 {
  flex-shrink: 0
}

.-rotate-12 {
  --tw-rotate: -12deg
}

.-rotate-12,
.transform {
  transform: translate(var(--tw-translate-x), var(--tw-translate-y)) rotate(var(--tw-rotate)) skewX(var(--tw-skew-x)) skewY(var(--tw-skew-y)) scaleX(var(--tw-scale-x)) scaleY(var(--tw-scale-y))
}

.cursor-move {
  cursor: move
}

.cursor-grab {
  cursor: -webkit-grab;
  cursor: grab
}

.cursor-pointer {
  cursor: pointer
}

.cursor-nw-resize {
  cursor: nw-resize
}

.cursor-ne-resize {
  cursor: ne-resize
}

.cursor-se-resize {
  cursor: se-resize
}

.cursor-n-resize {
  cursor: n-resize
}

.cursor-s-resize {
  cursor: s-resize
}

.cursor-e-resize {
  cursor: e-resize
}

.cursor-w-resize {
  cursor: w-resize
}

.select-none {
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none
}

.resize {
  resize: both
}

.grid-cols-2 {
  grid-template-columns: repeat(2, minmax(0, 1fr))
}

.flex-col {
  flex-direction: column
}

.flex-wrap {
  flex-wrap: wrap
}

.items-center {
  align-items: center
}

.justify-center {
  justify-content: center
}

.gap-4 {
  gap: 1rem
}

.gap-1 {
  gap: .25rem
}

.gap-16 {
  gap: 4rem
}

.gap-6 {
  gap: 1.5rem
}

.gap-2 {
  gap: .5rem
}

.gap-3 {
  gap: .75rem
}

.overflow-hidden {
  overflow: hidden
}

.whitespace-nowrap {
  white-space: nowrap
}

.rounded-xl {
  border-radius: .75rem
}

.rounded-lg {
  border-radius: .5rem
}

.border {
  border-width: 1px
}

.bg-gray-200 {
  --tw-bg-opacity: 1;
  background-color: rgb(229 231 235/var(--tw-bg-opacity))
}

.bg-white {
  --tw-bg-opacity: 1;
  background-color: rgb(255 255 255/var(--tw-bg-opacity))
}

.bg-opacity-90 {
  --tw-bg-opacity: 0.9
}

.p-4 {
  padding: 1rem
}

.pb-10 {
  padding-bottom: 2.5rem
}

.text-lg {
  font-size: 1.125rem;
  line-height: 1.75rem
}

.text-3xl {
  font-size: 1.875rem;
  line-height: 2.25rem
}

.text-7xl {
  font-size: 4.5rem;
  line-height: 1
}

.text-8xl {
  font-size: 6rem;
  line-height: 1
}

.text-xs {
  font-size: .75rem;
  line-height: 1rem
}

.text-sm {
  font-size: .875rem;
  line-height: 1.25rem
}

.font-bold {
  font-weight: 700
}

.font-black {
  font-weight: 900
}

.font-semibold {
  font-weight: 600
}

.uppercase {
  text-transform: uppercase
}

.text-gray-500 {
  --tw-text-opacity: 1;
  color: rgb(107 114 128/var(--tw-text-opacity))
}

.text-stone-700 {
  --tw-text-opacity: 1;
  color: rgb(68 64 60/var(--tw-text-opacity))
}

.text-white {
  --tw-text-opacity: 1;
  color: rgb(255 255 255/var(--tw-text-opacity))
}

.opacity-50 {
  opacity: .5
}

.shadow-xl {
  --tw-shadow: 0 20px 25px -5px rgba(0, 0, 0, .1), 0 8px 10px -6px rgba(0, 0, 0, .1);
  --tw-shadow-colored: 0 20px 25px -5px var(--tw-shadow-color), 0 8px 10px -6px var(--tw-shadow-color)
}

.shadow-lg,
.shadow-xl {
  box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow)
}

.shadow-lg {
  --tw-shadow: 0 10px 15px -3px rgba(0, 0, 0, .1), 0 4px 6px -4px rgba(0, 0, 0, .1);
  --tw-shadow-colored: 0 10px 15px -3px var(--tw-shadow-color), 0 4px 6px -4px var(--tw-shadow-color)
}

.shadow {
  --tw-shadow: 0 1px 3px 0 rgba(0, 0, 0, .1), 0 1px 2px -1px rgba(0, 0, 0, .1);
  --tw-shadow-colored: 0 1px 3px 0 var(--tw-shadow-color), 0 1px 2px -1px var(--tw-shadow-color)
}

.shadow,
.shadow-2xl {
  box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow)
}

.shadow-2xl {
  --tw-shadow: 0 25px 50px -12px rgba(0, 0, 0, .25);
  --tw-shadow-colored: 0 25px 50px -12px var(--tw-shadow-color)
}

.shadow-gray-400 {
  --tw-shadow-color: #9ca3af;
  --tw-shadow: var(--tw-shadow-colored)
}

.ring-1 {
  --tw-ring-offset-shadow: var(--tw-ring-inset) 0 0 0 var(--tw-ring-offset-width) var(--tw-ring-offset-color);
  --tw-ring-shadow: var(--tw-ring-inset) 0 0 0 calc(1px + var(--tw-ring-offset-width)) var(--tw-ring-color);
  box-shadow: var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow, 0 0 #0000)
}

.ring-gray-100 {
  --tw-ring-opacity: 1;
  --tw-ring-color: rgb(243 244 246/var(--tw-ring-opacity))
}

.ring-gray-300 {
  --tw-ring-opacity: 1;
  --tw-ring-color: rgb(209 213 219/var(--tw-ring-opacity))
}

.drop-shadow-xl {
  --tw-drop-shadow: drop-shadow(0 20px 13px rgba(0, 0, 0, .03)) drop-shadow(0 8px 5px rgba(0, 0, 0, .08))
}

.drop-shadow,
.drop-shadow-xl {
  filter: var(--tw-blur) var(--tw-brightness) var(--tw-contrast) var(--tw-grayscale) var(--tw-hue-rotate) var(--tw-invert) var(--tw-saturate) var(--tw-sepia) var(--tw-drop-shadow)
}

.drop-shadow {
  --tw-drop-shadow: drop-shadow(0 1px 2px rgba(0, 0, 0, .1)) drop-shadow(0 1px 1px rgba(0, 0, 0, .06))
}

.filter {
  filter: var(--tw-blur) var(--tw-brightness) var(--tw-contrast) var(--tw-grayscale) var(--tw-hue-rotate) var(--tw-invert) var(--tw-saturate) var(--tw-sepia) var(--tw-drop-shadow)
}

.transition-all {
  transition-property: all;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
  transition-duration: .15s
}

.transition-\[shadow\2c transform\] {
  transition-property: shadow, transform;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
  transition-duration: .15s
}

.transition {
  transition-property: color, background-color, border-color, text-decoration-color, fill, stroke, opacity, box-shadow, transform, filter, -webkit-backdrop-filter;
  transition-property: color, background-color, border-color, text-decoration-color, fill, stroke, opacity, box-shadow, transform, filter, backdrop-filter;
  transition-property: color, background-color, border-color, text-decoration-color, fill, stroke, opacity, box-shadow, transform, filter, backdrop-filter, -webkit-backdrop-filter;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
  transition-duration: .15s
}

.transition-shadow {
  transition-property: box-shadow;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
  transition-duration: .15s
}

.ease-in-out {
  transition-timing-function: cubic-bezier(.4, 0, .2, 1)
}

.will-change-\[filter\] {
  will-change: filter
}

#__next,
body,
html {
  width: 100%;
  height: 100%
}

body,
html {
  position: fixed;
  overflow: hidden
}

#__next {
  overflow-y: scroll;
  position: relative
}

* {
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none;
  scrollbar-width: none;
  -ms-overflow-style: none;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0)
}

::-webkit-scrollbar {
  display: none
}

@media (prefers-color-scheme: dark) {
  html {
    color-scheme: dark
  }

  body {
    color: #eee;
    background: #000
  }
}

.text-shadow {
  text-shadow: -1px 0 #ececec, 0 1px #ececec, 1px 0 #ececec, 0 -1px #ececec
}

@-webkit-keyframes shake {

  10%,
  90% {
    transform: translate3d(-1px, 0, 0)
  }

  20%,
  80% {
    transform: translate3d(2px, 0, 0)
  }

  30%,
  50%,
  70% {
    transform: translate3d(-4px, 0, 0)
  }

  40%,
  60% {
    transform: translate3d(4px, 0, 0)
  }
}

@keyframes shake {

  10%,
  90% {
    transform: translate3d(-1px, 0, 0)
  }

  20%,
  80% {
    transform: translate3d(2px, 0, 0)
  }

  30%,
  50%,
  70% {
    transform: translate3d(-4px, 0, 0)
  }

  40%,
  60% {
    transform: translate3d(4px, 0, 0)
  }
}

.shake {
  -webkit-animation: shake .82s cubic-bezier(.36, .07, .19, .97) both;
  animation: shake .82s cubic-bezier(.36, .07, .19, .97) both
}

@-webkit-keyframes confetti-slow {
  0% {
    transform: translateZ(0) rotateX(0) rotateY(0)
  }

  to {
    transform: translate3d(25px, 105vh, 0) rotateX(1turn) rotateY(180deg)
  }
}

@keyframes confetti-slow {
  0% {
    transform: translateZ(0) rotateX(0) rotateY(0)
  }

  to {
    transform: translate3d(25px, 105vh, 0) rotateX(1turn) rotateY(180deg)
  }
}

@-webkit-keyframes confetti-medium {
  0% {
    transform: translateZ(0) rotateX(0) rotateY(0)
  }

  to {
    transform: translate3d(100px, 105vh, 0) rotateX(100deg) rotateY(1turn)
  }
}

@keyframes confetti-medium {
  0% {
    transform: translateZ(0) rotateX(0) rotateY(0)
  }

  to {
    transform: translate3d(100px, 105vh, 0) rotateX(100deg) rotateY(1turn)
  }
}

@-webkit-keyframes confetti-fast {
  0% {
    transform: translateZ(0) rotateX(0) rotateY(0)
  }

  to {
    transform: translate3d(-50px, 105vh, 0) rotateX(10deg) rotateY(250deg)
  }
}

@keyframes confetti-fast {
  0% {
    transform: translateZ(0) rotateX(0) rotateY(0)
  }

  to {
    transform: translate3d(-50px, 105vh, 0) rotateX(10deg) rotateY(250deg)
  }
}

.confetti {
  position: absolute;
  z-index: 1;
  top: -10px;
  border-radius: 0
}

.confetti--animation-slow {
  -webkit-animation: confetti-slow 2.25s linear 1 forwards;
  animation: confetti-slow 2.25s linear 1 forwards
}

.confetti--animation-medium {
  -webkit-animation: confetti-medium 1.75s linear 1 forwards;
  animation: confetti-medium 1.75s linear 1 forwards
}

.confetti--animation-fast {
  -webkit-animation: confetti-fast 1.25s linear 1 forwards;
  animation: confetti-fast 1.25s linear 1 forwards
}

.todo .dnd-item {
  border-width: 1px;
  border-color: transparent
}

.todo .dnd-item.placeholder {
  border-width: 1px;
  --tw-border-opacity: 1;
  border-color: rgb(59 130 246/var(--tw-border-opacity));
  opacity: .5;
  --tw-ring-offset-shadow: var(--tw-ring-inset) 0 0 0 var(--tw-ring-offset-width) var(--tw-ring-offset-color);
  --tw-ring-shadow: var(--tw-ring-inset) 0 0 0 calc(2px + var(--tw-ring-offset-width)) var(--tw-ring-color);
  box-shadow: var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow, 0 0 #0000);
  --tw-ring-opacity: 1;
  --tw-ring-color: rgb(96 165 250/var(--tw-ring-opacity))
}

.todo .dnd-item:not(.ghost) {
  cursor: -webkit-grab;
  cursor: grab;
  --tw-shadow: 0 1px 3px 0 rgba(0, 0, 0, .1), 0 1px 2px -1px rgba(0, 0, 0, .1);
  --tw-shadow-colored: 0 1px 3px 0 var(--tw-shadow-color), 0 1px 2px -1px var(--tw-shadow-color);
  box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow)
}

.hover\:drop-shadow-lg:hover {
  --tw-drop-shadow: drop-shadow(0 10px 8px rgba(0, 0, 0, .04)) drop-shadow(0 4px 3px rgba(0, 0, 0, .1));
  filter: var(--tw-blur) var(--tw-brightness) var(--tw-contrast) var(--tw-grayscale) var(--tw-hue-rotate) var(--tw-invert) var(--tw-saturate) var(--tw-sepia) var(--tw-drop-shadow)
}

.active\:scale-95:active {
  --tw-scale-x: .95;
  --tw-scale-y: .95
}

.active\:scale-95:active,
.active\:scale-\[0\.97\]:active {
  transform: translate(var(--tw-translate-x), var(--tw-translate-y)) rotate(var(--tw-rotate)) skewX(var(--tw-skew-x)) skewY(var(--tw-skew-y)) scaleX(var(--tw-scale-x)) scaleY(var(--tw-scale-y))
}

.active\:scale-\[0\.97\]:active {
  --tw-scale-x: 0.97;
  --tw-scale-y: 0.97
}

.active\:shadow-lg:active {
  --tw-shadow: 0 10px 15px -3px rgba(0, 0, 0, .1), 0 4px 6px -4px rgba(0, 0, 0, .1);
  --tw-shadow-colored: 0 10px 15px -3px var(--tw-shadow-color), 0 4px 6px -4px var(--tw-shadow-color);
  box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow)
}

@media (prefers-color-scheme: dark) {
  .dark\:bg-\[\#121212\] {
    --tw-bg-opacity: 1;
    background-color: rgb(18 18 18/var(--tw-bg-opacity))
  }

  .dark\:bg-\[\#000000\] {
    --tw-bg-opacity: 1;
    background-color: rgb(0 0 0/var(--tw-bg-opacity))
  }
}
```

</div>
</details>


<br/>




### 드래그 이벤트 등록

<details>
<summary>registDragEvent.ts</summary>
<div markdown="1">

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

</div>
</details>

<br/>


### 거리계산 함수 추가

<details>
<summary>Resize.ts</summary>
<div markdown="1">

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

</div>
</details>

### 기본 넓이&높이, 최소 넓이&높이 설정

<details>
<summary>dnd.ts</summary>
<div markdown="1">

```ts
export const BOUNDARY_MARGIN = 12;
export const MIN_W = 80;
export const MIN_H = 80;
export const DEFAULT_W = 240;
export const DEFAULT_H = 120;
```

</div>
</details>

<br/>


### 바운더리 설정

<details>
<summary>Boundary.tsx</summary>
<div markdown="1">

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

</div>
</details>

<br/>


### 바운더리의 레이아웃 설정

<details>
<summary>BoundaryLayout.tsx</summary>
<div markdown="1">

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

    // element의 config(위치, 크기) 상태를 정의
    const [{ x, y, w, h }, setConfig] = useState({
        x: 0,
        y: 0,
        w: 0,
        h: 0,
    });


    // element의 config을 초기
    // boundary를 기준으로 중앙정렬
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
                        // position drag 이벤트를 등록
                        {...registDragEvent((deltaX: number, deltaY: number) => {
                            if (!boundaryRef.current) return;

                            const boundary = boundaryRef.current.getBoundingClientRect();
                            // config 상태를 변경하여 element의 크기를 변화
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

</div>
</details>

<br/>


### Drag Layout 설정

<details>
<summary>DragLayout.tsx</summary>
<div markdown="1">

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
                // resize되는 범위를 잘 설정
                {...registDragEvent((deltaX, deltaY) => {
                    setConfig({
                        firstMenuX: inrange(x + deltaX, BOUNDARY_MARGIN, x + w - MIN_W),
                        firstMenuY: inrange(y + deltaY, BOUNDARY_MARGIN, y + h - MIN_H),
                        firstMenuW: inrange(w - deltaX, MIN_W, x + w - BOUNDARY_MARGIN),
                        firstMenuH: inrange(h - deltaY, MIN_H, y + h - BOUNDARY_MARGIN),
                    });
                }, true)} // resize의 클릭 이벤트가 부모로 전파되지 않도록 stopPropagation
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

<br/>


### 모달 생성

<details>
<summary>ResizeModal.tsx</summary>
<div markdown="1">

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

</div>
</details>

<br/>


### 모달 등록

<details>
<summary>DragMenu.tsx</summary>
<div markdown="1">

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

</div>
</details>

<br/>


### 컴포넌트 호출


```ts
import { FC, useRef, useState, useMemo } from 'react';
import "./css/resize.css";
import BoundaryLayout from "components/dragandresize/BoundaryLayout";

const [isShowFirstMenu, setIsShowFirstMenu] = useState(false);

const openFirstMenu = () => {
    setIsShowFirstMenu(!isShowFirstMenu);
}

const bundaryLayout = useMemo(
    () => {
        return (<BoundaryLayout
            isShowFirstMenu={isShowFirstMenu}
        />)
    },
    [isShowFirstMenu]
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
        </div>
    </>
);

```