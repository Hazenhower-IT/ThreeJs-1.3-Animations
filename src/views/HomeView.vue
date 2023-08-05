<template>
  <canvas ref="canvasRef"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted} from "vue"
import * as THREE from "three"
import gsap from "gsap"


let canvasRef = ref();

const sizes = {
  width: 800,
  height: 600
}

// Per iniziare con threejs abbiamo bisogno di 4 elementi, una scena, una camera, un renderer, e degli oggetti da mostrare
//la Scena e la camera vengono inizializzate nel setup mentre il renderer nel onMounted

// La scena è come un container al cui interno posizioniamo i nostri oggetti, luci, modelli ecc.
let scene = new THREE.Scene()

// il renderer ci serve per dire a threeJs di renderizzare questa scena
let renderer

/* 
  All'interno della scena reindirizzeremo i nostri oggetti 
*/



// LA CAMERA
// 1) La camera ci serve per avere appunto il "punto di vista" sulla scena
//    Il renderer è come se facesse una foto alla nostra scena, ma da dove la sta facendo questa foto? 
//    modificando la posizione della camera, ed i suoi parametri noi cambiamo questo punto di vista. 
let camera = new THREE.PerspectiveCamera(
  75, //vertical field of view
  sizes.width / sizes.height, //aspect ratio
  /*Optional
  0.1, //near plane
  100 //far plane
  */
);

camera.position.set(0,0,3)

scene.add(camera);

// LookAt
//Un altra interessante funzione è LookAt, in sostanza possiamo dire a un 
//qualsiasi Object3D di "guardare" o "puntare" a qualcosa.
//per esempio possiamo dire alla camera di guardare il centro della scena
camera.lookAt(new THREE.Vector3(0,0,0))
//o di guardare al cubo per esempio, ma in questo caso dovremmo scrivere
//questo metodo dopo aver aggiunto il box alla scena altrimenti avremo un errore
//camera.lookAt(box.position)


//AXES HELPER
const axesHelper = new THREE.AxesHelper()
scene.add(axesHelper)

// BOX
const boxGeometry = new THREE.BoxGeometry(1,1,1)
const boxMaterial = new THREE.MeshBasicMaterial({color: 0xff0000}) 

const box = new THREE.Mesh(boxGeometry,boxMaterial)

// Posizione
//box.position.x = 0.7
//box.position.y = -0.6
//box.position.z = 1
box.position.set(0.7 , -0.6, 1)

// Scala
//box.scale.x = 2
//box.scale.y = 0.5
//box.scale.z = 0.5 
box.scale.set(2, 0.5, 0.5)

// Rotazione
//N.B.:Durante le rotazioni è possibile che si crei un errore che blocca 
//gli assi, è possibile risolverlo utilizzando questo metodo 
//per riorganizzare l'ordine degli assi. p.s. l impostazione seguente 
//si usa per avere la corretta visuale in un gioco fps
//box.rotation.reorder("YXZ")
box.rotation.x = Math.PI * 0.25
//box.rotation.y = Math.PI * 0.25

// Quaternion
//Un altro metodo efficace per risolvere il problema delle rotazioni
// è utilizzare il quaternion. Il quaternion è una rappresentazione della
//rotazione ma usando una forma piu matematica. Puoi quindi ottenere lo stesso
//risultato della rotazione ma senza incappare in quei problemi.
//Il problema con i quaternion è che sono difficili da immaginare,
//ma possiamo risolvere questo problema in quanto modificando la rotazione,
//viene in automatico aggiornato il valore dei quaternion, quindi possiamo
//letteralmente modificare la rotazione e incollare poi il valore in quaternion

//aggiungiamo la mesh alla scena
//scene.add(box)


// I Gruppi di Oggetti
// In ThreeJS possiamo creare gruppi di oggetti, questo ci consente ad esempio
// di muoverli come fossero un solo oggetto
const group = new THREE.Group()
scene.add(group)
group.position.y = 0
group.scale.y = 2


const cube1 = new THREE.Mesh(
  new THREE.BoxGeometry(),
  new THREE.MeshBasicMaterial({color: 0xff0000})
)
 
const cube2 = new THREE.Mesh(
  new THREE.BoxGeometry(),
  new THREE.MeshBasicMaterial({color: 0x00ff00})
)
cube2.position.x = -2
const cube3 = new THREE.Mesh(
  new THREE.BoxGeometry(),
  new THREE.MeshBasicMaterial({color: 0x0000ff})
)
cube3.position.x = 2

group.add(cube1,cube2,cube3)



// ANIMATION (1)
// Sfortunatamente visto che alcuni schermi hanno un framerate piu alto e altri piu basso,
// le animazioni scorreranno piu o meno velocemente. Per risolvere questo problema ci sono
// piu modi. 

//1) USING TIME
// 1) Adattarci al framerate è il valore in "tempo", in questo modo
// avremo la stessa animazione su tutti i dispositivi

//1)usiamo una variabile per riprendere il valore del "tempo" corrente
//let time = Date.now()// in realta questo valore rappresenta il tempo che è passato dal 1 gennaio 1970 (in millisecondi) ed è usato dai pc per "calcolare" una data

//2) USING CLOCK
// clock è una funziona "built-in" (cioè gia presente) in threeJS, che pressapoco fa la stessa cosa che abbiamo fatto poco fa con time
//const clock = new THREE.Clock()

//3) USING AN ETERNAL LIBRARY (GSAP)
// se vuoi avere maggiore controllo, e funzionalità avanzate come creare "tweens" (transazioni da A a B), timeline, delay etc puoi usare una libreria esterna
// in questo caso useremo GSAP, installiamo quindi gsap con npm install gsap@3.5.1 (in questo caso usiamo una versione specifica, invece il comando per installare l'ultima versione della libreria è npm install gsap ) 
gsap.to(cube1.position, {x: 2, delay:1,  duration:1})
gsap.to(cube1.position, {x: 0, delay:2,  duration:1})
//N.B. Gsap ha un "request animation frame" interno, quindi non dobbiamo dire a gsap di aggiornarsi nella funzione animate
// ricorda pero che abbiamo comuqnue bisogno di aggiornare il renderer.render all'interno della funzione animate!


const animate = () =>{

  //1) quindi compariamo il tempo attuale, con quello appena passato, calcolando la differenza 
  //otteniamo il valore deltaTime, che è il valore che useremo nelle nostre animazioni
  //const currentTime = Date.now() 
  
  //1)
  //const deltaTime = currentTime - time
  //time = currentTime
  //cube1.rotation.y +=0.001 * deltaTime
  //cube2.rotation.y +=0.001 * deltaTime
  //cube3.rotation.y +=0.001 * deltaTime
  
  //2) 
  //const elapsedTime = clock.getElapsedTime() // Come possiamo vedere elapsed time funziona come il date.now, solo che invece di essere in millisecondi, il tempo  parte da 0 ed è espresso in secondi
  //console.log(elapsedTime)
  //cube1.rotation.y = elapsedTime // in questo modo possiamo impostare direttamente la rotazione al valore di elapsedTime, e l'animazione sarà omogenea
  //cube2.position.y = Math.sin(elapsedTime) // usando seno e coseno (che ritornano un valore compreso tra 0 e 1) possiamo simulare un animazione
  //cube2.position.x = Math.cos(elapsedTime) // impostando le posizioni di x e y per il cubo2 in questo modo, si ha un effetto di "moto circolare"
  //cube3.rotation.y = elapsedTime
  //camera.position.y = Math.sin(elapsedTime) // In questo modo facciamo ruotare la telecamera intorno all'ggetto che selezioniamo con lookAt
  //camera.position.x = Math.cos(elapsedTime)
  //camera.lookAt(cube3.position)

  
  renderer.render(scene, camera)
}

onMounted(() => {
  // Il renderer renderizza la scena dal punto di vista della telecamera
  // e ne scrive i risultati sul canvas
  // Un canvas è un elemento html in cui abbiamo la possibilità di disegnare cose,
  // e threeJs usa WebGL per disegnare il render all'interno di questo canvas
  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value
  });

 
  renderer.setSize(sizes.width, sizes.height);
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.render(scene, camera);
  // Animaton(2)
  renderer.setAnimationLoop(animate)

});

onUnmounted(() => {
  renderer.setAnimationLoop(null);
});

</script>

<style>
canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style>