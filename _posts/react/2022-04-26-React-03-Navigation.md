---
title:  "[React] Navigation"
excerpt: "React Navigation 만들기"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React Navigation]"
 
date: 2022-04-26
last_modified_at: 2022-04-26
---

- 리액트로 게시판 만들기
  1.  [VSCode 와 Node 설치하기](https://ymkmoon.github.io/React-02-React/)
  2.  [리액트 프로젝트와 네비게이션 생성하기](https://ymkmoon.github.io/React-03-Navigation/)
  3.  [페이지 이동을 위한 Route 설정하기](https://ymkmoon.github.io/React-04-Route/)
  4.  [Axios 설치 및 사용해보기](https://ymkmoon.github.io/React-05-Axios/)
  5.  [Axios 를 이용하여 게시판 목록 만들기](https://ymkmoon.github.io/React-06-Voc/)
  6.  [Axios 를 이용하여 게시글 상세보기 만들기](https://ymkmoon.github.io/React-07-Voc-Question/)
  7.  [Axios 를 이용하여 게시글 작성 만들기](https://ymkmoon.github.io/React-08-Voc-Post-Question/)
  8.  [Apollo GraphQL 을 설치 및 사용해보기](https://ymkmoon.github.io/React-09-Apollo/)
  9.  [Apollo GraphQL 을 이용하여 게시판 목록 만들기](https://ymkmoon.github.io/React-10-Apollo-Voc/)
  10.  [Apollo GraphQL 을 이용하여 게시글 상세보기 만들기](https://ymkmoon.github.io/React-11-Apollo-Voc-Question/)
  11.  [Apollo GraphQL 을 이용하여 게시글 작성 만들기](https://ymkmoon.github.io/React-12-Apollo-Voc-Post-Question/)

> React 소스코드는 [여기](https://github.com/ymkmoon/toyseven-react){:target="_blank"} 에서 확인 가능합니다.

> 개발에 사용한 Restful API 는 [여기](https://github.com/ymkmoon/toyseven){:target="_blank"} 에서 확인 가능합니다.

<hr/>

### ``React 설치``
  
  ```console
  cd C:\ymk\toynumber\toyseven-react  # workspace 로 이동
  npm install -g create-react-app     # react 설치
  
  # 본인이 설치 할 경로에 따라 cra(create react app) 실행
  create-react-app toyseven-react     # toyseven-react 경로에 cra
  create-react-app .                  # 현재 경로에 cra
  ```

  ``create-react-app toyseven-react`` 명령어 에서 ``toyseven-react`` 는 애플리케이션 폴더이름이며, React 애플리케이션을 만들고 의존 라이브러리를 설치하는 내용이다.

### ``React 실행``

  ```console
  cd C:\ymk\toynumber\toyseven-react
  npm start
  ```

  위 명령어를 실행 후 http://localhost:3000/ 로 접속했을때 React 페이지가 나오면 서버가 정상적으로 실행 된 것

![React](/assets/image/react/React_toyseven_react_01.PNG)


### ``Navigation bar 생성``

  ```console
  npm install react-router-dom --save   # 페이지를 이동할 때 필요한 라이브러리
  ```

🛠 components/Button.css

```css
:root {
    --primary: #fff;
}

.btn{
    padding: 8px 20px;
    border-radius: 2px;
    outline: none;
    border: none;
    cursor: pointer;
}

.btn--primary{
    background-color: var(--primary);
    color: #242424;
    border: 1px solid var(--primary);
}

.btn--outline {
    background-color: transparent;
    color: white;
    padding:8px 20px;
    border: 1px solid var(--primary);
    transition: all 0.3s ease-out;
}

.btn-medium {
    padding: 8px 20px;
    font-size: 20px;

}

.btn--large {
    padding: 12px 26px;
    font-size: 20px;

}


.btn--medium:hover, 
.btn--large:hover{
    background: white;
    color: #242424;
    transition: all 0.3s ease-out;

}
```

<br>

🛠 components/Button.js

```js
import React from 'react';
import './Button.css';
import {Link} from 'react-router-dom';

const STYLES = ['btn--primary', 'btn--outline'];

const SIZES = ['btn--medium', 'btn--large'];

export const Button = ({children, type, onClick, buttonStyle, buttonSize}) => {
    // buttonstyle이 따로 지정되지 않으면 가장 기본인 배열의 0번째가 설정된다. 
    const checkButtonStyle = STYLES.includes(buttonStyle) ? buttonStyle : STYLES[0];
    // buttonsize가 지정되지 않으면 가장 기본 사이즈인 0번재 btn-medium이 설정된다. 
    const checkButtonSize = SIZES.includes(buttonSize) ? buttonSize : SIZES[0];

    return (
        // children안에 어떤 내용을 입력해도 버튼 이름이 된다. 
        <Link to = '/sign-up' className = 'btn-mobile'>
            <button 
            className={`btn ${checkButtonStyle} ${checkButtonSize}`}
            onClick={onClick}
            type={type}
            >
                {children} 
            </button>
        </Link>
    );
    
};
```

<br>

🛠 components/Navbar.css

```css
.navbar {
    background: linear-gradient(90deg, rgb(28, 27, 27) 0%, rgb(26, 23, 23) 100%);
    height: 80px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.2rem;
    position: sticky;
    top: 0;
    z-index: 999;
  }
  
  .navbar-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 80px;
    max-width: 1500px;
  }
  
  .navbar-logo {
    color: #fff;
    justify-self: start;
    margin-left: 20px;
    cursor: pointer;
    text-decoration: none;
    font-size: 2rem;
    display: flex;
    align-items: center;
  }
  
  .fa-typo3 {
    margin-left: 0.5rem;
    font-size: 1.8rem;
  }
  
  .nav-menu {
    display: grid;
    grid-template-columns: repeat(4, auto);
    grid-gap: 10px;
    list-style: none;
    text-align: center;
    width: 60vw;
    justify-content: end;
    margin-right: 2rem;
  }
  
  .nav-item {
    height: 80px;
  }
  
  .nav-links {
    color: #fff;
    display: flex;
    align-items: center;
    text-decoration: none;
    padding: 0.5rem 1rem;
    height: 100%;
  }
  
  .nav-links:hover {
    border-bottom: 4px solid #fff;
    transition: all 0.2s ease-out;
  }
  
  .fa-bars {
    color: #fff;
  }
  
  .nav-links-mobile {
    display: none;
  }
  
  .menu-icon {
    display: none;
  }
  
  @media screen and (max-width: 960px) {
    .NavbarItems {
      position: relative;
    }
  
    .nav-menu {
      display: flex;
      flex-direction: column;
      width: 100%;
      height: 90vh;
      position: absolute;
      top: 80px;
      left: -100%;
      opacity: 1;
      transition: all 0.5s ease;
    }
  
    .nav-menu.active {
      background: #242222;
      left: 0;
      opacity: 1;
      transition: all 0.5s ease;
      z-index: 1;
    }
  
    .nav-links {
      text-align: center;
      padding: 2rem;
      width: 100%;
      display: table;
    }
  
    .nav-links:hover {
      background-color: #fff;
      color: #242424;
      border-radius: 0;
    }
  
    .navbar-logo {
      position: absolute;
      top: 0;
      left: 0;
      transform: translate(25%, 50%);
    }
  
    .menu-icon {
      display: block;
      position: absolute;
      top: 0;
      right: 0;
      transform: translate(-100%, 60%);
      font-size: 1.8rem;
      cursor: pointer;
    }
  
    .fa-times {
      color: #fff;
      font-size: 2rem;
    }
  
    .nav-links-mobile {
      display: block;
      text-align: center;
      margin: 2rem auto;
      border-radius: 4px;
      width: 80%;
      text-decoration: none;
      font-size: 1.5rem;
      background-color: transparent;
      color: #fff;
      padding: 14px 20px;
      border: 1px solid #fff;
      transition: all 0.3s ease-out;
    }
  
    .nav-links-mobile:hover {
      background: #fff;
      color: #242424;
      transition: 250ms;
    }
  }
```

<br>

🛠 components/Navbar.js

```js
import React, { useState, useEffect} from 'react';
import { Link } from 'react-router-dom';
import {Button} from './Button';
import './Navbar.css';

function Navbar() {
    const [click, setClick] = useState(false);
    const [button, setButton] = useState(true);


    const handleClick = () => setClick(!click) ;
    const closeMobileMenu = () => setClick(false);

    // 화면 크기에 따라서 버튼이 보이고 안보이도록 설정한다. 
    const showButton = () => {
        if(window.innerWidth <= 960){
            setButton(false)
        }
        else {
            setButton(true);
        }
    };

    // SIGNUP버튼이 사이즈가 줄어들면 없어지도록 한다. 
    useEffect(() => {
        showButton();
    }, []);


    window.addEventListener('resize', showButton);

    return (
        <>
        <nav className = 'navbar'>
            <div className = 'navbar-container'>
                {/* 모바일버전에서 클릭하면 메뉴 보이도록 설정하는 것도 한다. (close Mobile Menu)는 다시 버튼 누르면 없어지고 생기고 하도록 한다.  */}
                <Link to='/' className='navbar-logo' onClick={closeMobileMenu}>
                    Toyseven-Request 
                    <i className='fab fa-typo3' />
                </Link>
                <div className='menu-icon' onClick={handleClick}>
                    <i className = {click ? 'fas fa-times' : 'fas fa-bars' } />
                </div>
                <ul className={click ? 'nav-menu active' : 'nav-menu'}>
                    <li className='nav-item'>
                        <Link to='/' className='nav-links' onClick = {closeMobileMenu}>
                            Home
                        </Link>
                    </li>
                    <li className='nav-item'>
                        <Link to='/station' className='nav-links' onClick = {closeMobileMenu}>
                            Station
                        </Link>
                    </li>
                    <li className='nav-item'>
                        <Link to='/voc' className='nav-links' onClick = {closeMobileMenu}>
                            Voc
                        </Link>
                    </li>
                    <li className='nav-item'>
                        <Link to='/sign-up' className='nav-links-mobile' onClick = {closeMobileMenu}>
                            Sign Up
                        </Link>
                    </li>
                </ul>
                {button && <Button buttonStyle='btn--outline'>SIGN UP</Button>}
            </div>
        </nav>
        </>
    );
}

export default Navbar
```

<br>

🛠 App.js

```js
import React from 'react';
import {BrowserRouter as Router, Route, Routes} from 'react-router-dom';
import Navbar from'./components/Navbar';

function App() {
    

    return (
      <>
      <Router>
        <Navbar />
        <Routes>
          <Route path='/' exact />
        </Routes>
      </Router>
      </>
    );
}
export default App;
```

![React](/assets/image/react/React_toyseven_react_02.PNG)
