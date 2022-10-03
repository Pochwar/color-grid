<template>
  <div>
    <div
      id="grid"
      :style="gridStyle"
    >
      <div
        v-for="area in areas"
        :key="`area-${area.col}-${area.row}`"
        :style="areaStyle(area.col, area.row)"
        @click="seed(area.col, area.row, 255, 255, 255)"
      />
      <Cell
        v-for="cell in cells"
        :key="`cell-${cell.col}-${cell.row}`"
        :uuid="cell.uuid"
        :size="size"
        :seed-col="cell.col"
        :seed-row="cell.row"
        :seed-r="cell.r"
        :seed-g="cell.g"
        :seed-b="cell.b"
        :grid-cols="cols"
        :grid-rows="rows"
        :lifecycle="lifecycle"
        @duplicate="duplicate"
        @die="removeCell"
      />
    </div>
    <button @click="reset">
      Reset
    </button>
  </div>

</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Cell from '@/components/Cell.vue';
import { v4 as uuidv4 } from 'uuid';

interface CellInterface {
  uuid: string,
  col: number,
  row: number,
  r: number,
  g: number,
  b: number,
}

interface AreaInterface {
  col: number,
  row: number,
}

@Component({
  components: {
    Cell,
  },
})
export default class Grid extends Vue {
  private cols = 80;

  private rows = 60;

  private size = 10;

  private cells: Array<CellInterface> = [];

  private lifecycle = 0;

  get areas(): Array<AreaInterface> {
    const areas: Array<AreaInterface> = [];
    for (let i = 1; i < this.cols; i += 1) {
      for (let j = 1; j < this.rows; j += 1) {
        areas.push({
          col: i,
          row: j,
        });
      }
    }

    return areas;
  }

  get gridStyle() {
    return {
      display: 'grid',
      'grid-auto-columns': `${this.size}px`,
      'grid-auto-rows': `${this.size}px`,
    };
  }

  private mounted(): void {
    setInterval(() => {
      this.lifecycle += 1;
    }, 1000);
  }

  private reset(): void {
    this.cells = [];
    this.lifecycle = 0;
  }

  private areaStyle(col: number, row: number) {
    return {
      'z-index': 40,
      position: 'relative',
      'background-color': 'silver',
      width: `${this.size}px`,
      height: `${this.size}px`,
      'grid-area': `${row} / ${col}`,
    };
  }

  private seed(col: number, row: number, r = 0, g = 0, b = 0): void {
    this.cells.push({
      uuid: uuidv4(),
      col,
      row,
      r,
      g,
      b,
    });
  }

  private duplicate(uuid: string): void {
    const parent = this.cells.find((cell) => cell.uuid === uuid);
    if (parent) {
      const nextAreas = this.findNextAreas(parent);
      nextAreas.forEach((area) => {
        if (!this.isAreaOccupied(area)) {
          if (Grid.random(0, 1) === 1) {
            this.seed(area.col, area.row, parent.r, parent.g, parent.b);
          }
        }
      });
    }
  }

  private isAreaOccupied(area: AreaInterface): boolean {
    return this.cells
      .filter((cell) => cell.col === area.col && cell.row === area.row)
      .length > 0;
  }

  private findNextAreas(cell: CellInterface): Array<AreaInterface> {
    return this.areas.filter((area) => {
      const isAreaLeft = (area.row === cell.row && area.col === cell.col - 1);
      const isAreaRight = (area.row === cell.row && area.col === cell.col + 1);
      const isAreaUp = (area.row === cell.row - 1 && area.col === cell.col);
      const isAreaDown = (area.row === cell.row + 1 && area.col === cell.col);

      return isAreaLeft || isAreaRight || isAreaUp || isAreaDown;
    });
  }

  private removeCell(uuid: string) {
    this.cells = this.cells.filter((cell) => cell.uuid !== uuid);
  }

  private static random(min: number, max: number) {
    return Math.floor(Math.random() * (max - min + 1) + min);
  }
}
</script>
