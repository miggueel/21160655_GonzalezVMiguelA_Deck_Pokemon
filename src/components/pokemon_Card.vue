
<script setup lang="ts">
import { ref, computed, watch } from 'vue'

const props = defineProps({
  pokemon: { type: Object, required: true }
})

const nombre = ref('')
const tipos = ref<string[]>([])
const habilidades = ref<string[]>([])
const cargando = ref(true)

const colores: Record<string, string> = {
  normal: '#C6C6A7', fire: '#F08030', water: '#6890F0',
  electric: '#F8D030', grass: '#78C850', ice: '#98D8D8',
  fighting: '#C03028', poison: '#A040A0', ground: '#E0C068',
  flying: '#A890F0', psychic: '#F85888', bug: '#A8B820',
  rock: '#B8A038', ghost: '#705898', dragon: '#7038F8',
  dark: '#705848', steel: '#B8B8D0', fairy: '#EE99AC'
}

const nombresTipos: Record<string, string> = {
  normal: 'Normal', fire: 'Fuego', water: 'Agua',
  electric: 'Eléctrico', grass: 'Planta', ice: 'Hielo',
  fighting: 'Lucha', poison: 'Veneno', ground: 'Tierra',
  flying: 'Volador', psychic: 'Psíquico', bug: 'Bicho',
  rock: 'Roca', ghost: 'Fantasma', dragon: 'Dragón',
  dark: 'Siniestro', steel: 'Acero', fairy: 'Hada'
}

const fondoCarta = computed(() => {
  const t = props.pokemon.types || []
  const c1 = colores[t[0]] || '#C6C6A7'
  const c2 = colores[t[1]] || c1
  return t.length > 1
    ? `linear-gradient(135deg, ${c1}, ${c2})`
    : c1
})

async function traducir() {
  cargando.value = true

  try {
    const r = await fetch(`https://pokeapi.co/api/v2/pokemon/${props.pokemon.id}`)
    if (!r.ok) throw new Error('Error al consultar Pokémon')
    const p = await r.json()

    const especie = await (await fetch(p.species.url)).json()
    nombre.value = especie.names.find(
      (n: any) => n.language.name === 'es'
    )?.name || p.name

    tipos.value = await Promise.all(p.types.map(async (t: any) => {
      const info = await (await fetch(t.type.url)).json()
      return info.names.find(
        (n: any) => n.language.name === 'es'
      )?.name || nombresTipos[t.type.name] || t.type.name
    }))

    habilidades.value = await Promise.all(
      p.abilities.slice(0, 2).map(async (a: any) => {
        const info = await (await fetch(a.ability.url)).json()
        return info.names.find(
          (n: any) => n.language.name === 'es'
        )?.name || a.ability.name
      })
    )
  } catch (error) {
    console.error(error)
    nombre.value = props.pokemon.name
    tipos.value = props.pokemon.types || []
    habilidades.value = props.pokemon.abilities || []
  } finally {
    cargando.value = false
  }
}

watch(() => props.pokemon.id, traducir, { immediate: true })
</script>

<template>
  <article class="pokemon-card" :style="{ background: fondoCarta }">
    <header>
      <div>
        <small>POKÉMON</small>
        <h2>{{ nombre || pokemon.name }}</h2>
      </div>
      <strong>N.º {{ pokemon.id }}</strong>
    </header>

    <div class="imagen">
      <img :src="pokemon.image" :alt="nombre || pokemon.name">
    </div>

    <div class="tipos">
      <span v-for="(tipo, i) in tipos" :key="i">{{ tipo }}</span>
    </div>

    <section>
      <h3>Características</h3>
      <div class="datos">
        <div><small>ALTURA</small><strong>{{ pokemon.height }} m</strong></div>
        <div><small>PESO</small><strong>{{ pokemon.weight }} kg</strong></div>
        <div class="experiencia">
          <small>EXPERIENCIA BASE</small>
          <strong>{{ pokemon.experience ?? 'No disponible' }}</strong>
        </div>
      </div>
    </section>

    <section>
      <h3>Habilidades</h3>
      <p v-if="cargando">Cargando...</p>
      <p v-else v-for="(h, i) in habilidades" :key="i">✦ {{ h }}</p>
    </section>
  </article>
</template>

<style scoped>
.pokemon-card {
  width: 100%;
  min-width: 0;
  box-sizing: border-box;
  padding: 12px;
  border: 6px solid #d6b45c;
  border-radius: 17px;
  color: #292929;
  display: flex;
  flex-direction: column;
  gap: 10px;
  box-shadow: 0 7px 15px #0005;
  transition: transform .25s;
}

.pokemon-card:hover { transform: translateY(-6px); }

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #ffffffbf;
  padding: 9px;
  border-radius: 8px;
}

header small { letter-spacing: 2px; font-weight: bold; }
header h2 { margin: 3px 0 0; font-size: 20px; text-transform: capitalize; }

.imagen {
  height: 190px;
  background: #fff;
  border: 3px solid #d4b35e;
  border-radius: 9px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.imagen img {
  width: 95%;
  height: 95%;
  object-fit: contain;
}

.tipos { display: flex; justify-content: center; flex-wrap: wrap; gap: 7px; }

.tipos span {
  background: #ffffffdf;
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: bold;
}

section { background: #ffffffc9; padding: 10px; border-radius: 8px; }
section h3 { margin: 0 0 9px; font-size: 13px; text-transform: uppercase; }
section p { margin: 5px 0; font-size: 12px; text-transform: capitalize; }

.datos { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
.datos div { display: flex; flex-direction: column; gap: 4px; }
.datos small { font-size: 9px; color: #555; font-weight: bold; }
.datos strong { font-size: 13px; }
.experiencia { grid-column: span 2; }
</style>
