<template>
    <div>
        <div class="title">
            <h1 class="title__main">Battleship</h1>
            <h3 class="title__small">Place your ships and battle against the computer</h3>
        </div>
        <div class="container">
            <div class="side side--human">
                <div class="aside">
                    <div class="ships ships--human">
                        <div
                            v-for="ship in ships"
                            @click="selectShip(ship)"
                            :key="ship.id"
                            class="ships__unit"
                            :class="[`ships__unit--${ship.length}`, {'ships__unit--unavailable': ship.human.isReady }, { 'ships__unit--sinked': ship.computer.isSinked}, { 'ships__unit--not-hoverable': isPlaying}]"
                        >{{ ship.name }}</div>
                    </div>
                    <div class="cta" v-if="!isPlaying">
                        <button
                            class="cta__button"
                            :disabled="!selectedShip"
                            :class="{ 'cta__button--disabled': !selectedShip }"
                            @click="changeShipDirection"
                        >Change ship direction</button>
                        <button
                            class="cta__button"
                            :disabled="!isPlayerReady"
                            :class="{ 'cta__button--disabled': !isPlayerReady }"
                            @click="startGame"
                        >Start game</button>
                        <button class="cta__button" @click="placeShipsRandom">Place ships randomly</button>
                    </div>
                </div>
                <div class="grid grid--human">
                    <div
                        v-for="cell in grid"
                        :key="cell.id"
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
                            v-for="cell in AIGrid"
                            :key="cell.id"
                            @click="attackShip({x: cell.x, y: cell.y})"
                            class="grid__cell"
                            :class="{ 'grid__cell--hit': cell.hit, 'grid__cell--missed': cell.missed }"
                        ></div>
                    </div>
                    <div class="aside">
                        <div class="ships ships--computer">
                            <div
                                v-for="ship in ships"
                                @click="selectShip(ship)"
                                :key="ship.id"
                                class="ships__unit"
                                :class="[`ships__unit--${ship.length}`, {'ships__unit--sinked': ship.human.isSinked }, { 'ships__unit--not-hoverable': isPlaying}]"
                            >{{ ship.name }}</div>
                        </div>
                    </div>
                </div>
            </transition>
        </div>
        <modal-component @closeModal="isModalOpen = false" v-show="isModalOpen">
            <div class="popup">
                <h3 class="popup__title">{{ popup.title }}</h3>
                <p class="popup__text">{{ popup.text }}</p>
                <p class="popup__text">Feel like trying again ?</p>
                <button @click="newGame" class="popup__cta">New Game</button>
            </div>
        </modal-component>
    </div>
</template>

<script>
import ModalComponent from "./components/ModalComponent";
import shipsList from "./assets/js/ships";
export default {
    components: { ModalComponent },
    data() {
        return {
            isModalOpen: false,
            popup: {},
            isPlaying: false,
            gridSize: 10,
            ships: shipsList,
            firstHit: "",
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
                return;
            } else {
                cell.hit = true;
                ship.human.hitCounter += 1;
                // check if the ship is sinked
                if (ship.length === ship.human.hitCounter)
                    ship.human.isSinked = true;
                // check all ships are sinked
                if (this.ships.every(ship => ship.human.isSinked)) {
                    this.popup.title = "congratulations!";
                    this.popup.text = "you won";
                    return (this.isModalOpen = true);
                }
            }
            this.AIAttack();
        },
        AIAttack(skip = false) {
            let randomX = Math.floor(Math.random() * 10);
            let randomY = Math.floor(Math.random() * 10);

            let cell = this.grid.find(
                cell => cell.x === randomX && cell.y === randomY
            );

            if (!this.firstHit && !skip) {
                // if all cells are surrounded by at least one missed cell, relauch this method but skip this part
                if (
                    this.grid
                        .filter(
                            target =>
                                (target.y === randomY &&
                                    (target.x === randomX + 1 ||
                                        target.x === randomX - 1)) ||
                                (target.x === randomX &&
                                    (target.y === randomY + 1 ||
                                        target.y === randomY - 1))
                        )
                        .every(target => target.missed)
                )
                    return this.AIAttack(true);
                // avoid hitting a cell next to a missed one
                if (
                    this.grid
                        .filter(
                            target =>
                                (target.y === randomY &&
                                    (target.x === randomX + 1 ||
                                        target.x === randomX - 1)) ||
                                (target.x === randomX &&
                                    (target.y === randomY + 1 ||
                                        target.y === randomY - 1))
                        )
                        .some(target => target.missed)
                )
                    return this.AIAttack();
            }

            if (this.firstHit && this.grid.some(cell => cell.prob === 1)) {
                const cells = this.grid.filter(cell => cell.prob === 1);
                const randomCell = Math.floor(Math.random() * cells.length);
                cell = cells[randomCell];
                randomX = cell.x;
                randomY = cell.y;
            }

            if (cell.isEmpty && !cell.missed) {
                cell.prob = 0;
                return (cell.missed = true);
            } else if (cell.missed || cell.hit) {
                return this.AIAttack();
            }

            const ship = this.ships[cell.shipId];
            cell.hit = true;
            ship.computer.hitCounter += 1;

            cell.prob = 0;
            this.grid
                .filter(
                    target =>
                        (target.y === randomY &&
                            (target.x === randomX + 1 ||
                                target.x === randomX - 1)) ||
                        (target.x === randomX &&
                            (target.y === randomY + 1 ||
                                target.y === randomY - 1))
                )
                .forEach(target => {
                    if (!target.hit && !target.missed) {
                        target.prob = 1;
                    }
                });
            if (this.firstHit && this.firstHit.x === cell.x) {
                this.grid
                    .filter(target => target.prob === 1 && target.x !== cell.x)
                    .forEach(target => (target.prob = 0));
            } else if (this.firstHit && this.firstHit.y === cell.y) {
                this.grid
                    .filter(target => target.prob === 1 && target.y !== cell.y)
                    .forEach(target => (target.prob = 0));
            }
            if (!this.firstHit) {
                this.firstHit = cell;
            }
            // check if the ship is sinked and if not record the coordinates and the ship ID for the next attack
            if (ship.length === ship.computer.hitCounter) {
                ship.computer.isSinked = true;
                this.firstHit = "";
                this.grid.forEach(cell => (cell.prob = 0));
            }

            if (this.ships.every(ship => ship.computer.isSinked)) {
                this.popup.title = "oops!";
                this.popup.text = "you lost";
                return (this.isModalOpen = true);
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
            this.isPlaying = true;
            this.createAIGrid();
            this.placeAIShips();
        },
        placeAIShips() {
            this.ships.forEach(ship => (ship.human.isReady = false));
            this.generateRandomCoordinates(this.AIGrid);
            this.ships.forEach(ship => (ship.human.isReady = false));
        },
        createGrid(grid = this.grid) {
            let id = 1;
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
        newGame() {
            this.isModalOpen = false;
            this.resetGame();
            this.createGrid();
        },
        resetGame() {
            (this.grid = []),
                (this.AIGrid = []),
                (this.firstHit = ""),
                this.ships.forEach(ship => {
                    ship.human.isSinked = false;
                    ship.human.hitCounter = 0;
                    ship.human.isReady = false;
                    ship.computer.isSinked = false;
                    ship.computer.hitCounter = 0;
                });
        }
    },
    created() {
        this.createGrid();
    }
};
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css?family=Roboto:300,400,500,700|Audiowide");
@import url("https://fonts.googleapis.com/css?family=Audiowide|Bungee+Inline");

* {
    padding: 0;
    margin: 0;
}

body,
html {
    height: 100%;
}

body {
    position: relative;
    font-family: "Roboto", sans-serif;
    box-sizing: border-box;
    background-color: #333333;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='100%25' height='100%25' viewBox='0 0 800 400'%3E%3Cdefs%3E%3CradialGradient id='a' cx='396' cy='281' r='514' gradientUnits='userSpaceOnUse'%3E%3Cstop offset='0' stop-color='%232685ff'/%3E%3Cstop offset='1' stop-color='%23333333'/%3E%3C/radialGradient%3E%3ClinearGradient id='b' gradientUnits='userSpaceOnUse' x1='400' y1='148' x2='400' y2='333'%3E%3Cstop offset='0' stop-color='%23fdfff2' stop-opacity='0'/%3E%3Cstop offset='1' stop-color='%23fdfff2' stop-opacity='0.5'/%3E%3C/linearGradient%3E%3C/defs%3E%3Crect fill='url(%23a)' width='800' height='400'/%3E%3Cg fill-opacity='0.4'%3E%3Ccircle fill='url(%23b)' cx='267.5' cy='61' r='300'/%3E%3Ccircle fill='url(%23b)' cx='532.5' cy='61' r='300'/%3E%3Ccircle fill='url(%23b)' cx='400' cy='30' r='300'/%3E%3C/g%3E%3C/svg%3E");
    background-attachment: fixed;
    background-size: cover;
}

$cellWidth: 3vw;
$blue: #348fda;
$grey: #a7a7a7;
$red: #b90000;
$red-button: #b74242;

.container {
    height: 100%;
    display: flex;
    align-items: center;
}

.title {
    text-align: center;
    text-transform: uppercase;
    padding: 3vw 0;

    &__main {
        font-family: "Audiowide", cursive;
        font-size: 2.5rem;
        margin-bottom: 0.8rem;

        &::after {
            content: "";
            display: block;
            background-color: whitesmoke;
            height: 1px;
            width: 30%;
            margin: auto;
            opacity: 0.4;
            margin-top: 0.2rem;
        }
    }
    &__small {
        font-weight: 300;
    }
}

.side {
    display: flex;
    justify-content: center;
    flex-basis: 50vw;
}

.cta {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;

    &__button {
        font-weight: 500;
        cursor: pointer;
        border: none;
        padding: 1rem 1.5rem;
        background-color: $red-button;
        border-radius: 3px;
        color: whitesmoke;
        transition: all 0.3s ease-in-out;
        outline: none;
        font-size: 0.9rem;
        opacity: 0.95;

        &:hover:not(&--disabled) {
            transform: translateY(-2px);
            background-color: darken($red-button, 10%);
        }

        &--disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        &:active {
            transform: translateY(1px);
        }

        &:not(:last-of-type) {
            margin-bottom: 0.8rem;
        }
    }
}
.aside {
    display: flex;
    flex-direction: column;
    color: whitesmoke;
}

.ships {
    display: flex;
    flex-direction: column;
    font-size: 0.9rem;

    &--human {
        align-items: flex-end;
        margin-bottom: 1vw;
        padding-right: 1vw;
    }

    &--computer {
        align-items: flex-start;
        padding-left: 1vw;
    }
    &__unit {
        position: relative;
        display: flex;
        align-items: center;
        justify-content: center;
        margin: 0.2rem 0;
        background-color: $grey;
        height: $cellWidth;
        border-radius: $cellWidth;

        &--unavailable {
            opacity: 0.5;
        }

        &--sinked {
            background-color: $red;
            text-decoration: line-through;
        }

        &:hover:not(&--unavailable):not(&--not-hoverable) {
            cursor: pointer;
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
    width: $cellWidth * 10;
    height: $cellWidth * 10;
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
        background-color: $grey;
        border: 3px $grey solid;

        & .grid__cell:hover:not(.grid__cell--hit):not(.grid__cell--missed) {
            background-color: darken($grey, 20%);
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
        transition: all 0.2s ease-in-out;

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

.popup {
    background-color: whitesmoke;
    padding: 1.5rem 2rem;
    border-radius: 0.5rem;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    color: darken($grey, 30%);

    &__title {
        text-transform: uppercase;
        margin-bottom: 1rem;
        font-size: 1.5rem;
        font-weight: 300;
    }

    &__cta {
        margin-top: 1.3rem;
        border: none;
        background-color: $blue;
        color: whitesmoke;
        padding: 0.8rem 1.2rem;
        border-radius: 0.3rem;
        text-transform: uppercase;
        font-size: 1rem;
        cursor: pointer;
        transition: all 0.2s ease-in-out;
        outline: none;

        &:hover {
            background-color: darken($blue, 10%);
            transform: translateY(-2px);
        }
    }
}

.slide-enter-active {
    transition: all 0.4s ease-in-out;
}
.slide-enter {
    opacity: 0;
    transform: translateX(20vw);
}
</style>
