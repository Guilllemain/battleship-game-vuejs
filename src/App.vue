<template>
  <div>
    <div class="ships">
      <div
        v-for="ship in availableShips"
        @click="selectShip(ship)"
        :key="ship.id"
        class="ships__size"
        :class="`ships__size--${ship.length}`"
      >{{ ship.name }}</div>
    </div>
    <button @click="changeShipDirection">Change direction</button>
    <div class="grid">
      <template v-for="y in grid" style="display:flex;">
        <!-- eslint-disable-next-line -->
        <div
          v-for="x in y.rows"
          @mouseover="showingShip"
          @mouseout="removeShip"
          @click="test({x: x.row, y: y.column})"
          :data-column="`${y.column}`"
          :data-row="`${x.row}`"
          class="grid__cell"
          :class="{ 'grid__cell--hit': x.hit, 'grid__cell--placed': x.shipId}"
        ></div>
      </template>
    </div>
  </div>
</template>

<script>
import shipsList from './assets/js/ships'
export default {
  data() {
    return {
      isPlaying: false,
      gridSize: 10,
      ships: shipsList,
      selectedShip: {},
      grid: [],
    };
  },
  computed: {
      availableShips() {
          return this.ships.filter(ship => !ship.ready)
      }
  },
  methods: {
    test(coordinates) {
      if (this.isPlaying) {
        this.grid[coordinates.y].rows[coordinates.x].hit = true;
      } else {
          for (let i = 0; i < this.selectedShip.length; i++) {
            if (!this.selectedShip.isVertical) {
                this.grid[coordinates.y].rows[coordinates.x + i].shipId = this.selectedShip.id;
            } else {
                console.log(this.grid[coordinates.y + i].rows[coordinates.x])
                this.grid[coordinates.y + i].rows[coordinates.x].shipId = this.selectedShip.id;
            }
        }
        this.ships.forEach(ship =>
          ship.id === this.selectedShip.id ? (ship.ready = true) : ""
        );
        this.selectedShip = {};
      }
    },
    showingShip() {
      const cell_list = [...document.querySelectorAll(".grid__cell")];
      let elements = [];

      if (!this.selectedShip.isVertical) {
        const selectedCell = Number(event.target.dataset.row);
        for (let y = 0; y < cell_list.length; y++) {
          for (let i = 0; i < this.selectedShip.length; i++) {
            if (
              cell_list[y].dataset.row == selectedCell + i &&
              cell_list[y].dataset.column === event.target.dataset.column &&
              selectedCell + this.selectedShip.length < 11
            ) {
                elements.push(cell_list[y])
            }
          }
        }
      } else {
        const selectedCell = Number(event.target.dataset.column);
        for (let y = 0; y < cell_list.length; y++) {
          for (let i = 0; i < this.selectedShip.length; i++) {
            if (
              cell_list[y].dataset.column == selectedCell + i &&
              cell_list[y].dataset.row === event.target.dataset.row &&
              selectedCell + this.selectedShip.length < 11
            ) {
                elements.push(cell_list[y])
            }
          }
        }
      }
      if (!elements.some(el => el.className.includes('grid__cell--placed'))) {
          elements.forEach(el => el.classList.add("grid__cell--ship"))
      }
    },
    removeShip() {
      document
        .querySelectorAll(".grid__cell")
        .forEach(grid => grid.classList.remove("grid__cell--ship"));
    },
    selectShip(ship) {
      if (this.isPlaying) return;
      if (!ship.ready) this.selectedShip = ship;
    },
    changeShipDirection() {
      if (!this.selectedShip) return;
      this.selectedShip.isVertical = !this.selectedShip.isVertical;
    }
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

    &--placed {
        background-color: green;
    }

    &:hover {
      // background-color: grey;
    }
  }
}
</style>
