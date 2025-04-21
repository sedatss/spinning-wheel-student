<template>
  <div class="container">
    <canvas ref="wheelCanvas" width="500" height="500" @click="spinWheel"></canvas>
    <button @click="spinWheel">Spin the Wheel</button>
    <div v-if="winner" class="modal">
      <div class="modal-content">
        <h2>Congratulations!</h2>
        <p>The winner is: <strong>{{ winner }}</strong></p>
        <button @click="closeModal">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { defineProps, onMounted, ref } from 'vue';

const wheelCanvas = ref<HTMLCanvasElement | null>(null);
const rotationAngle = ref<number>(0);
const winner = ref<string | null>(null);
const isSpinning = ref<boolean>(false);

const props = defineProps<{
  names: { name: string; color: string | null }[];
}>();

const segmentAngle = (2 * Math.PI) / props.names.length;

onMounted(function() : void  {
  drawWheel();
});

function drawWheel(): void {
  const canvas = wheelCanvas.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  // Clear canvas before redrawing
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Draw each segment
  props.names.forEach((segment, index) => {
    ctx.save();
    ctx.translate(canvas.width / 2, canvas.height / 2);
    ctx.rotate(segmentAngle * index + rotationAngle.value);
    ctx.beginPath();
    ctx.moveTo(0, 0);
    ctx.lineTo(canvas.width / 2, 0);
    ctx.arc(0, 0, canvas.width / 2, 0, segmentAngle);
    ctx.lineTo(0, 0);
    ctx.fillStyle = segment.color?.toLocaleLowerCase() || '#8ddba4';
    ctx.fill();
    ctx.stroke();

    // Draw the text on each segment
    ctx.fillStyle = 'black';
    ctx.font = '16px Arial';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.rotate(segmentAngle / 2);
    ctx.fillText(segment.name, (canvas.width / 2) / 2, 0);
    ctx.restore();
  });

  // draw the winners triangle
  const triangleX = canvas.width;
  const triangleY = canvas.height / 2;

  ctx.beginPath();
  ctx.moveTo(triangleX, triangleY - 10);
  ctx.lineTo(triangleX - 20, triangleY);
  ctx.lineTo(triangleX, triangleY + 10);
  ctx.closePath();
  ctx.fillStyle = 'black';
  ctx.fill();
}

// Spin the wheel
function spinWheel() : void {
  if (isSpinning.value) return;

  isSpinning.value = true;

  const startTime = performance.now();
  const spinDuration = 4000;
  const maxSpinAngle = Math.random() * 2 * Math.PI + 4 * Math.PI;
  const currentAngle = rotationAngle.value;

  // Function to animate the spin
  function animateSpin(currentTime: number) {
    const elapsedTime = currentTime - startTime;
    if (elapsedTime < spinDuration) {
      const progress = elapsedTime / spinDuration;
      const easingProgress = 1 - Math.pow(1 - progress, 3);

      rotationAngle.value = currentAngle + easingProgress * maxSpinAngle;

      requestAnimationFrame(animateSpin);
    } else {
      rotationAngle.value = currentAngle + maxSpinAngle;
      isSpinning.value = false;
      determineWinner();
    }

    drawWheel();
  }

  requestAnimationFrame(animateSpin);
}

function determineWinner(): void {
  const canvas = wheelCanvas.value;
  if (!canvas) return;

  // Normalize rotation angle to [0, 2π]
  const normalizedRotation = (rotationAngle.value % (2 * Math.PI) + 2 * Math.PI) % (2 * Math.PI);

  // Invert the rotation since we rotate the wheel clockwise visually
  const pointerAngle = (2 * Math.PI - normalizedRotation) % (2 * Math.PI);

  const winnerIndex = Math.floor(pointerAngle / segmentAngle) %props.names.length;
  winner.value = props.names[winnerIndex].name;
}

function closeModal(): void {
  winner.value = null;
}

</script>

<style scoped>

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

canvas {
  margin-bottom: 20px;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  max-width: 300px;
  width: 100%;
}
</style>
