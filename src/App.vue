<template>
    <div>
        <div style="display:flex;">
            <div class="ships">
                <div class="ships__list">
                    <div
                        v-for="ship in ships"
                        @click="selectShip(ship)"
                        :key="ship.id"
                        class="ships__size"
                        :class="[`ships__size--${ship.length}`, {'not-available': ship.human.isReady }]"
                    >{{ ship.name }}</div>
                    <button @click="changeShipDirection">Change ship direction</button>
                    <button :disabled="!isPlaying" @click="startGame">Start Game</button>
                    <button @click="placeShipsRandom">Place ships randomly</button>
                </div>
            </div>
            <div class="grid" style="margin-right:10rem">
                <!-- eslint-disable-next-line -->
                <div
                    v-for="cell in grid"
                    @mouseover="moveShipOnGrid"
                    @mouseout="removeShip"
                    @click="addShipIdToGrid({x: cell.x, y: cell.y}, grid)"
                    :data-column="`${cell.y}`"
                    :data-row="`${cell.x}`"
                    class="grid__cell"
                    :class="{ 'grid__cell--hit': cell.hit, 'grid__cell--placed': !cell.isEmpty && !cell.hit, 'grid__cell--ship': cell.isHovered, 'grid__cell--missed': cell.missed}"
                ></div>
            </div>
            <div v-if="isPlaying && AIGrid.length > 0" style="display:flex">
                <div class="grid">
                    <!-- eslint-disable-next-line -->
                    <div
                        v-for="cell in AIGrid"
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
                            :class="[`ships__size--${ship.length}`, {'not-available': ship.isSinkedByHuman }]"
                        >{{ ship.name }}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import shipsList from "./assets/js/ships";
export default {
    data() {
        return {
            gridSize: 10,
            ships: shipsList,
            grid: [],
            AIGrid: []
        };
    },
    computed: {
        isPlaying() {
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
            if (cell.isEmpty) {
                cell.missed = true;
            } else {
                cell.hit = true;
                ship.human.hitCounter += 1;
                // check if the ship is sinked
                if (ship.length === ship.human.hitCounterHuman)
                    ship.human.isSinked = true;
                // check all ships are sinked
                if (this.ships.every(ship => ship.human.isSinked))
                    alert("You won");
            }
            this.AIAttack();
        },
        AIAttack() {
            let randomX = Math.floor(Math.random() * 10);
            let randomY = Math.floor(Math.random() * 10);

            const cell = this.grid.find(
                cell => cell.x === randomX && cell.y === randomY
            );
            if (cell.missed || cell.hit) return this.AIAttack();
            if (cell.isEmpty) return (cell.missed = true);

            const ship = this.ships[cell.shipId];
            cell.isEmpty = true;
            cell.hit = true;
            ship.computer.hitCounter += 1;
            // check if the ship is sinked and if not record the coordinates and the ship ID for the next attack
            if (ship.length === ship.computer.hitCounter) {
                ship.computer.isSinked = true;
            }
            if (this.ships.every(ship => ship.computer.isSinked))
                alert("You lose");
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
        moveShipOnGrid() {
            if (!this.selectedShip) return;
            const mouseX = Number(event.target.dataset.row);
            const mouseY = Number(event.target.dataset.column);

            if (!this.selectedShip.isVertical) {
                if (mouseX + this.selectedShip.length > 10) return;
                // check if there is already a ship in the hover area
                if (
                    this.grid.some(
                        cell =>
                            cell.y === mouseY &&
                            cell.x >= mouseX &&
                            cell.x < mouseX + this.selectedShip.length &&
                            !cell.isEmpty
                    )
                )
                    return;

                this.grid
                    .filter(
                        cell =>
                            cell.y === mouseY &&
                            cell.x >= mouseX &&
                            cell.x < mouseX + this.selectedShip.length
                    )
                    .forEach(cell => (cell.isHovered = true));
            } else {
                if (mouseY + this.selectedShip.length > 10) return;
                // check if there is already a ship in the hover area
                if (
                    this.grid.some(
                        cell =>
                            cell.x === mouseX &&
                            cell.y >= mouseY &&
                            cell.y < mouseY + this.selectedShip.length &&
                            !cell.isEmpty
                    )
                )
                    return;

                this.grid
                    .filter(
                        cell =>
                            cell.x === mouseX &&
                            cell.y >= mouseY &&
                            cell.y < mouseY + this.selectedShip.length
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
            this.createAIGrid();
            this.placeAIShips();
        },
        placeAIShips() {
            this.ships.forEach(ship => (ship.human.isReady = false));
            this.generateRandomCoordinates(this.AIGrid);
        },
        createGrid(grid) {
            for (let y = 0; y < this.gridSize; y++) {
                for (let x = 0; x < this.gridSize; x++) {
                    grid.push({
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
        }
    },
    created() {
        this.createGrid(this.grid);
    }
};
</script>

<style lang="scss">
.not-available {
    opacity: 0.5;
    background-color: grey;
}
.ships {
    width: 20%;
    &__list {
        height: 17rem;
    }
    &__size {
        display: flex;
        align-items: center;
        justify-content: center;
        margin: 0.2rem 0;
        background-color: coral;
        height: 3rem;
        &--1 {
            width: 3rem;
        }
        &--2 {
            width: 6rem;
        }
        &--3 {
            width: 9rem;
        }
        &--4 {
            width: 12rem;
        }
        &--5 {
            width: 15rem;
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
    background-color: blue;
    border: 3px blue solid;

    &__cell {
        cursor: pointer;
        background-color: whitesmoke;

        &--hit {
            background-color: black;
        }

        &--ship {
            background-color: greenyellow;
        }

        &--missed {
            background-color: orange;
        }

        &--placed {
            background-color: green;
        }

        &:hover {
            // background-color: grey;
        }
    }
}
</style>
