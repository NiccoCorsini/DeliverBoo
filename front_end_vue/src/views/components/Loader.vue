<template>
  <div class="loading">
        <div class="loader">
          <div class="duo duo1">
            <div class="dot dot-a"></div>
            <div class="dot dot-b"></div>
          </div>
          <div class="duo duo2">
            <div class="dot dot-a"></div>
            <div class="dot dot-b"></div>
          </div>
        </div>
      </div>
</template>

<script>
export default {
    name: 'Loader',
}
</script>

<style lang="scss" scoped>
.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 50px;
}
.loader {
  position: relative;
  width: 80px;
  margin: 100px auto;
}

.duo {
  height: 20px;
  width: 50px;
  background: hsla(0, 0%, 0%, 0);
  position: absolute;
}

.duo,
.dot {
  animation-duration: 0.8s;
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
}

.duo1 {
  left: 0;
}

.duo2 {
  left: 30px;
}

.dot {
  width: 20px;
  height: 20px;
  border-radius: 10px;
  background: #333;
  position: absolute;
}

.dot-a {
  left: 0px;
}

.dot-b {
  right: 0px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  50% {
    transform: rotate(180deg);
  }
  100% {
    transform: rotate(180deg);
  }
}

@keyframes onOff {
  0% {
    opacity: 0;
  }
  49% {
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    opacity: 1;
  }
}

.duo1 {
  animation-name: spin;
}

.duo2 {
  animation-name: spin;
  animation-direction: reverse;
}

.duo2 .dot-b {
  animation-name: onOff;
}

.duo1 .dot-a {
  opacity: 0;
  animation-name: onOff;
  animation-direction: reverse;
}
</style>