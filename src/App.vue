<template>
    <Header 
        :carrito="carrito"
        :guitarra="guitarra" 
        @incrementar-cantidad="incrementarCantidad" 
        @decrementar-cantidad="decrementarCantidad"
        @agregar-carrito="agregarCarrito"
        @eliminar-guitarra="eliminarGuitarra"
        @vaciar-carrito="vaciarCarrito"
    />

    <main class="container-xl mt-5">
        <h2 class="text-center">Nuestra Colección</h2>

        <div class="row mt-5">
            <Guitarra 
                v-for="guitarra in guitarras"
                :key="guitarra.id"
                :guitarra="guitarra"
                @agregar-carrito="agregarCarrito"
            />
        </div>
    </main>

    <Footer />
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { db } from '@/data/guitarras'
import Guitarra from '@/components/Guitarra.vue'
import Header from '@/components/Header.vue'
import Footer from '@/components/Footer.vue'

// Reactive siempre maneja objetos, ideal para agrupar valores
// const state = reactive({ guitarras: [] });

// Ref maneja arreglor, boolean u otros tipos de datos
const guitarras = ref([]);
const carrito = ref([]);
const guitarra = ref({});

// Watch permite la observacion de cambios en el state y reducir la utilizacion de codigo con dicho state
watch( carrito , () => {
    // Cada vez que se agrega un elemento al carrito, este se queda guardado
    guardarLocalStorage();
}, {
    // Entra a todos los elementos del carrito y revisa incluso los valores de cantidades a pedir de un producto
    deep: true
})

// onMounted e sun metodo de ciclo de vida de componente, 
// se ejecuta cuando el componente ya es creado
onMounted(() => {
    // guitarras.value = db; Con Ref
    // state.guitarras = db; Con Reactive

    guitarras.value = db;
    guitarra.value = db[3];

    // Cargar carrito desde LS
    const carritoLS = localStorage.getItem( "carrito" );
    if( carritoLS ) carrito.value = JSON.parse( carritoLS );
})

const guardarLocalStorage = () => {
    // Si se eliminan las guitarras, la variable de LS queda reescrita con nuevas
    localStorage.setItem( "carrito", JSON.stringify(carrito.value) );
}

// Agrega una guitarra al carrito desde el componente hijo con Custom Emits
const agregarCarrito = ( guitarra ) => {
    // Comprobar si ya existe el producto en el carrito, si exite devuelve su posicion en el arreglo
    const existeCarrito = carrito.value.findIndex( producto => producto.id === guitarra.id );
    
    if( existeCarrito >= 0 ) {
        // Si ya existe solo modifica la cantidad a llevar
        carrito.value[ existeCarrito ].cantidad++;
    } else {
        guitarra.cantidad = 1;
        carrito.value.push( guitarra );
    }
}

// Decrementa la cantidad que hay en el carrito con Custom Emits
const decrementarCantidad = (id) => {
    // Comprobar si ya existe el producto en el carrito, si exite devuelve su posicion en el arreglo
    const index = carrito.value.findIndex( producto => producto.id === id );

    // Si la cantidad es uno o menor no hace nada
    if( carrito.value[ index ].cantidad <= 1 ) return

    carrito.value[ index ].cantidad--; 
}

// Incrementa la cantidad que hay en el carrito con Custom Emits
const incrementarCantidad = (id) => {
    // Comprobar si ya existe el producto en el carrito, si exite devuelve su posicion en el arreglo
    const index = carrito.value.findIndex( producto => producto.id === id );

    // Se limita la cantidad a pedir
    if( carrito.value[ index ].cantidad >= 5 ) return

    carrito.value[ index ].cantidad++;
}

// Eliminar una guitarra del carrito
const eliminarGuitarra = (id) => {
    carrito.value = carrito.value.filter( guitarra => guitarra.id !== id)
}

// Eliminar todas las guitarras dek carrito
const vaciarCarrito = () => {
    carrito.value.splice(0, carrito.value.length);
}
</script>