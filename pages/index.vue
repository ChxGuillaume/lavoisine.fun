<template>
  <div>
    <v-card
      id="well-done"
      v-intersect="goTo"
      class="screen-card d-flex flex-column justify-center"
      color="transparent"
      elevation="0"
    >
      <v-avatar
        id="toggleConfetti"
        size="128"
        rounded
        class="mx-auto"
        :class="iconClasses"
        @click="toggleConfetti"
      >
        <v-img v-if="iconStatus" src="/confetti-512.png"/>
        <v-img v-else src="/confetti-off-512.png"/>
      </v-avatar>
      <v-card-title class="d-block my-5 text-center text-h2">Good job Julien!</v-card-title>
      <v-card-subtitle class="d-block text-center text-h4">
        You succeeded with <span class="font-italic">la voisine</span>.
      </v-card-subtitle>
      <canvas id="confetti" ref="confetti"/>
    </v-card>
    <v-card
      id="statistics"
      v-intersect="goTo"
      class="screen-card d-flex flex-column justify-center"
      color="dark-gray"
      elevation="0"
    >
      <v-card-title class="d-block text-center text-h2">Yeyyyyy!</v-card-title>
      <v-card-subtitle class="d-block text-center text-h4">Time for some Statistics</v-card-subtitle>
    </v-card>
    <v-card
      id="footer"
      v-intersect="goTo"
      class="screen-card d-flex flex-column justify-center"
      color="transparent"
      elevation="0"
    >
      <v-card-title class="d-block text-center text-h2">Hey!</v-card-title>
      <v-card-subtitle class="d-block text-center text-h4">I don't know why i made this.</v-card-subtitle>
    </v-card>
  </div>
</template>

<script>
import anime from 'animejs/lib/anime.es.js';

const confetti = require('canvas-confetti');

function randomInRange(min, max) {
  return Math.random() * (max - min) + min;
}

export default {
  data() {
    return {
      confetti: null,
      confettiColors: [
        '#78eac4',
        '#f7c4db',
        '#ef8bbd',
        '#cab4ef',
        '#a183e2',
        '#ffa585',
        '#8bb6ef',
      ],
      currentPage: 'well-done',
      showConfetti: true,
      toggleAnimation: null,
      animationStatus: false,
      iconStatus: true,
    }
  },
  computed: {
    iconClasses() {
      return this.animationStatus ? '' : ['pointer', 'transition'];
    },
  },
  mounted() {
    this.confetti = confetti.create(this.$refs.confetti, {resize: true, useWorker: true});

    this.confettiLeft();

    this.toggleAnimation = anime.timeline({
      duration: 800,
      ease: 'easeInOutSine',
      autoplay: false,
    });

    let changedIcon = false;
    this.toggleAnimation.add({
      targets: '#toggleConfetti',
      rotate: '1turn',
      scale: [
        {value: 1.5, duration: 300},
        {value: 1, duration: 300},
      ],
      begin: () => {
        changedIcon = false;
        this.animationStatus = true;
      },
      complete: () => {
        this.animationStatus = false;
      },
      update: (anim) => {
        if (anim.progress > 10 && !changedIcon) {
          changedIcon = true;
          this.iconStatus = !this.iconStatus;
          this.showConfetti = this.iconStatus;
          this.confettiLeft();
        }
      },
    });
  },
  beforeDestroy() {
    this.showConfetti = false;
  },
  methods: {
    toggleConfetti() {
      this.toggleAnimation.restart();
    },
    confettiLeft() {

      if (this.showConfetti && this.currentPage === 'well-done') {
        this.confetti({
          angle: randomInRange(30, 90),
          colors: this.confettiColors,
          spread: 100,
          particleCount: Math.max(window.innerWidth / 2560 * 150, 100),
          startVelocity: Math.max(window.innerWidth / 2560 * 75, 20),
          origin: {x: 0, y: Math.max(window.innerWidth / 2560 * 0.6, 0.2)},
          ticks: 300,
        })

        setTimeout(() => window.requestAnimationFrame(this.confettiRight), Math.random() * 1000 + 125);
      }
    },
    confettiRight() {
      if (this.showConfetti && this.currentPage === 'well-done') {
        this.confetti({
          angle: randomInRange(180 - 30, 180 - 90),
          colors: this.confettiColors,
          spread: 100,
          particleCount: Math.max(window.innerWidth / 2560 * 150, 100),
          startVelocity: Math.max(window.innerWidth / 2560 * 75, 20),
          origin: {x: 1, y: Math.max(window.innerWidth / 2560 * 0.6, 0.2)},
          ticks: 300,
        })

        setTimeout(() => window.requestAnimationFrame(this.confettiLeft), Math.random() * 1000 + 125);
      }
    },
    goTo(entries) {
      if (entries[0].isIntersecting) {
        this.currentPage = entries[0].target.id;

        this.$vuetify.goTo(`#${this.currentPage}`, {
          duration: 500,
          easing: 'easeInOutCubic'
        }).then(() => {
          if (entries[0].target.id === 'well-done') {
            this.confettiLeft();
          }
        });
      }
    },
  }
}
</script>

<style scoped>
#confetti {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;

  pointer-events: none;
}

.transition {
  transition: .2s ease;
}

.pointer {
  cursor: pointer;
}

.pointer:hover {
  transform: scale(0.8) !important;
}

.screen-card {
  position: relative;

  width: 100vw;
  min-height: 100vh;

  z-index: 1;
}

.screen-card:first-child {
  margin-bottom: 50px;
}

.screen-card:not(:first-child):not(:last-child) {
  margin: 50px 0;
}

.screen-card:last-child {
  margin-top: 50px;
}
</style>
