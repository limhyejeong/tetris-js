## 💡 알게 된 것

- ES6 문법
- 클래스 활용



</br></br>


# ⚙️ 설계

## Setting

- 테트리스 판 생성
    
    ```html
    <div class="playground">
    	<ul></ul>
    </div>
    ```
    
    ```jsx
    // DOM 가져오기
    const playground = document.querySelector(".playground > ul");
    
    // 가로 세로 값 세팅
    const GAME_ROWS = 20;
    const GAME_COLS = 10;
    
    init();
    
    // 초기 세팅 함수
    function init() {
    	..
    	for (let i = 0; i < GAME_ROWS; i++) {
            prependNewLine();
        }
    	..
    }
    
    // 테트리스 판 생성 함수
    function prependNewLine() {
        const li = document.createElement("li");
        const ul = document.createElement("ul");
        for (let j = 0; j < GAME_COLS; j++) {
            const matrix = document.createElement("li");
            ul.prepend(matrix);
        }
        li.prepend(ul);
        playground.prepend(li);
    }
    ```
    

- 블럭 생성
    
    ```jsx
    // 블럭 생성 함수
    function generateNewBlock() {
    
        clearInterval(downInterval); // 새로 시작되면 진행중인 Interval 중단
    
        downInterval = setInterval(() => {
            moveBlock('top', 1);
        }, duration);
    
        const blockArray = Object.entries(BLOCKS);
        const randomIndex = Math.floor(Math.random() * blockArray.length);
        movingItem.type = blockArray[randomIndex][0];
        
        movingItem.top = 0;
        movingItem.left = 3;
        movingItem.direction = 0;
        tempMovingItem = { ...movingItem };
        renderBlocks();
    }
    ```
    


## Block Object

- 블럭 정보
    - 모양
    - 방향
    - 좌표
    
    ```jsx
    // 움직이는 블럭의 타입 및 좌표 등
    const movingItem = {
        type: "", // 모양
        direction: 0, // 방향
        top: 0, // y좌표
        left: 3, // x좌표
    };
    ```
    

- 사용자 조작
    - 방향키 → ← ↓ : 블럭 이동
        
        ```jsx
        document.addEventListener("keydown", e => {
            switch (e.keyCode) {
                // 우측 키
                case 39:
                    moveBlock("left", 1);
                    break;
                // 좌측 키
                case 37:
                    moveBlock("left", -1);
                    break;
                // 하단 키
                case 40:
                    moveBlock("top", 1);
                    break;
                default:
                    break;
            }
        })
        ```
        
        ```jsx
        document.addEventListener("keydown", e => {
            switch (e.keyCode) {
                // 우측 키
                case 39:
                    moveBlock("left", 1);
                    break;
                // 좌측 키
                case 37:
                    moveBlock("left", -1);
                    break;
                // 하단 키
                case 40:
                    moveBlock("top", 1);
                    break;
                default:
                    break;
            }
        })
        ```
        
    - 방향키 ↑ : 블럭 회전
        
        ```jsx
        ..
            switch (e.keyCode) {
                // 상단 키
                case 38:
                    changeDirection();
                    break;
        ..
        
        ```
        
        ```jsx
        // (위 방향키 입력시) 블럭이 로테이션 되는 함수
        function changeDirection() {
            const direction = tempMovingItem.direction;
        
            direction === 3 ? tempMovingItem.direction = 0 : tempMovingItem.direction += 1;
        
            renderBlocks();
        }
        ```
        
    - 스페이스바 : 최하단으로 블럭 하강
        
        ```jsx
        ..
            switch (e.keyCode) {
               // 스페이스 바
                case 32:
                    dropBlock();
                    break;
        ..
        ```
        
        ```jsx
        // (스페이스 바 입력시) 블럭이 밑으로 쭉 떨어지는 함수
        function dropBlock() {
            clearInterval(downInterval);
            downInterval = setInterval(() => {
                moveBlock("top", 1);
            }, 10);
        }
        ```
        
    



## Function

- 1) 일정 시간마다 아래로 한칸씩 이동
    
    ```jsx
    // 블럭 그리기 함수
    function renderBlocks(moveType = "") {
        // tempMovingItem의 요소 한번에 가져오기
        const { type, direction, top, left } = tempMovingItem;
    
        // 이동 전 칸 블럭 지우기
        const movingBlocks = document.querySelectorAll(".moving");
        movingBlocks.forEach(moving => {
            moving.classList.remove(type, "moving");
        });
    
        // 블럭 그리기
        BLOCKS[type][direction].some(block => {
            const x = block[0] + left;
            const y = block[1] + top;
    
            // 칸을 벗어나는 걸 방지하기 위한 조건문
            const target = playground.childNodes[y] ? playground.childNodes[y].childNodes[0].childNodes[x] : null;
    
            // 블럭이 떨어졌을 때 밑에 빈칸이 있는지 확인
            const isAvailable = checkEmpty(target);
    
            // 빈공간이 있으면
            if (isAvailable) {
                target.classList.add(type, "moving");
            } else {
                // 원상복구
                tempMovingItem = { ...movingItem };
                if (moveType === 'retry') {
                    clearInterval(downInterval);
                    showGameoverText();
                }
                setTimeout(() => { // 이벤트 스택이 넘치는 걸 방지
                    renderBlocks('retry');
    
                    if (moveType === "top") {
                        seizeBlock();
                    }
                }, 0);
                return true;
            }
        })
        movingItem.left = left;
        movingItem.top = top;
        movingItem.direction = direction;
    }
    ```
    

- 2) 아래에 블럭이 있거나 바닥에 닿았을 때
    
    ```jsx
    // 빈공간이 있으면
    if (isAvailable) {
    	target.classList.add(type, "moving");
    }
    else { // 없으면
    	tempMovingItem = { ...movingItem };
    	
    	if (moveType === 'retry') {
    		clearInterval(downInterval);
    		showGameoverText();
    	}
    	
    	setTimeout(() => { // 이벤트 스택이 넘치는 걸 방지
    		renderBlocks('retry');
    		if (moveType === "top") {
    			seizeBlock();
    		}
    	}, 0);
    	
    	return true;
    }
    ```
    
    - 정지
        
        ```jsx
        // 블럭 고정 함수
        function seizeBlock() {
            const movingBlocks = document.querySelectorAll(".moving");
            movingBlocks.forEach(moving => {
                moving.classList.remove("moving");
                moving.classList.add("seized");
            });
        ..
        ```
        
    - 한 줄이 모두 채워졌는지 확인
        
        ```jsx
        ..
            checkMatch();
        }
        ```
        

- 3) 한 줄이 모두 찰 때
    - 해당 줄 삭제
    - 새로운 줄 생성
    - 득점
        
        ```jsx
        // 한 줄이 모두 채워졌는지 확인하는 함수
        function checkMatch() {
            const childNodes = playground.childNodes;
        
            childNodes.forEach(child => {
                let matched = true;
                child.children[0].childNodes.forEach(li => {
                    if (!li.classList.contains("seized")) {
                        matched = false;
                    }
                })
                if(matched) {
                    child.remove();
                    prependNewLine();
                    score++;
                    scoreDisplay.innerHTML = score;
                }
            })
        
        		// 새 블럭 생성
            generateNewBlock();
        }
        ```
        

- 4) 블럭이 넘쳐 더이상 내려갈 수 없을 때
    - 게임 오버 창 표시
    
    ```jsx
    // 게임 오버 함수
    function showGameoverText() {
        gameText.style.display = "flex";
    }
    ```
