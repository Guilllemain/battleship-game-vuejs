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
                    :class="[`ships__size--${ship.length}`, {'not-available': ship.isReady }]"
                >{{ ship.name }}
                </div>
                <button @click="changeShipDirection">Change ship direction</button>
                <button :disabled="!isPlaying" @click="startGame">Start Game</button>
            </div>
        </div>
        <div class="grid" style="margin-right:10rem;">
            <template v-for="y in grid" style="display:flex;">
                <!-- eslint-disable-next-line -->
                <div
                v-for="x in y.rows"
                @mouseover="moveShipOnGrid"
                @mouseout="removeShip"
                @click="addShipIdToGrid({x: x.row, y: y.column})"
                :data-column="`${y.column}`"
                :data-row="`${x.row}`"
                class="grid__cell"
                :class="{ 'grid__cell--hit': x.hit, 'grid__cell--placed': !x.isEmpty, 'grid__cell--ship': x.isHovered, 'grid__cell--missed': x.missed}"
                ></div>
            </template>
        </div>
        <div v-if="isPlaying && AIGrid.length > 0" style="display:flex">
            <div class="grid">
                <template v-for="y in AIGrid" style="display:flex;">
                    <!-- eslint-disable-next-line -->
                    <div
                    v-for="x in y.rows"
                    @click="attackShip({x: x.row, y: y.column})"
                    class="grid__cell"
                    :class="{ 'grid__cell--hit': x.hit, 'grid__cell--missed': x.missed}"
                    ></div>
                </template>
            </div>
            <div class="ships">
                <div class="ships__list">
                    <div
                        v-for="ship in ships"
                        @click="selectShip(ship)"
                        :key="ship.id"
                        class="ships__size"
                        :class="[`ships__size--${ship.length}`, {'not-available': ship.isSinked }]"
                    >{{ ship.name }}
                    </div>
                </div>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
import shipsList from './assets/js/ships'
export default {
  data() {
    return {
      gridSize: 10,
      ships: shipsList,
      selectedShip: {},
      lastAIAttack: '',
      grid: [],
      AIGrid: []
    };
  },
  computed: {
      isPlaying() {
          return this.ships.every(ship => ship.isReady)
      }
  },
  methods: {
    attackShip(coordinates) {
        const cell = this.AIGrid[coordinates.y].rows[coordinates.x]
        const ship = this.ships[cell.shipId]
        if (cell.isEmpty) {
            cell.missed = true
        } else {
            cell.isEmpty = true
            cell.hit = true
            ship.hitCounterHuman += 1
            if (ship.length === ship.hitCounterHuman) ship.isSinkedByHuman = true // check if the ship is sinked
            if (this.ships.every(ship => ship.isSinkedByHuman)) alert('You won')
        }
        this.AIAttack()
    },
    AIAttack() {
        const randomX = Math.floor(Math.random() * 10)
        const randomY = Math.floor(Math.random() * 10)
        const cell = this.grid[randomY].rows[randomX]
        const ship = this.ships[cell.shipId]
        if (cell.isEmpty) {
            cell.missed = true
        } else {
            cell.isEmpty = true
            cell.hit = true
            ship.hitCounterAI += 1
            if (ship.length === ship.hitCounterAI) {
                    ship.isSinkedByAI = true
                    this.lastAIAttack = {randomX, randomY}
                } // check if the ship is sinked
            if (this.ships.every(ship => ship.isSinkedByAI)) alert('You lose')
        }
    },
    addShipIdToGrid(coordinates) {
        // check if there is already a ship in the selected area
        for (let i = 0; i < this.selectedShip.length; i++) {
            if (!this.selectedShip.isVertical && !this.grid[coordinates.y].rows[coordinates.x + i].isEmpty ||
                this.selectedShip.isVertical && !this.grid[coordinates.y + i].rows[coordinates.x].isEmpty) return
        }

        for (let i = 0; i < this.selectedShip.length; i++) {
            if (!this.selectedShip.isVertical) {
                this.grid[coordinates.y].rows[coordinates.x + i].isEmpty = false
                this.grid[coordinates.y].rows[coordinates.x + i].shipId = this.selectedShip.id
            } else {
                this.grid[coordinates.y + i].rows[coordinates.x].isEmpty = false
                this.grid[coordinates.y + i].rows[coordinates.x].shipId = this.selectedShip.id
            }
        }
        this.ships.forEach(ship =>
          ship.id === this.selectedShip.id ? (ship.isReady = true) : ""
        );
        this.selectedShip = {};
    },
    removeShip() {
        this.grid.forEach(column => column.rows.forEach(row => row.isHovered = false))
    },
    moveShipOnGrid() {
        const x = Number(event.target.dataset.row)
        const y = Number(event.target.dataset.column)
        let selectedCell = Number(event.target.dataset.row);

        if (selectedCell + this.selectedShip.length < 11) {
            if (!this.selectedShip.isVertical) {
                // check if there is already a ship in the hover area
                for (let i = 0; i < this.selectedShip.length; i++) {
                    if (!this.grid[event.target.dataset.column].rows[x + i].isEmpty) return
                }
                for (let i = 0; i < this.selectedShip.length; i++) {
                    if (this.grid[event.target.dataset.column].rows[x + i]) {
                        this.grid[event.target.dataset.column].rows[x + i].isHovered = true
                    }
                }
            } else {
                selectedCell = Number(event.target.dataset.column);
                // check if there is already a ship in the hover area
                for (let i = 0; i < this.selectedShip.length; i++) {
                    if (!this.grid[y + i].rows[event.target.dataset.row].isEmpty) return
                }
                for (let i = 0; i < this.selectedShip.length; i++) {
                    if (this.grid[y + i] && selectedCell + this.selectedShip.length < 11) {
                        this.grid[y + i].rows[event.target.dataset.row].isHovered = true
                    }
                }
            }
        }
    },
    selectShip(ship) {
      if (this.isPlaying) return;
      if (!ship.ready) this.selectedShip = ship;
    },
    changeShipDirection() {
      if (!this.selectedShip) return;
      this.selectedShip.isVertical = !this.selectedShip.isVertical;
    },
    startGame() {
        this.createAIGrid()
        this.placeAIShips()
    },
    placeAIShips() {
        this.ships.forEach(ship => {
            if (!ship.isAiPlaced) {
                const randomX = Math.floor(Math.random() * 10)
                const randomY = Math.floor(Math.random() * 10)
                const isVertical = Math.round(Math.random())

                if (!isVertical) {
                    // check if the ship exceeds the grid
                    if (ship.length + randomX > 10) return this.placeAIShips()
                    // check if the area is free of ships
                    for (let i = 0; i < ship.length; i++) {
                        if (!this.AIGrid[randomY].rows[randomX + i].isEmpty) return this.placeAIShips()
                    }
                    for (let i = 0; i < ship.length; i++) {
                        if (this.AIGrid[randomY].rows[randomX + i]) {
                            this.AIGrid[randomY].rows[randomX + i].isEmpty = false
                            this.AIGrid[randomY].rows[randomX + i].shipId = ship.id
                        }
                    }
                } else {
                    // check if the ship exceeds the grid
                    if (ship.length + randomY > 10) return this.placeAIShips()
                    // check if the area is free of ships
                    for (let i = 0; i < ship.length; i++) {
                        if (!this.AIGrid[randomY + i].rows[randomX].isEmpty) return this.placeAIShips()
                    }
                    for (let i = 0; i < ship.length; i++) {
                        if (this.AIGrid[randomY + i].rows[randomX]) {
                            this.AIGrid[randomY + i].rows[randomX].isEmpty = false
                            this.AIGrid[randomY + i].rows[randomX].shipId = ship.id
                        }
                    }
                }
                ship.isAiPlaced = true
            }
        })
    },
    createGrid(grid) {
        for (let y = 0; y < this.gridSize; y++) {
          const col = { column: y, rows: [] }
          for (let x = 0; x < this.gridSize; x++) {
            col.rows.push({ row: x, hit: false, isEmpty: true, isHovered: false, missed: false})
          }
          grid.push(col);
        }
    },
    createAIGrid() {
        this.createGrid(this.AIGrid)
    }
  },
  created() {
    this.createGrid(this.grid)
  }
};
</script>

<style lang="scss">
.not-available {
    opacity: .5;
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
