<template>
  <div :style="cellStyle"/>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

const FULL_GAUGE_MAX = 256;

const LUCK_RATIO_MODIFIER = 0.01;

@Component({})
export default class Cell extends Vue {
  @Prop()
  private uuid!: string;

  @Prop()
  private size!: number;

  @Prop()
  private seedCol!: number;

  @Prop()
  private seedRow!: number;

  @Prop({
    default: 0,
  })
  private seedR!: number;

  @Prop({
    default: 0,
  })
  private seedG!: number;

  @Prop({
    default: 0,
  })
  private seedB!: number;

  @Prop()
  private gridCols!: number;

  @Prop()
  private gridRows!: number;

  private col = this.seedCol;

  private row = this.seedRow;

  private r = {
    amount: this.seedR,
    mod: 0,
    up: true,
  };

  private g = {
    amount: this.seedG,
    mod: 0,
    up: true,
  };

  private b = {
    amount: this.seedB,
    mod: 0,
    up: true,
  };

  private opacity = 1;

  private duration = 0;

  private fuel = 0;

  private fuelGauge = 0;

  private nextCheckPoint = 0;

  private luckRatio = 0.8;

  private luckRatioModifier = 0.01;

  private isAlive = false;

  private decadence = false;

  private records: Array<{ luck?: boolean, ratio: number, duration: number, fuel: number}> = [];

  get cellStyle() {
    return {
      'z-index': 50,
      'background-color': `rgba(${this.r.amount}, ${this.g.amount}, ${this.b.amount}, ${this.opacity})`,
      width: `${this.size}px`,
      height: `${this.size}px`,
      'grid-area': `${this.row} / ${this.col}`,
    };
  }

  private mounted(): void {
    this.live();
    this.$set(this.r, 'mod', Cell.random(15, 50));
    this.$set(this.g, 'mod', Cell.random(15, 50));
    this.$set(this.b, 'mod', Cell.random(15, 50));
  }

  private live(): void {
    this.isAlive = true;
    const heartBreak = setInterval(() => {
      this.run();
      if (!this.isAlive) {
        clearInterval(heartBreak);
      }
    }, 1000);
  }

  private run(): void {
    let hasLuck: boolean|undefined;
    if (this.duration === this.nextCheckPoint) {
      hasLuck = this.check();
    }

    if (this.fuel > 0) {
      this.duration += 1;
      this.fuel -= 1;
      this.addRecord(hasLuck);
      this.evolve();
    } else {
      this.kill();
    }
  }

  private evolve(): void {
    this.updateR();
    this.updateG();
    this.updateB();

    // this.move();
    this.duplicate();
  }

  private duplicate() {
    this.$emit('duplicate', {
      uuid: this.uuid,
      r: this.r.amount,
      g: this.g.amount,
      b: this.b.amount,
    });
  }

  private move() {
    const direction = Cell.random(1, 4);
    switch (direction) {
      case 1:
        this.moveUp();
        break;
      case 2:
        this.moveDown();
        break;
      case 3:
        this.moveLeft();
        break;
      case 4:
        this.moveRight();
        break;
      default:
        break;
    }
  }

  private updateR() {
    if (this.r.up) {
      if (this.r.amount + this.r.mod <= 255) {
        this.$set(this.r, 'amount', this.r.amount + this.r.mod);
      } else {
        this.$set(this.r, 'amount', 255);
        this.$set(this.r, 'up', false);
      }
    } else if (this.r.amount - this.r.mod >= 0) {
      this.$set(this.r, 'amount', this.r.amount - this.r.mod);
    } else {
      this.$set(this.r, 'amount', 0);
      this.$set(this.r, 'up', true);
    }
  }

  private updateG() {
    if (this.g.up) {
      if (this.g.amount + this.g.mod <= 255) {
        this.$set(this.g, 'amount', this.g.amount + this.g.mod);
      } else {
        this.$set(this.g, 'amount', 255);
        this.$set(this.g, 'up', false);
      }
    } else if (this.g.amount - this.g.mod >= 0) {
      this.$set(this.g, 'amount', this.g.amount - this.g.mod);
    } else {
      this.$set(this.g, 'amount', 0);
      this.$set(this.g, 'up', true);
    }
  }

  private updateB() {
    if (this.b.up) {
      if (this.b.amount + this.b.mod <= 255) {
        this.$set(this.b, 'amount', this.b.amount + this.b.mod);
      } else {
        this.$set(this.b, 'amount', 255);
        this.$set(this.b, 'up', false);
      }
    } else if (this.b.amount - this.b.mod >= 0) {
      this.$set(this.b, 'amount', this.b.amount - this.b.mod);
    } else {
      this.$set(this.b, 'amount', 0);
      this.$set(this.b, 'up', true);
    }
  }

  private kill() {
    this.isAlive = false;
    this.$emit('die', this.uuid);
  }

  private check(): boolean {
    const hasLuck = this.hasLuck();
    this.updateFuelGauge(hasLuck);
    this.fillFuel();
    this.updateNextCheckPoint();
    this.updateLuckRatio(hasLuck);

    return hasLuck;
  }

  private fillFuel(): void {
    this.fuel = this.fuelGauge;
  }

  private updateLuckRatio(hasLuck: boolean): void {
    const increaseResult = this.luckRatio + this.luckRatioModifier;
    const decreaseResult = this.luckRatio - this.luckRatioModifier;

    if (!this.decadence && hasLuck) {
      if (increaseResult < 1) {
        this.luckRatio = increaseResult;
        this.luckRatioModifier += LUCK_RATIO_MODIFIER;
      } else {
        this.luckRatio = 1;
        // this.decadence = true;
      }
    } else if (decreaseResult > 0) {
      this.luckRatio -= decreaseResult;
      this.luckRatioModifier -= LUCK_RATIO_MODIFIER;
    } else {
      this.luckRatio = 0;
    }
  }

  private updateNextCheckPoint() {
    this.nextCheckPoint += this.fuelGauge;
  }

  private addRecord(hasLuck: boolean|undefined): void {
    this.records.push({
      luck: hasLuck,
      ratio: Cell.round(this.luckRatio),
      duration: this.duration,
      fuel: this.fuel,
    });
  }

  private updateFuelGauge(rand: boolean): void {
    if (rand) {
      if (this.fuelGauge <= 1) {
        this.fuelGauge += 1;
      } else if (this.fuelGauge < FULL_GAUGE_MAX) {
        this.fuelGauge *= 2;
      } else {
        this.fuelGauge = FULL_GAUGE_MAX;
      }
    } else if (this.fuelGauge <= 1 && this.fuelGauge !== 0) {
      this.fuelGauge -= 1;
    } else {
      this.fuelGauge /= 2;
    }
  }

  private moveUp(): void {
    const nextMove = this.row - 1;
    if (nextMove > 0) {
      this.row = nextMove;
    }
  }

  private moveDown(): void {
    const nextMove = this.row + 1;
    if (nextMove < this.gridRows) {
      this.row = nextMove;
    }
  }

  private moveLeft(): void {
    const nextMove = this.col - 1;
    if (nextMove > 0) {
      this.col = nextMove;
    }
  }

  private moveRight(): void {
    const nextMove = this.col + 1;
    if (nextMove < this.gridCols) {
      this.col = nextMove;
    }
  }

  private hasLuck(): boolean {
    return Math.random() <= this.luckRatio;
  }

  private static random(min: number, max: number) {
    return Math.floor(Math.random() * (max - min + 1) + min);
  }

  private static round(n: number) {
    return Math.round(n * 100) / 100;
  }
}
</script>
