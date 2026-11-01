
<script setup>
import { ref, onMounted } from 'vue'
import PokemonCard from './components/pokemon_Card.vue'

const pokemones = ref([])
const cargando = ref(true)
const error = ref('')

// Obtener 40 ids aleatorios sin repetir
function obtenerIdsAleatorios() {
  const ids = new Set()

  while (ids.size < 40) {
    ids.add(Math.floor(Math.random() * 1025) + 1)
  }

  return Array.from(ids)
}

// Obtener información de un Pokémon
async function obtenerPokemon(id) {
  const respuesta = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`)

  if (!respuesta.ok) {
    throw new Error('No se pudo obtener un Pokémon')
  }

  const datos = await respuesta.json()

  return {
    id: datos.id,
    name: datos.name,
    height: datos.height / 10,
    weight: datos.weight / 10,
    experience: datos.base_experience ?? 'No disponible',
    types: datos.types.map(tipo => tipo.type.name),
    abilities: datos.abilities
      .slice(0, 2)
      .map(habilidad => habilidad.ability.name),
    image: datos.sprites.other['official-artwork'].front_default
      || datos.sprites.front_default
  }
}

// Obtener los 40 Pokémon
async function cargarPokemones() {
  cargando.value = true
  error.value = ''

  try {
    const ids = obtenerIdsAleatorios()

    const resultados = await Promise.all(
      ids.map(id => obtenerPokemon(id))
    )

    pokemones.value = resultados
  } catch (err) {
    error.value = 'Ocurrió un error al cargar los Pokémon.'
    console.error(err)
  } finally {
    cargando.value = false
  }
}

onMounted(() => {
  cargarPokemones()
})
</script>

<template>
  <main>
    <header class="encabezado">
      <h1>POKÉMON DECK</h1>
      <p>¡40 Pokémons aleatorios!</p>

      <button @click="cargarPokemones">
        Generar otro deck
      </button>
    </header>

    <div v-if="cargando" class="mensaje">
      Cargando tus Pokémon...
    </div>

    <div v-else-if="error" class="mensaje">
      {{ error }}
      <button @click="cargarPokemones">Reintentar</button>
    </div>

    <section v-else class="pokemon-grid">
      <PokemonCard
        v-for="pokemon in pokemones"
        :key="pokemon.id"
        :pokemon="pokemon"
      />
    </section>
  </main>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: #202b40;
  font-family: Arial, sans-serif;
}

.encabezado {
  text-align: center;
  padding: 30px 15px;
  color: white;
}

.encabezado h1 {
  font-size: 38px;
  color: #ffcb05;
  text-shadow: 3px 3px #3466af;
}

.encabezado p {
  font-size: 17px;
}

button {
  background: #ffcb05;
  color: #202b40;
  border: none;
  padding: 12px 22px;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  margin-top: 10px;
}

button:hover {
  background: #f2b800;
}

.pokemon-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(250px, 1fr));
  gap: 25px;
  width: 100%;
  max-width: 1400px;
  margin: 30px auto;
  padding: 20px;
  justify-items: stretch;
}

.mensaje {
  text-align: center;
  color: white;
  padding: 50px;
  font-size: 20px;
}

@media (max-width: 1150px) {
  .pokemon-grid {
    grid-template-columns: repeat(3, minmax(220px, 1fr));
  }
}

@media (max-width: 850px) {
  .pokemon-grid {
    grid-template-columns: repeat(2, minmax(220px, 1fr));
  }
}

@media (max-width: 550px) {
  .pokemon-grid {
    grid-template-columns: 1fr;
  }
}
</style>
