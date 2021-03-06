π https://limteractive.github.io/tetris-js/

</br>

## π‘ μκ² λ κ²

- ES6 λ¬Έλ²
- ν΄λμ€ νμ©

</br>

# βοΈΒ μ€κ³

## Setting

- ννΈλ¦¬μ€ ν μμ±
    
    ```html
    <div class="playground">
    	<ul></ul>
    </div>
    ```
    
    ```jsx
    // DOM κ°μ Έμ€κΈ°
    const playground = document.querySelector(".playground > ul");
    
    // κ°λ‘ μΈλ‘ κ° μΈν
    const GAME_ROWS = 20;
    const GAME_COLS = 10;
    
    init();
    
    // μ΄κΈ° μΈν ν¨μ
    function init() {
    	..
    	for (let i = 0; i < GAME_ROWS; i++) {
            prependNewLine();
        }
    	..
    }
    
    // ννΈλ¦¬μ€ ν μμ± ν¨μ
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
    

- λΈλ­ μμ±
    
    ```jsx
    // λΈλ­ μμ± ν¨μ
    function generateNewBlock() {
    
        clearInterval(downInterval); // μλ‘ μμλλ©΄ μ§νμ€μΈ Interval μ€λ¨
    
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

- λΈλ­ μ λ³΄
    - λͺ¨μ
    - λ°©ν₯
    - μ’ν
    
    ```jsx
    // μμ§μ΄λ λΈλ­μ νμ λ° μ’ν λ±
    const movingItem = {
        type: "", // λͺ¨μ
        direction: 0, // λ°©ν₯
        top: 0, // yμ’ν
        left: 3, // xμ’ν
    };
    ```
    

- μ¬μ©μ μ‘°μ
    - λ°©ν₯ν€ β β β : λΈλ­ μ΄λ
        
        ```jsx
        document.addEventListener("keydown", e => {
            switch (e.keyCode) {
                // μ°μΈ‘ ν€
                case 39:
                    moveBlock("left", 1);
                    break;
                // μ’μΈ‘ ν€
                case 37:
                    moveBlock("left", -1);
                    break;
                // νλ¨ ν€
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
                // μ°μΈ‘ ν€
                case 39:
                    moveBlock("left", 1);
                    break;
                // μ’μΈ‘ ν€
                case 37:
                    moveBlock("left", -1);
                    break;
                // νλ¨ ν€
                case 40:
                    moveBlock("top", 1);
                    break;
                default:
                    break;
            }
        })
        ```
        
    - λ°©ν₯ν€ β : λΈλ­ νμ 
        
        ```jsx
        ..
            switch (e.keyCode) {
                // μλ¨ ν€
                case 38:
                    changeDirection();
                    break;
        ..
        
        ```
        
        ```jsx
        // (μ λ°©ν₯ν€ μλ ₯μ) λΈλ­μ΄ λ‘νμ΄μ λλ ν¨μ
        function changeDirection() {
            const direction = tempMovingItem.direction;
        
            direction === 3 ? tempMovingItem.direction = 0 : tempMovingItem.direction += 1;
        
            renderBlocks();
        }
        ```
        
    - μ€νμ΄μ€λ° : μ΅νλ¨μΌλ‘ λΈλ­ νκ°
        
        ```jsx
        ..
            switch (e.keyCode) {
               // μ€νμ΄μ€ λ°
                case 32:
                    dropBlock();
                    break;
        ..
        ```
        
        ```jsx
        // (μ€νμ΄μ€ λ° μλ ₯μ) λΈλ­μ΄ λ°μΌλ‘ μ­ λ¨μ΄μ§λ ν¨μ
        function dropBlock() {
            clearInterval(downInterval);
            downInterval = setInterval(() => {
                moveBlock("top", 1);
            }, 10);
        }
        ```
        
    



## Function

- 1) μΌμ  μκ°λ§λ€ μλλ‘ νμΉΈμ© μ΄λ
    
    ```jsx
    // λΈλ­ κ·Έλ¦¬κΈ° ν¨μ
    function renderBlocks(moveType = "") {
        // tempMovingItemμ μμ νλ²μ κ°μ Έμ€κΈ°
        const { type, direction, top, left } = tempMovingItem;
    
        // μ΄λ μ  μΉΈ λΈλ­ μ§μ°κΈ°
        const movingBlocks = document.querySelectorAll(".moving");
        movingBlocks.forEach(moving => {
            moving.classList.remove(type, "moving");
        });
    
        // λΈλ­ κ·Έλ¦¬κΈ°
        BLOCKS[type][direction].some(block => {
            const x = block[0] + left;
            const y = block[1] + top;
    
            // μΉΈμ λ²μ΄λλ κ±Έ λ°©μ§νκΈ° μν μ‘°κ±΄λ¬Έ
            const target = playground.childNodes[y] ? playground.childNodes[y].childNodes[0].childNodes[x] : null;
    
            // λΈλ­μ΄ λ¨μ΄μ‘μ λ λ°μ λΉμΉΈμ΄ μλμ§ νμΈ
            const isAvailable = checkEmpty(target);
    
            // λΉκ³΅κ°μ΄ μμΌλ©΄
            if (isAvailable) {
                target.classList.add(type, "moving");
            } else {
                // μμλ³΅κ΅¬
                tempMovingItem = { ...movingItem };
                if (moveType === 'retry') {
                    clearInterval(downInterval);
                    showGameoverText();
                }
                setTimeout(() => { // μ΄λ²€νΈ μ€νμ΄ λμΉλ κ±Έ λ°©μ§
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
    

- 2) μλμ λΈλ­μ΄ μκ±°λ λ°λ₯μ λΏμμ λ
    
    ```jsx
    // λΉκ³΅κ°μ΄ μμΌλ©΄
    if (isAvailable) {
    	target.classList.add(type, "moving");
    }
    else { // μμΌλ©΄
    	tempMovingItem = { ...movingItem };
    	
    	if (moveType === 'retry') {
    		clearInterval(downInterval);
    		showGameoverText();
    	}
    	
    	setTimeout(() => { // μ΄λ²€νΈ μ€νμ΄ λμΉλ κ±Έ λ°©μ§
    		renderBlocks('retry');
    		if (moveType === "top") {
    			seizeBlock();
    		}
    	}, 0);
    	
    	return true;
    }
    ```
    
    - μ μ§
        
        ```jsx
        // λΈλ­ κ³ μ  ν¨μ
        function seizeBlock() {
            const movingBlocks = document.querySelectorAll(".moving");
            movingBlocks.forEach(moving => {
                moving.classList.remove("moving");
                moving.classList.add("seized");
            });
        ..
        ```
        
    - ν μ€μ΄ λͺ¨λ μ±μμ‘λμ§ νμΈ
        
        ```jsx
        ..
            checkMatch();
        }
        ```
        

- 3) ν μ€μ΄ λͺ¨λ μ°° λ
    - ν΄λΉ μ€ μ­μ 
    - μλ‘μ΄ μ€ μμ±
    - λμ 
        
        ```jsx
        // ν μ€μ΄ λͺ¨λ μ±μμ‘λμ§ νμΈνλ ν¨μ
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
        
        		// μ λΈλ­ μμ±
            generateNewBlock();
        }
        ```
        

- 4) λΈλ­μ΄ λμ³ λμ΄μ λ΄λ €κ° μ μμ λ
    - κ²μ μ€λ² μ°½ νμ
    
    ```jsx
    // κ²μ μ€λ² ν¨μ
    function showGameoverText() {
        gameText.style.display = "flex";
    }
    ```
