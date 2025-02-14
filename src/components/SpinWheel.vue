<template>
  <Dropdown
    id="group-dropdown"
    :model-value="GroupLabel"
    :options="GroupLabels"
    class="mt-4 z-1"
    @update:model-value="itemService?.changeGroupLabel"
    :pt="{
      input: {
        class: 'text-xl sm:text-4xl md:text-6xl'
      },
      item: {
        class: 'text-xl sm:text-xl md:text-3xl'
      }
    }"
  />
  <ShareLink class="w-full flex justify-content-center z-1 mt-3" />
  <div ref="container" class="flex spin-container">
    <picture>
      <source srcset="/img/image.avif" type="image/avif" />
      <source srcset="/img/image.webp" type="image/webp" />
      <img src="/img/image.png" class="image" alt="background image" />
    </picture>
    <div
      class="icon"
      @click="spin"
      @keyup.enter="spin"
      @keyup.space="spin"
      v-tooltip.bottom="{
        value: `↻ Spin!`,
        class: 'text-xl',
        escape: true
      }"
      tabindex="0"
    ></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, inject, watch } from 'vue';
import random from 'random';
import { Wheel, type WheelProps } from 'spin-wheel';
import { useDialog } from 'primevue/usedialog';
import { TickSound, LabelLength } from '@/services/SettingService';
import { GroupLabel, GroupLabels, ItemService, Items } from '@/services/ItemService';
import CongratulationDialog from '@/components/CongratulationDialog.vue';

const itemService = inject<ItemService>('ItemService');

const properties: WheelProps = {
  // debug: import.meta.env.DEV,
  isInteractive: false,
  radius: 0.48,
  rotationResistance: 0,
  itemLabelRadius: 0.92,
  itemLabelRadiusMax: 0.3,
  itemLabelRotation: 180,
  itemLabelAlign: 'left',
  itemLabelColors: ['#fff'],
  itemLabelBaselineOffset: -0.07,
  // Should also change app.scss
  itemLabelFont:
    '"Suez One", "Mochiy Pop P One", "Jua", "Unbounded", "Mitr", "Noto Sans TC", "Noto Sans SC", "Noto Sans Lao", "Noto Color Emoji"',
  itemLabelFontSizeMax: 55,
  itemBackgroundColors: [
    '#3F3F3F',
    '#A0000C',
    '#7EA53A',
    '#00773B',
    '#602796',
    '#1233AF',
    '#008E87',
    '#B26B24',
    '#CC00B4',
    '#C1BC00',
    '#FF6161'
  ],
  rotationSpeedMax: 2000,
  lineWidth: .3,
  lineColor: '#000',
  items: []
};

const container = ref();

let spinCount = 10;
let wheel: Wheel | undefined = undefined;

const stopAndClearSound = () => {
  if (!wheel) return;

  wheel.onCurrentIndexChange = () => {};
  wheel.stop();
};

const playSound = () => {
  if (!TickSound.value) return;

  var src = TickSound.value.value.startsWith('data:')
    ? TickSound.value.value
    : `/sound/${TickSound.value.value}`;
  const audio = new Audio(src);
  audio.volume = 0.2;
  audio.play();
};

const spin = () => {
  if (!wheel) return;

  wheel.onCurrentIndexChange = () => {
    if (!wheel) return;

    playSound();

    // Change rotation resistance based on current speed.
    // Provide a more entertaining performance.
    switch (true) {
      case wheel.rotationSpeed < 400:
        wheel.rotationResistance = -500;
        break;
      case wheel.rotationSpeed < 100:
        wheel.rotationResistance = -50;
        break;
      case wheel.rotationSpeed < 30:
        wheel.rotationResistance = 10;
        break;
    }
  };

  wheel.rotationResistance = -40;
  wheel.spin(wheel.rotationSpeed + random.int(800, 1400));
};

const dialog = useDialog();
const openCongratulationDialog = ($event: {
  type: 'rest';
  currentIndex: number;
  rotation: number;
}) => {
  dialog.open(CongratulationDialog, {
    props: {
      modal: true,
      showHeader: false,
      style: 'border: 0',
      contentStyle: 'border: 0; backgroundColor: transparent',
      dismissableMask: true
    },
    data: {
      item: Items.value![$event.currentIndex]
    }
  });
};

onMounted(() => {
  watch(Items, (newValue) => (wheel!.items = newValue || []));
  watch(LabelLength, (newValue) => {
    wheel!.itemLabelRadiusMax = 1 - newValue;
  });

  wheel = new Wheel(container.value, {
    ...properties,
    items: Items.value,
    itemLabelRadiusMax: 1 - LabelLength.value
  });

  wheel.spin(50);

  wheel.onRest = ($event) => {
    stopAndClearSound;
    openCongratulationDialog($event);
  };

  wheel.onSpin = () => {
    gtag('event', 'spin');
    gtag('event', 'spin_count', {
      count: ++spinCount
    });
  };

  // Workaround for itemLabelRadiusMax not working on first load.
  setTimeout(() => {
    wheel!.itemLabelRadiusMax = 1 - LabelLength.value;
  }, 50);
});
</script>

<style lang="scss" scoped>
@import 'primeflex/core/_variables.scss';

.spin-container {
  aspect-ratio: 1/1;
  width: 200vw;
  height: 90vh;

  margin-top: -3.5rem;
  margin-bottom: -10vh;
  position: relative;

  @media (min-width: map-get($breakpoints, 'sm')) {
    height: 100vh;
  }

  @media (min-width: map-get($breakpoints, 'md')) {
    height: 110vh;
  }
}

.image {
  object-position: center;
  object-fit: contain;

  aspect-ratio: 1/1;
  width: 200vw;
  height: 90vh;

  position: absolute;
  top: calc(calc(50%) - calc(90vh / 2));
  left: calc(calc(50%) - calc(200vw / 2));

  @media (min-width: map-get($breakpoints, 'sm')) {
    height: 100vh;
    top: calc(calc(50%) - calc(100vh / 2));
  }

  @media (min-width: map-get($breakpoints, 'md')) {
    height: 110vh;
    top: calc(calc(50%) - calc(110vh / 2));
  }
}

.button-container {
  margin-top: -5.5rem;

  button {
    z-index: 2;
    position: relative;

    $background-color: #0c0f1d;
    background: $background-color;

    &:hover {
      filter: brightness(1.3);
    }
  }
}

.icon {
  $icon-size: 13vh;
  cursor: pointer;

  width: $icon-size;
  height: $icon-size;
  border-radius: 50%;

  background-image: url(/img/icon.png);
  background-image: -webkit-image-set(
    url(/img/icon.avif) type('image/avif'),
    url(/img/icon.webp) type('image/webp'),
    url(/img/icon.png) type('image/png')
  );
  background-image: image-set(
    url(/img/icon.avif) type('image/avif'),
    url(/img/icon.webp) type('image/webp'),
    url(/img/icon.png) type('image/png')
  );

  background-size: contain;

  position: absolute;
  top: calc(calc(50%) - calc($icon-size / 2));
  left: calc(calc(50%) - calc($icon-size / 2));

  &:hover {
    filter: brightness(1.1);
  }
}
</style>
