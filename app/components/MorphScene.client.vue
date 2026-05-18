<template>
  <TresGroup v-if="isReady">
    <TresPoints ref="pointsRef">
      <TresBufferGeometry ref="geometryRef" />
      <TresPointsMaterial 
        :size="0.04" 
        color="#746f29" 
        :transparent="true" 
        :opacity="0.8"
        :size-attenuation="true"
        :blending="THREE.AdditiveBlending"
      />
    </TresPoints>
  </TresGroup>
  
  <TresAmbientLight :intensity="1.5" />
  <TresDirectionalLight :position="[5, 5, 5]" :intensity="1" />
</template>

<script setup>
import * as THREE from 'three'
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader'
import { MeshSurfaceSampler } from 'three/examples/jsm/math/MeshSurfaceSampler'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { ref, onMounted, nextTick } from 'vue' // Añadimos las importaciones de Vue por seguridad

gsap.registerPlugin(ScrollTrigger)

const geometryRef = ref(null)
const isReady = ref(false)

// Configuración técnica
// 1. AJUSTE: Definimos las 20,000 partículas desde aquí arriba de forma global
const particleCount = 20000 
const currentPositions = new Float32Array(particleCount * 3)

const loadModels = async () => {
  const loader = new STLLoader()
  const urls = [
    '/models/1.stl', '/models/2.stl', '/models/3.stl', 
    '/models/4.stl', '/models/5.stl'
  ]

  try {
    const geometriesData = await Promise.all(urls.map(async (url) => {
      const geo = await loader.loadAsync(url)
      geo.rotateX(-Math.PI / 2) 
      geo.center() 

      geo.computeBoundingSphere()
      if (geo.boundingSphere.radius > 0) {
        const scale = 3 / geo.boundingSphere.radius // Escala estándar
        geo.scale(scale, scale, scale)
      } else {
        console.warn(`El modelo en ${url} tiene un radio de 0. Verifica los vértices.`)
      }

      const tempMesh = new THREE.Mesh(geo)
      const sampler = new MeshSurfaceSampler(tempMesh).build()
      
      // Usará el particleCount de 20,000 definido arriba
      const sampledPoints = new Float32Array(particleCount * 3)
      const tempVec = new THREE.Vector3()

      for (let i = 0; i < particleCount; i++) {
        sampler.sample(tempVec)
        sampledPoints[i * 3] = tempVec.x
        sampledPoints[i * 3 + 1] = tempVec.y
        sampledPoints[i * 3 + 2] = tempVec.z
      }
      return sampledPoints
    }))

    currentPositions.set(geometriesData[0])
    isReady.value = true

    nextTick(() => {
      if (geometryRef.value) {
        geometryRef.value.setAttribute(
          'position', 
          new THREE.BufferAttribute(currentPositions, 3)
        )
        setupScrollAnimation(geometriesData)
      }
    })
  } catch (err) {
    console.error("Error cargando o procesando los STL:", err)
  }
}

const setupScrollAnimation = (geometriesData) => {
  const tl = gsap.timeline({
    scrollTrigger: {
      trigger: '#scroll-wrapper',
      start: 'top top',
      end: 'bottom bottom',
      scrub: 1.5,
    }
  })

  // Animación secuencial entre piezas
  for (let i = 0; i < geometriesData.length - 1; i++) {
    tl.to(currentPositions, {
      endArray: geometriesData[i + 1],
      duration: 1,
      ease: "power2.inOut",
      onUpdate: () => {
        if (geometryRef.value) {
          geometryRef.value.attributes.position.needsUpdate = true
        }
      }
    })
  }
}

onMounted(() => {
  loadModels()
})
</script>