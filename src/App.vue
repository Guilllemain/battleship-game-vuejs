<template>
    <div>
        <div class="ships">
            <div
                v-for="ship in ships"
                @click="selectShip(ship)"
                :key="ship.id"
                class="ships__size"
                :class="`ships__size--${ship.length}`"
            >{{ ship.name }}</div>
        </div>
        <button @click="changeShipDirection">Change direction</button>
        <div class="card" ref="card">
            <template v-for="y in grid" style="display:flex;">
                <!-- eslint-disable-next-line -->
                <div
                    v-for="x in y.rows"
                    @mouseover="showingShip"
                    @mouseout="removeShip"
                    @click="test({x: x.row, y: y.column})"
                    :data-column="`${y.column}`"
                    :data-row="`${x.row}`"
                    class="card__tile"
                    :class="{ 'card__tile--hit': x.hit, 'card__tile--ship': x.shipId}"
                ></div>
            </template>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            isPlaying: false,
            gridSize: 10,
            ships: [
                {
                    id: 1,
                    name: "Destroyer",
                    vertical: false,
                    length: 2,
                    ready: false
                },
                {
                    id: 2,
                    name: "Submarine",
                    vertical: false,
                    length: 3,
                    ready: false
                },
                {
                    id: 3,
                    name: "Cruiser",
                    vertical: false,
                    length: 3,
                    ready: false
                },
                {
                    id: 4,
                    name: "Battleship",
                    vertical: false,
                    length: 4,
                    ready: false
                },
                {
                    id: 5,
                    name: "Carrier",
                    isVertical: false,
                    length: 5,
                    ready: false
                }
            ],
            selectedShip: {},
            grid: [],
            dragging: {
                isDragging: false,
                selectedElement: '',
                initialMouseX: '',
                initialMouseY: '',
                startX : '',
                startY: ''
            }
        };
    },
    methods: {
        test(coordinates) {
            if (this.isPlaying) {
                this.grid[coordinates.y].rows[coordinates.x].hit = true;
            } else {
                for (let i = 0; i < this.selectedShip.length; i++) {
                    if (!this.selectedShip.vertical) {
                        this.grid[coordinates.y].rows[
                            coordinates.x + i
                        ].shipId = this.selectedShip.id;
                    } else {
                        this.grid[coordinates.y + i].rows[
                            coordinates.x
                        ].shipId = this.selectedShip.id;
                    }
                }
                this.ships.forEach(ship =>
                    ship.id === this.selectedShip.id ? (ship.ready = true) : ""
                );
                this.selectedShip = {};
            }
        },
        showingShip() {
            if (!this.selectedShip.isVertical) {
                const selectedCell = Number(event.target.dataset.column)
                const cell_list = [...document.querySelectorAll('.card__tile')]

                for (let y = 0; y < cell_list.length; y++) {
                        for(let i = selectedCell; i < this.selectedShip.length; i++) {
                            
                            cell_list[i + selectedCell].classList.add('card__tile--ship')
                        }
                }
                
                
            }
        },
        removeShip() {
            event.target.classList.remove('card__tile--ship')
        },
        selectShip(ship) {
            if (this.isPlaying) return;
            this.selectedShip = ship;
        },
        changeShipDirection() {
            if (!this.selectedShip) return;
            this.selectedShip.isVertical = !this.selectedShip.isVertical;
        },
        // startDrag(event) {
        //     this.dragging.selectedElement = event.target;
        //     this.dragging.selectedElement.style.position = "absolute";
        //     this.dragging.startX = event.target.offsetLeft;
        //     this.dragging.startY = event.target.offsetTop;
        //     this.dragging.initialMouseX = event.clientX
        //     this.dragging.initialMouseY = event.clientY
        //     this.dragging.isDragging = true;
        // },
        // stopDrag() {
        //     this.dragging.isDragging = false;
        // },
        // doDrag(event) {
        //     if (this.dragging.isDragging) {
        //         this.dragging.selectedElement.style.top = this.dragging.startY + (event.clientY - this.dragging.initialMouseY) + 'px';
        //         this.dragging.selectedElement.style.left = this.dragging.startX + (event.clientX - this.dragging.initialMouseX) + 'px';
        //         return false
        //     }
        // }
    },
    created() {
        for (let y = 0; y < this.gridSize; y++) {
            const col = { column: y, rows: [] };
            for (let x = 0; x < this.gridSize; x++) {
                col.rows.push({ row: x, hit: false, shipId: null });
            }
            this.grid.push(col);
        }
    }
};
</script>

<style lang="scss">
.ships {
    height: 17rem;
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
.card {
    width: 30rem;
    height: 30rem;
    display: grid;
    grid-template-columns: repeat(10, 1fr);
    grid-template-rows: repeat(10, 1fr);
    grid-gap: 3px;
    background-color: blue;
    border: 3px blue solid;

    &__tile {
        cursor: pointer;
        background-color: whitesmoke;

        &--hit {
            background-color: black;
        }

        &--ship {
            background-color: greenyellow;
        }

        &:hover {
            // background-color: grey;
        }
    }
}
</style>
