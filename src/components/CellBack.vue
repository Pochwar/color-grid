<template>
  <div>
    COUCOU
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

const FULL_GAUGE_MAX = 256;

const LUCK_RATIO_MODIFIER = 0.01;

@Component({})
export default class Grid extends Vue {
  @Prop()
  private x!: number;

  @Prop()
  private y!: number;

  private r = 0;

  private g = 0;

  private b = 0;

  private duration = 0;

  private fuel = 0;

  private fuelGauge = 0;

  private nextCheckPoint = 0;

  private luckRatio = 0.6;

  private luckRatioModifier = 0.01;

  private isAlive = false;

  private decadence = false;

  private records: Array<{ luck?: boolean, ratio: number, duration: number, fuel: number}> = [];

  private mounted(): void {
    this.addRecord(undefined);
  }

  private live(): void {
    this.isAlive = true;
    while (this.isAlive) {
      this.run();
    }
  }

  private check(): boolean {
    const hasLuck = this.hasLuck();
    this.updateFuelGauge(hasLuck);
    this.fillFuel();
    this.updateNextCheckPoint();
    this.updateLuckRatio(hasLuck);

    return hasLuck;
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
    } else {
      this.isAlive = false;
    }
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

  private updateNextCheckPoint() {
    this.nextCheckPoint += this.fuelGauge;
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
        this.decadence = true;
      }
    } else if (decreaseResult > 0) {
      this.luckRatio -= decreaseResult;
      this.luckRatioModifier -= LUCK_RATIO_MODIFIER;
    } else {
      this.luckRatio = 0;
    }
  }

  private reset(): void {
    this.fuel = 0;
    this.fuelGauge = 0;
    this.luckRatio = 0.7;
    this.duration = 0;
    this.records = [];
    this.decadence = false;
  }

  private addRecord(hasLuck: boolean|undefined): void {
    this.records.push({
      luck: hasLuck,
      ratio: Grid.round(this.luckRatio),
      duration: this.duration,
      fuel: this.fuel,
    });
  }

  private hasLuck(): boolean {
    return Math.random() <= this.luckRatio;
  }

  private static round(n: number) {
    return Math.round(n * 100) / 100;
  }
}
</script>
