<template>
    
</template>

<script>
import confetti from 'canvas-confetti'; // Make sure to install and import the confetti library

export default {
    mounted() {
        this.startConfetti();
    },
    methods: {
        startConfetti() {
            const duration = 15 * 1000;
            const end = Date.now() + duration;
            const defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 0 };

            const randomInRange = (min, max) => Math.random() * (max - min) + min;

            const interval = setInterval(() => {
                const timeLeft = end - Date.now();
                if (timeLeft <= 0) {
                    clearInterval(interval);
                    return;
                }

                const particleCount = 50 * (timeLeft / duration);
                confetti(Object.assign({}, defaults, { particleCount, origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 } }));
                confetti(Object.assign({}, defaults, { particleCount, origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 } }));
            }, 500);
        }
    }
};
</script>
  
<style scoped>
/* Add any additional styling here */
</style>
  