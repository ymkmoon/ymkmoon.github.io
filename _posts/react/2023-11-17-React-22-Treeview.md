---
title:  "[React] Tree View 사용하기 (Typescript)"
excerpt: "아이템을 Tree 형태로 보여주기"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React Tree View]"
 
date: 2023-11-17
last_modified_at: 2023-11-17
---

### ``React Grid Layout``

- 참고사이트
    - [Document](https://mui.com/x/react-tree-view/){:target="_blank"}
    - [npm](https://www.npmjs.com/package/@mui/x-tree-view){:target="_blank"}

<br>


### `시작하기 전에`

트리뷰와 DND 기능을 결합하는 기능을 구현해야 한다. <br/>

결합 전에 앞서, TreeView 사용 방법을 알아보자.

<br/>


### Tree View 란?

간단하다. 아이템을 말그대로 트리형태로 보여준다. 이게 끝.<br/>
<br/>

트리형태란 아래와 같다. <br/>
<br/>

![Algorithm](/assets/image/java/Java_Algorithm_Binary_Search_01.PNG)

<br/>


다만, 데이터 형태를 보여줄때(개발자입장)에서는 직관적이지만, 핸들링 하는 유저 입장에서는 불편 할 수 있으니 아래와 같이 표시한다.

```
src
 ┣ constants
 ┃ ┣ dnd.ts
 ┣ components
```

<br/>

이제 실습해보자.

<br/>


### 모듈 설치

```
npm i @mui/x-tree-view
```



### 트리 생성

> Tree.tsx

```ts
import Box from '@mui/material/Box';
import ExpandMoreIcon from '@mui/icons-material/ExpandMore';
import ChevronRightIcon from '@mui/icons-material/ChevronRight';
import { TreeView } from '@mui/x-tree-view/TreeView';
import { TreeItem } from '@mui/x-tree-view/TreeItem';

const Tree = () => {
    return (<>
        <Box sx={{ minHeight: 180, flexGrow: 1, maxWidth: 300 }}>
            <TreeView
                aria-label="file system navigator"
                defaultCollapseIcon={<ExpandMoreIcon />}
                defaultExpandIcon={<ChevronRightIcon />}
            >
                <TreeItem nodeId="1" label="Applications">
                    <TreeItem nodeId="2" label="Calendar" />
                </TreeItem>
                <TreeItem nodeId="5" label="Documents">
                    <TreeItem nodeId="10" label="OSS" />
                    <TreeItem nodeId="6" label="MUI">
                        <TreeItem nodeId="8" label="index.js" />
                    </TreeItem>
                </TreeItem>
            </TreeView>
        </Box>
    </>)
}

export default Tree;
```


### 결과화면

<img src="https://ymkmoon.github.io/assets/image/react/React_Treeview_01.PNG" width="100%" height="100%"/>