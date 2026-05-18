<template>
  <main>
    <div class="fixed-canvas">
      <ClientOnly>
        <TresCanvas window-size clear-color="#050505">
          <TresPerspectiveCamera :position="[0, 0, 8]" />
          
          <MorphScene />
          <TresAmbientLight :intensity="0.5" />
        </TresCanvas>
        <template #fallback>
          <div class="loader">Cargando modelos 3D...</div>
        </template>
      </ClientOnly>
    </div>

    <div id="scroll-wrapper">
      <section v-for="n in 5" :key="n" class="step-section">
        <div class="content-card">
          <span>0{{ n }}</span>
          <h2>Fase de Desintegración</h2>
        </div>
      </section>
    </div>
  </main>
</template>

<style scoped>
/* Tus estilos se quedan exactamente igual que antes... */
.fixed-canvas {
  position: fixed;
  inset: 0;
  z-index: 0;
}
#scroll-wrapper {
  height: 800vh;
  position: relative;
  z-index: 10;
}
.step-section {
  height: 200vh;
  display: flex;
  align-items: center;
  padding-left: 10%;
}
.content-card {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  padding: 2rem;
  border-left: 2px solid #00f3ff;
  color: white;
  max-width: 400px;
}
.loader {
  color: #00f3ff;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  font-family: monospace;
}
</style>