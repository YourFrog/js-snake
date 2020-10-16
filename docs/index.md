    <script
            src="https://code.jquery.com/jquery-3.5.1.min.js"
            integrity="sha256-9/aliU8dGd2tb6OSsuzixeV4y/faTqgFtohetphbbj0="
            crossorigin="anonymous"></script>
    <script>
        // Not mine: https://stackoverflow.com/a/7228322
        function randomIntFromInterval(min, max) { // min and max included
            return Math.floor(Math.random() * (max - min + 1) + min);
        }

        let GAME_WIDTH = 36
        let GAME_HEIGHT = 18

        const STATE_WAITING = 'waiting'
        const STATE_GAME = 'game'
        const STATE_GAME_OVER = 'game_over'

        var state = STATE_WAITING

        class Snake
        {
            constructor() {
                /** @type {SnakePart} */
                this.head = undefined

                /** @type {SnakePart} */
                this.tail = undefined

                this.setDirection(direction.EAST)
            }

            move() {
                // Get last "part" to memory
                let memoryHead = this.tail

                // Move pre-last to tail
                this.tail = memoryHead.getPrev()
                this.tail.setNext(undefined) // Reset next element to undefined

                // Move current head to "secondary" element
                this.head.setPrev(memoryHead)
                memoryHead.setNext(this.head)

                // Move memory head to new head position
                memoryHead.setPosition(this.head.getValue().x + this.direction.x, this.head.getValue().y + this.direction.y)

                // Change current head
                this.head = memoryHead

                // Check collision
                let headPosition = this.head.getValue()
                let current = this.head.getNext()

                while(current !== undefined) {
                    let currentPositon = current.getValue()

                    if (headPosition.x === currentPositon.x && headPosition.y === currentPositon.y) {
                        throw 'You ate yourself'
                    }

                    current = current.getNext()
                }
            }

            addPart(candidate) {
                if (this.head === undefined && this.tail === undefined) {
                    this.head = candidate
                    this.tail = candidate

                    return;
                }

                candidate.setPrev(this.tail)
                this.tail.setNext(candidate)

                this.tail = candidate
                this.tail.setNext(undefined)
            }

            getTail() {
                return this.tail
            }

            getHead() {
                return this.head
            }

            setDirection(candidate) {
                if (this.head !== undefined) {
                    // Verify it next move will be back block set direction
                    let primary = this.head
                    let secondary = primary.getNext()

                    let memoryValue = primary.clone().getValue()
                    memoryValue.x = memoryValue.x + candidate.x
                    memoryValue.y = memoryValue.y + candidate.y

                    if (secondary.getValue().x === memoryValue.x && secondary.getValue().y === memoryValue.y) {
                        return
                    }
                }

                this.direction = candidate
            }

            /**
             * @param {OffscreenCanvasRenderingContext2D} context
             */
            draw(context) {
                context.strokeStyle = 'white'

                let current = this.head

                while(current !== undefined) {

                    let pos = current.getValue()

                    context.beginPath()
                    if( current.getValue().x === this.head.getValue().x && current.getValue().y === this.head.getValue().y) {
                        context.fillStyle = "#A9A9A9";
                        context.fillRect(pos.x * 16 + 1, pos.y * 16 + 1, 14, 14)
                        context.fill()
                    } else {
                        context.rect(pos.x * 16 + 2, pos.y * 16 + 2, 12, 12)
                        context.stroke()
                    }
                    
                    current = current.getNext()
                }
            }
        }

        class SnakePart
        {
            constructor(x, y) {
                this._prev = undefined
                this._next = undefined
                this.setPosition(x, y)
            }

            setPosition(x, y) {
                this._value = {
                    'x': x,
                    'y': y
                }
            }

            setPrev(value) {
                this._prev = value
            }

            setNext(value) {
                this._next = value
            }

            getValue() {
                return this._value
            }

            getPrev() {
                return this._prev
            }

            getNext() {
                return this._next
            }

            clone() {
                return new SnakePart(this._value.x, this._value.y)
            }
        }


        function drawGame()
        {
            const canvas = $('canvas#game').get(0)
            const context = canvas.getContext('2d');

            context.canvas.width  = 36 * 16 ;
            context.canvas.height = 18 * 16;
            context.fillStyle = 'black'
            context.fillRect(0, 0, canvas.width, canvas.height);

            switch(state) {
                case STATE_GAME:
                        snake.draw(context)
                        fruit.draw(context)
                    break;

                case STATE_GAME_OVER:
                        $('canvas#next').hide()
                        context.fillStyle = "red";
                        context.textAlign = "center";
                        context.font = "30px Arial";
                        context.fillText("Game over", canvas.width / 2, canvas.height / 2);
                    break;

                case STATE_WAITING:
                        $('canvas#next').hide()

                        context.fillStyle = "red";
                        context.textAlign = "center";
                        context.font = "16px Arial";
                        context.fillText("Click for start", canvas.width / 2, canvas.height / 2);
                    break;
            }
        }

        class Fruit
        {
            constructor() {
                this.setPosition(10, 5)
            }

            setPosition(x, y) {
                this.x = x
                this.y = y
            }

            /**
             * @param {OffscreenCanvasRenderingContext2D} context
             */
            draw(context) {
                context.strokeStyle = 'green'

                context.beginPath()
                context.rect(this.x * 16, this.y * 16, 16, 16)
                context.stroke()
            }

            relocation(snake) {
                let foundPart = false

                do {
                    foundPart = false
                    this.x = randomIntFromInterval(0, GAME_WIDTH - 1)
                    this.y = randomIntFromInterval(0, GAME_HEIGHT - 1)

                    let current = snake.getHead().getNext()

                    while(current !== undefined) {
                        let currentPositon = current.getValue()

                        if (this.x === currentPositon.x && this.y === currentPositon.y) {
                            foundPart = true
                            break
                        }

                        current = current.getNext()
                    }
                } while(foundPart)
            }
        }

        let keyboard = {
            arrowUp: 38,
            arrowRight: 39,
            arrowDown: 40,
            arrowLeft: 37
        }

        let direction = {
            NORTH: {x:  0, y: -1},
            EAST:  {x:  1, y:  0},
            SOUTH: {x:  0, y:  1},
            WEST:  {x: -1, y:  0},
        }

        let points = 0
        let snake = new Snake()
        snake.addPart(new SnakePart(5, 5))
        snake.addPart(new SnakePart(4, 5))
        snake.addPart(new SnakePart(3, 5))

        let fruit = new Fruit()

        $(document).ready(() => {
            drawGame()
            let i = 0

            $('canvas#game').click(() => {
                if( state !== STATE_GAME ) {
                    points = 0
                    snake = new Snake()
                    snake.addPart(new SnakePart(5, 5))
                    snake.addPart(new SnakePart(4, 5))
                    snake.addPart(new SnakePart(3, 5))

                    state = STATE_GAME
                    drawGame()
                }
            })


            $(document).keyup((event) => {
                switch(event.keyCode) {
                    case keyboard.arrowUp:
                            snake.setDirection(direction.NORTH)
                        break;
                    case keyboard.arrowRight:
                            snake.setDirection(direction.EAST)
                        break;
                    case keyboard.arrowDown:
                            snake.setDirection(direction.SOUTH)
                        break;
                    case keyboard.arrowLeft:
                            snake.setDirection(direction.WEST)
                        break;
                }
            })

            setInterval(() => {
                if( state !== STATE_GAME ) {
                    return
                }

                i++
                if( i % 15 === 0 ) {
                    i = 0

                    try {
                        let tail = snake.getTail().getValue()
                        snake.move()

                        let head = snake.getHead().getValue()

                        if (head.x <= -1 || head.x > GAME_WIDTH - 1 || head.y <= -1 || head.y > GAME_HEIGHT - 1) {
                            throw "You left the map"
                        }

                        if (head.x === fruit.x && head.y === fruit.y) {
                            snake.addPart(new SnakePart(tail.x, tail.y))
                            fruit.relocation(snake)

                            points += 10
                        }
                    } catch (e) {
                        alert('Koniec gry')
                        state = STATE_GAME_OVER
                    }

                    let element = $('#points')
                    element.html(points)
                }

                drawGame()
            }, 1)
        })
    </script>

    <style>
        #game {
            width: 576px;
            height: 288px;
            border: 3px solid orange;
        }
    </style>
    
    <p>Punkty: <span id="points">0</span></p>
    <canvas id="game"></canvas>
