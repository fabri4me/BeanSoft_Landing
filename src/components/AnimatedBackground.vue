<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue';

const containerRef = ref<HTMLDivElement | null>(null);
let animationFrames: number[] = [];
let mouseHandler: ((e: MouseEvent) => void) | null = null;

function createParticle(container: HTMLElement) {
  const particle = document.createElement('div');
  particle.className = 'bg-particle';

  const size = Math.random() * 3 + 1;
  particle.style.cssText = `
    position: absolute;
    border-radius: 50%;
    background: white;
    width: ${size}px;
    height: ${size}px;
    pointer-events: none;
    opacity: 0;
  `;

  const posX = Math.random() * 100;
  const posY = Math.random() * 100;
  particle.style.left = `${posX}%`;
  particle.style.top = `${posY}%`;

  container.appendChild(particle);
  animateParticle(particle, posX, posY);
}

function animateParticle(particle: HTMLElement, posX: number, posY: number) {
  particle.style.left = `${posX}%`;
  particle.style.top = `${posY}%`;
  particle.style.opacity = '0';
  particle.style.transition = 'none';

  const duration = Math.random() * 10 + 10;
  const delay = Math.random() * 5;

  const tid = setTimeout(() => {
    particle.style.transition = `all ${duration}s linear`;
    particle.style.opacity = String(Math.random() * 0.35 + 0.1);
    particle.style.left = `${posX + (Math.random() * 20 - 10)}%`;
    particle.style.top = `${posY - Math.random() * 30}%`;

    const resetTid = setTimeout(() => {
      const newX = Math.random() * 100;
      const newY = Math.random() * 100;
      animateParticle(particle, newX, newY);
    }, duration * 1000);

    animationFrames.push(resetTid as unknown as number);
  }, delay * 1000);

  animationFrames.push(tid as unknown as number);
}

onMounted(() => {
  if (!containerRef.value) return;

  const reduce = window.matchMedia?.('(prefers-reduced-motion: reduce)').matches;

  const particlesContainer = containerRef.value.querySelector<HTMLElement>('.bg-particles');
  if (particlesContainer && !reduce) {
    for (let i = 0; i < 80; i++) createParticle(particlesContainer);
  }

  mouseHandler = (e: MouseEvent) => {
    if (!containerRef.value) return;

    const mouseX = (e.clientX / window.innerWidth) * 100;
    const mouseY = (e.clientY / window.innerHeight) * 100;

    const p = document.createElement('div');
    p.style.cssText = `
      position: absolute;
      border-radius: 50%;
      background: white;
      pointer-events: none;
      width: ${Math.random() * 4 + 2}px;
      height: ${Math.random() * 4 + 2}px;
      left: ${mouseX}%;
      top: ${mouseY}%;
      opacity: 0.6;
      z-index: 3;
    `;
    particlesContainer?.appendChild(p);

    const raf = requestAnimationFrame(() => {
      p.style.transition = 'all 2s ease-out';
      p.style.left = `${mouseX + (Math.random() * 10 - 5)}%`;
      p.style.top = `${mouseY + (Math.random() * 10 - 5)}%`;
      p.style.opacity = '0';
      setTimeout(() => p.remove(), 2000);
    });
    animationFrames.push(raf);

    const moveX = (e.clientX / window.innerWidth - 0.5) * 5;
    const moveY = (e.clientY / window.innerHeight - 0.5) * 5;
    containerRef.value.querySelectorAll<HTMLElement>('.bg-sphere').forEach(sphere => {
      sphere.style.transform = `translate(${moveX}px, ${moveY}px)`;
    });
  };

  window.addEventListener('mousemove', mouseHandler, { passive: true });
});

onBeforeUnmount(() => {
  animationFrames.forEach(id => {
    clearTimeout(id);
    cancelAnimationFrame(id);
  });
  animationFrames = [];
  if (mouseHandler) window.removeEventListener('mousemove', mouseHandler);
  mouseHandler = null;
});
</script>

<template>
  <div ref="containerRef" class="bg-root pointer-events-none fixed inset-0 -z-10 overflow-hidden">
    <div class="bg-sphere bg-sphere-1"></div>
    <div class="bg-sphere bg-sphere-2"></div>
    <div class="bg-sphere bg-sphere-3"></div>
    <div class="bg-glow"></div>
    <div class="bg-grid"></div>
    <div class="bg-noise"></div>
    <div class="bg-particles"></div>
  </div>
</template>

<style scoped>
.bg-root {
  background-color: #050505;
}

.bg-sphere {
  position: absolute;
  border-radius: 50%;
  filter: blur(60px);
  transition: transform 0.3s ease-out;
}

.bg-sphere-1 {
  width: 40vw;
  height: 40vw;
  background: linear-gradient(40deg, rgba(255, 90, 0, 0.85), rgba(255, 160, 30, 0.45));
  top: -10%;
  left: -10%;
  animation: bg-float-1 15s ease-in-out infinite alternate;
}

.bg-sphere-2 {
  width: 45vw;
  height: 45vw;
  background: linear-gradient(240deg, rgba(180, 40, 0, 0.8), rgba(255, 120, 0, 0.4));
  bottom: -20%;
  right: -10%;
  animation: bg-float-2 18s ease-in-out infinite alternate;
}

.bg-sphere-3 {
  width: 30vw;
  height: 30vw;
  background: linear-gradient(120deg, rgba(255, 180, 0, 0.5), rgba(255, 100, 20, 0.3));
  top: 60%;
  left: 20%;
  animation: bg-float-3 20s ease-in-out infinite alternate;
}

.bg-glow {
  position: absolute;
  width: 40vw;
  height: 40vh;
  background: radial-gradient(circle, rgba(200, 70, 0, 0.18), transparent 70%);
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 2;
  filter: blur(30px);
  animation: bg-pulse 8s infinite alternate;
}

.bg-grid {
  position: absolute;
  inset: 0;
  background-size: 40px 40px;
  background-image:
    linear-gradient(to right, rgba(255, 255, 255, 0.03) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(255, 255, 255, 0.03) 1px, transparent 1px);
  z-index: 2;
}

.bg-noise {
  position: absolute;
  inset: 0;
  opacity: 0.05;
  z-index: 5;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.65' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}

.bg-particles {
  position: absolute;
  inset: 0;
  z-index: 3;
  pointer-events: none;
}

@keyframes bg-float-1 {
  0%   { transform: translate(0, 0) scale(1); }
  100% { transform: translate(10%, 10%) scale(1.1); }
}

@keyframes bg-float-2 {
  0%   { transform: translate(0, 0) scale(1); }
  100% { transform: translate(-10%, -5%) scale(1.15); }
}

@keyframes bg-float-3 {
  0%   { transform: translate(0, 0) scale(1); opacity: 0.3; }
  100% { transform: translate(-5%, 10%) scale(1.05); opacity: 0.6; }
}

@keyframes bg-pulse {
  0%   { opacity: 0.3; transform: translate(-50%, -50%) scale(0.9); }
  100% { opacity: 0.7; transform: translate(-50%, -50%) scale(1.1); }
}
</style>
