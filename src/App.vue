<template>
        <div class="container">
            <div class="side side--human">

            <div class="ships">
                <div class="ships__list">
                    <div
                        v-for="ship in ships"
                        @click="selectShip(ship)"
                        :key="ship.id"
                        class="ships__size"
                        :class="[`ships__size--${ship.length}`, {'not-available': ship.human.isReady || ship.computer.isSinked }]"
                    >{{ ship.name }}</div>
                </div>
                <template v-if="!isPlaying">
                    <button class="button" :disabled="!selectedShip" :class="{ 'button--disabled': !selectedShip }" @click="changeShipDirection">Change ship direction</button>
                    <button class="button" :disabled="!isPlayerReady" :class="{ 'button--disabled': !isPlayerReady }" @click="startGame">Start Game</button>
                    <button class="button" @click="placeShipsRandom">Place ships randomly</button>
                </template>
            </div>
            <div class="grid grid--human" style="margin-right:10rem">
                <div
                    v-for="cell in grid" :key="cell.id"
                    @mouseover="moveShipOnGrid({x: cell.x, y: cell.y})"
                    @mouseout="removeShip"
                    @click="addShipIdToGrid({x: cell.x, y: cell.y}, grid)"
                    class="grid__cell"
                    :class="{ 'grid__cell--hit': cell.hit, 'grid__cell--placed': !cell.isEmpty && !cell.hit, 'grid__cell--hovering': cell.isHovered, 'grid__cell--missed': cell.missed}"
                ></div>
            </div>
            </div>
            <transition name="slide">
                <div v-show="isPlaying && AIGrid.length > 0" class="side side--computer">

                <div class="grid grid--computer">
                    <div
                        v-for="cell in AIGrid" :key="cell.id"
                        @click="attackShip({x: cell.x, y: cell.y})"
                        class="grid__cell"
                        :class="{ 'grid__cell--hit': cell.hit, 'grid__cell--missed': cell.missed }"
                    ></div>
                </div>
                <div class="ships">
                    <div class="ships__list">
                        <div
                            v-for="ship in ships"
                            @click="selectShip(ship)"
                            :key="ship.id"
                            class="ships__size"
                            :class="[`ships__size--${ship.length}`, {'not-available': ship.human.isSinked }]"
                        >{{ ship.name }}</div>
                    </div>
                </div>
                </div>
            </transition>


        </div>
</template>

<script>
import shipsList from "./assets/js/ships";
export default {
    data() {
        return {
            isPlaying: false,
            gridSize: 10,
            ships: shipsList,
            firstHit : '',
            grid: [],
            AIGrid: []
        };
    },
    computed: {
        isPlayerReady() {
            return this.ships.every(ship => ship.human.isReady);
        },
        selectedShip() {
            return this.ships.find(ship => ship.isSelected);
        }
    },
    methods: {
        attackShip(coordinates) {
            const cell = this.AIGrid.find(
                cell => cell.x === coordinates.x && cell.y === coordinates.y
            );
            const ship = this.ships[cell.shipId];
            if (cell.isEmpty && !cell.missed) {
                cell.missed = true;
            } else if (cell.hit || cell.missed) {
                return    
            } else {
                cell.hit = true;
                ship.human.hitCounter += 1;
                // check if the ship is sinked
                if (ship.length === ship.human.hitCounter)
                    ship.human.isSinked = true;
                // check all ships are sinked
                if (this.ships.every(ship => ship.human.isSinked)) {
                    alert("You won")
                    this.resetGame()
                    this.createGrid(this.grid)
                }
            }
            this.AIAttack();
        },
        AIAttack() {
            let randomX = Math.floor(Math.random() * 10);
            let randomY = Math.floor(Math.random() * 10);

            let cell = this.grid.find(
                cell => cell.x === randomX && cell.y === randomY
            )

            if (!this.firstHit) {
                if (this.grid.filter(target => target.y === randomY && (target.x === randomX + 1 || target.x === randomX - 1) ||
                    target.x === randomX && (target.y === randomY + 1 || target.y === randomY - 1))
                    .some(target => target.missed)) return this.AIAttack()
            }
            
            if (this.firstHit && this.grid.some(cell => cell.prob === 1)) {
                const cells = this.grid.filter(cell => cell.prob === 1)
                const randomCell = Math.floor(Math.random() * cells.length)
                cell = cells[randomCell]
                randomX = cell.x
                randomY = cell.y
            }

            if (cell.isEmpty && !cell.missed) {
                cell.prob = 0
                return cell.missed = true
            } else if (cell.missed || cell.hit){
                return this.AIAttack()
            }

            const ship = this.ships[cell.shipId];
            cell.hit = true;
            ship.computer.hitCounter += 1;
                
                cell.prob = 0
                this.grid.filter(target => target.y === randomY && (target.x === randomX + 1 || target.x === randomX - 1) ||
                    target.x === randomX && (target.y === randomY + 1 || target.y === randomY - 1))
                    .forEach(target => {
                        if (!target.hit && !target.missed) {
                            target.prob = 1
                        }
                    })
                if (this.firstHit && this.firstHit.x === cell.x) {
                    this.grid.filter(target => target.prob === 1 && target.x !== cell.x).forEach(target => target.prob = 0)
                } else if (this.firstHit && this.firstHit.y === cell.y) {
                    this.grid.filter(target => target.prob === 1 && target.y !== cell.y).forEach(target => target.prob = 0)
                }
                if (!this.firstHit) {
                    this.firstHit = cell
                } 
            // check if the ship is sinked and if not record the coordinates and the ship ID for the next attack
            if (ship.length === ship.computer.hitCounter) {
                ship.computer.isSinked = true;
                this.firstHit = ''
                this.grid.forEach(cell => cell.prob = 0)
            }
            
            if (this.ships.every(ship => ship.computer.isSinked)) {
                alert("You lose")
                this.resetGame()
                this.createGrid(this.grid)
            }
        },
        addShipIdToGrid(coordinates, grid) {
            if (!this.selectedShip) return;
            let shipCells;
            //   check if the ship exceeds the grid limit
            if (
                !this.selectedShip.isVertical &&
                coordinates.x + this.selectedShip.length > 10
            )
                return;
            if (
                this.selectedShip.isVertical &&
                coordinates.y + this.selectedShip.length > 10
            )
                return;
            // check if there is already a ship in the selected area
            if (
                grid.some(
                    cell =>
                        !this.selectedShip.isVertical &&
                        cell.y === coordinates.y &&
                        cell.x >= coordinates.x &&
                        cell.x < coordinates.x + this.selectedShip.length &&
                        !cell.isEmpty
                )
            )
                return;

            if (
                grid.some(
                    cell =>
                        this.selectedShip.isVertical &&
                        cell.x === coordinates.x &&
                        cell.y >= coordinates.y &&
                        cell.y < coordinates.y + this.selectedShip.length &&
                        !cell.isEmpty
                )
            )
                return;

            if (!this.selectedShip.isVertical) {
                shipCells = grid.filter(
                    cell =>
                        cell.x >= coordinates.x &&
                        cell.x < coordinates.x + this.selectedShip.length &&
                        cell.y === coordinates.y
                );
            } else {
                shipCells = grid.filter(
                    cell =>
                        cell.y >= coordinates.y &&
                        cell.y < coordinates.y + this.selectedShip.length &&
                        cell.x === coordinates.x
                );
            }
            shipCells.forEach(cell => {
                cell.isEmpty = false;
                cell.shipId = this.selectedShip.id;
            });
            this.ships[this.selectedShip.id].human.isReady = true;
            this.selectedShip.isSelected = false;
        },
        removeShip() {
            this.grid.forEach(cell => (cell.isHovered = false));
        },
        moveShipOnGrid(coordinates) {
            if (!this.selectedShip) return;

            if (!this.selectedShip.isVertical) {
                if (coordinates.x + this.selectedShip.length > 10) return;
                // check if there is already a ship in the hover area
                if (
                    this.grid.some(
                        cell =>
                            cell.y === coordinates.y &&
                            cell.x >= coordinates.x &&
                            cell.x < coordinates.x + this.selectedShip.length &&
                            !cell.isEmpty
                    )
                )
                    return;

                this.grid
                    .filter(
                        cell =>
                            cell.y === coordinates.y &&
                            cell.x >= coordinates.x &&
                            cell.x < coordinates.x + this.selectedShip.length
                    )
                    .forEach(cell => (cell.isHovered = true));
            } else {
                if (coordinates.y + this.selectedShip.length > 10) return;
                // check if there is already a ship in the hover area
                if (
                    this.grid.some(
                        cell =>
                            cell.x === coordinates.x &&
                            cell.y >= coordinates.y &&
                            cell.y < coordinates.y + this.selectedShip.length &&
                            !cell.isEmpty
                    )
                )
                    return;

                this.grid
                    .filter(
                        cell =>
                            cell.x === coordinates.x &&
                            cell.y >= coordinates.y &&
                            cell.y < coordinates.y + this.selectedShip.length
                    )
                    .forEach(cell => (cell.isHovered = true));
            }
        },
        placeShipsRandom() {
            this.grid.forEach(cell => (cell.isEmpty = true));
            this.ships.forEach(ship => (ship.human.isReady = false));
            this.generateRandomCoordinates(this.grid);
        },
        generateRandomCoordinates(grid) {
            this.ships.forEach(ship => {
                if (ship.human.isReady) return;
                const isVertical = Math.round(Math.random());
                let randomX, randomY;
                if (!isVertical) {
                    randomX = Math.floor(Math.random() * (10 - ship.length));
                    randomY = Math.floor(Math.random() * 10);
                } else {
                    randomX = Math.floor(Math.random() * 10);
                    randomY = Math.floor(Math.random() * (10 - ship.length));
                }
                ship.isVertical = isVertical;
                ship.isSelected = true;
                this.addShipIdToGrid({ x: randomX, y: randomY }, grid);
                if (!ship.human.isReady) this.generateRandomCoordinates(grid);
            });
        },
        selectShip(ship) {
            if (this.isPlaying) return;
            if (!ship.human.isReady) ship.isSelected = true;
        },
        changeShipDirection() {
            if (this.selectedShip)
                this.selectedShip.isVertical = !this.selectedShip.isVertical;
        },
        startGame() {
            this.isPlaying = true
            this.createAIGrid();
            this.placeAIShips();
        },
        placeAIShips() {
            this.ships.forEach(ship => (ship.human.isReady = false));
            this.generateRandomCoordinates(this.AIGrid);
            this.ships.forEach(ship => (ship.human.isReady = false));
        },
        createGrid(grid) {
            let id = 1
            for (let y = 0; y < this.gridSize; y++) {
                for (let x = 0; x < this.gridSize; x++) {
                    grid.push({
                        id: id++,
                        x: x,
                        y: y,
                        hit: false,
                        isEmpty: true,
                        isHovered: false,
                        missed: false
                    });
                }
            }
        },
        createAIGrid() {
            this.createGrid(this.AIGrid);
        },
        resetGame() {
            this.grid = [],
            this.AIGrid = [],
            this.firstHit = '',
            this.ships.forEach(ship => {
                ship.human.isSinked = false
                ship.human.hitCounter = 0
                ship.human.isReady = false
                ship.computer.isSinked = false
                ship.computer.hitCounter = 0
            })
        }
    },
    created() {
        this.createGrid(this.grid);
    }
};
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Roboto:300,400,500,700');

* {
    padding: 0;
    margin: 0;
}

body, html {
    height: 100%;
}

body {
    font-family: 'Roboto', sans-serif;
    background-color: #eeeeee;
}

$cellWidth: 3rem;
$blue: #348fda;
$grey: #a7a7a7;
$red: #b90000;

.container {
    height: 100%;
    display: flex;
    align-items: center;
    overflow: hidden;
}

.side {
    display: flex;

    &--human {
        justify-self: start;
    }
}

.not-available {
    opacity: 0.5;
    background-color: $grey;
}
.button {
    cursor: pointer;
    border: none;
    padding: 1rem 1.5rem;
    background-color: $blue;
    border-radius: 3px;
    color: whitesmoke;
    font-weight: 700;
    transition: all .3s ease-in-out;
    outline: none;

    &:hover:not(&--disabled) {
        transform: translateY(-2px);
        background-color: darken($blue, 10%);
    }

    &--disabled {
        background-color: lighten($blue, 20%);
    }

    &:active {
        transform: translateY(1px);
    }

    &:not(:last-of-type) {
        margin-bottom: .8rem;
    }
}
.ships {
    display: flex;
    flex-direction: column;
    align-items: flex-start;

    &__list {
        height: 17rem;
    }
    &__size {
        display: flex;
        align-items: center;
        justify-content: center;
        margin: 0.2rem 0;
        cursor: pointer;
        background-color: $grey;
        height: $cellWidth;
        border-radius: $cellWidth;

        &:hover:not(.not-available) {
            background-color: darken($grey, 10%);
        }

        &--1 {
            width: $cellWidth;
        }
        &--2 {
            width: $cellWidth * 2;
        }
        &--3 {
            width: $cellWidth * 3;
        }
        &--4 {
            width: $cellWidth * 4;
        }
        &--5 {
            width: $cellWidth * 5;
        }
    }
}
.grid {
    width: 30rem;
    height: 30rem;
    display: grid;
    grid-template-columns: repeat(10, 1fr);
    grid-template-rows: repeat(10, 1fr);
    grid-gap: 3px;
    border-radius: 1%;
    

    &--human {
        border: 3px $blue solid; 
        background-color: $blue;

        & .grid__cell {
            cursor: default;
        }
    }

    &--computer {
        background-color: lightgray;
        border: 3px lightgray solid;

        & .grid__cell:hover:not(.grid__cell--hit):not(.grid__cell--missed) {
            background-color: $grey;
        }
    }

    &__cell {
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        background-color: whitesmoke;
        width: 95%;
        height: 95%;
        border-radius: 50%;
        transition: all .2s ease-in-out;

        &--hit {
            background-color: $red;
            cursor: not-allowed;
        }

        &--hovering {
            background-color: $grey;
        }

        &--missed {
            background-color: transparent;
            cursor: not-allowed;
        }

        &--placed {
            background-color: darken($grey, 20%);
        }
    }
}
.slide-enter-active {
    transition: all .4s ease-in-out;
}
.slide-enter {
    opacity: 0;
    transform: translateX(20vw);
}
</style>
