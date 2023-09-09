# Cart Plugin

By Alex Timoncini

# Documentation

Welcome to the `Cart Plugin` Documentation! 

Ho creato un breve descrizione per facilitare l’impementazione del carello per DeliverBoo in `vue.js`. 

La tecnologia utilizzata é simile alla session in PHP con leggere differenze,  si chiama LocalStorage su JS.

> 📘 Utilizziamo LocalStorage invece che SessionStorage perchè in questo modo i dati salvati non hanno scadenza.
> 

# <template></template>

Iniziamo Implementando il codice html nel componente in cui ci sarà l’elenco iniziale dei piatti in vendità (es: menù dentro la restaurant.show);

Aggiungiamo un semplice lista chiamata menù che elenca tutti i piatti forniti (nel nostro caso tramite chiamata api);

Inseriamo poi un evento al click per ogni del piatto che vogliamo aggiungere al carello chiamando la funzione `addDishToCart(dish)` e passandogli l’oggetto `dish` ;

```jsx
<template>
	<ul id="menu">
	  <li v-for="dish in dish_list" @click="addDishToCart(dish)">
	    <span>{{ dish.name }}</span>
	    <img :src="dish.photo">
	    <span>{{ dish.price }} €</span>
	  </li>
	</ul>
</template>
```

# <script></script>

## methods : {}

### saveCart()

> Questa funzione ci servirà ogni volta che aggiungiamo, modifichiamo o rimuoviamo oggetti nel carrello per salvare le modifiche dentro la sessione;
> 

Partiamo ora dalla funzione `saveCart()`;

Il localStorage (ed in generale le session in qualsiasi linguaggio) riesce a salvare solamente dati semplici (stringhe);

Per questo motivo giochiamo con il `JSON.stringify` per tradurre l’oggetto in stringa dentro la variabile `parsed` ;

Salviamo poi successivamente il dato dentro il `localStorage` aprendo una nuova chiave chiamata `cart` a cui assegnamo il valore di `parsed` ;

```jsx
methods: {
	    saveCart(){
	      const parsed = JSON.stringify(this.cart_list);
	      localStorage.setItem('cart', parsed);
	    }
}
```

### addDishToCart(dishObj)

> Questa funzione ci servirà per aggiungere un nuovo piatto al carrello o aumentare la quantità di un piatto già presente;
> 

Passiamo ora alla funzione `addDishCart(dishObj)`;

La funzione verrà chiamata quando l’utente cliccherà sull’icona del carrello affianco ad ogni piatto nel menù con l’intento di aggiungerlo al carrello;

Concentriamoci inizialmente sulla semplice funzionalità di aggiunta.

- Aggiungere un nuovo piatto
    
    All’interno dei `data()` oltre all’elenco normale dei piatti, salviamo una variabile `cart_list` con un array vuoto;
    
    La funzione si occuperà semplicemente di aggiungere a `cart_list` l’oggetto passato tramite attributo con un semplice `push()` ;
    
    Chiamerà poi la precreata funzione `saveCart()` per salvare `cart_list`  dentro la sessione;
    
    ```jsx
    data() {
        return {
    	      dish_list: [/**Array di Oggetti contenti  Piatti**/]     
    	      cart_list: [],
        }
     },
    methods: {
    		addDishToCart(dishObj){
    				this.cart_list.push(dishObj);
    				this.saveCart();
    		},
    	  saveCart(){
    	      const parsed = JSON.stringify(this.cart_list);
    	      localStorage.setItem('cart', parsed);
    		}
    }
    ```
    

Andiamo ora a gestire la casistica in cui un andiamo ad aggiungere un piatto già inserito per evitare di duplicare il piatto

- Aggiungere un piatto esistente
    
    Per prima cosa andiamo a verificare con un `if()` se il piatto è già esistente;
    
    Salviamo dentro `cart` l’array presente dentro il localStorage salvato a chiave `cart` ;
    
    Ricordandoci che per salvarlo nel localStorage abbiamo utilizzato il `JSON.stringify` riconvertiamolo ad array di oggetti per poterci iterare tramite `JSON.parse` ;
    
    > **`|| []`**: Questa parte della riga di codice è un operatore logico OR (**`||`**). Se la chiave **`'cart'`** non esiste nel **`localStorage`** o se il valore è `null` (ad esempio, se non è stato ancora impostato), il **`localStorage.getItem('cart')`** restituirà **`null`**. In questo caso, per evitare errori, assegniamo un array vuoto **`[]`** alla variabile **`cart`**. Quindi, se il **`localStorage`** è vuoto o non contiene dati validi, `cart ****`sarà un array vuoto su cui possiamo comunque lavorare.
    > 
    
    Apriamo ora l’`if`  avente come condizione l’esistenza dell’attributo passato `dishObj` dentro l’array salvato in `cart` ;
    
    Usiamo `find()` per iterare dentro `cart`, `find()` ritorna l’oggetto trovato se presento oppure `undefind` ;
    
    Una volta impostato l’if andiamo a gestire entrambi i casi;
    
    Andiamo a creare una nuova chiave dentro l’oggetto `dishObj` chiamata `quantity` che conterrà la quantità di piatti uguali dentro il carrello;
    
    Nel caso in cui il piatto sia nuovo, la creiamo e settiamo a 1 `dishObj.quantity = 1;`;
    
    > Se invece il piatto è già esistente potremmo pensare di fare un semplice `dishObj.quantity++` e salvare, il problema è la modifica dei dati dentro il localStorage che è, impostato il codice in questo modo, impossibile;
    > 
    
    Gestiamo questo caso andando per prima cosa a cercare l’elemento uguale per ottenere la sua `quantity` sempre tramite un `find()` identico a quello dentro l’if e lo salviamo dentro la variabile `dish` ;
    
    Successivamente estraiamo la sua `quantity` e la aumentiamo di 1;
    
    Ora non potendo modificare direttamente il dish andiamo a eliminare tramite `splice()` il vecchio dish e aggiungiamo quello nuovo tramite `push()` per fare ciò otteniamo prima la posizione del dish tramite un semplice `findIndex()` ;
    
    ```jsx
    data() {
        return {
    	      dish_list: [/**Array di Oggetti contenti  Piatti**/]     
    	      cart_list: [],
        }
     },
    methods: {
    		addDishToCart(dishObj){
    				let cart = JSON.parse(localStorage.getItem('cart')) || [];
    				if( (cart.find(item) => item.name === dishObj.name)) !== undefined ){
    						let dish = cart.find((item) => item.name === dishObj.name);
    		        dishObj.quantity = dish.quantity + 1;
    						let dishIndex = cart.findIndex((item) => item.name === dishObj.name);
    		        this.cart_list.splice(dishIndex, 1);
    		        this.cart_list.push(dishObj);
    				else{
    						dishObj.quantity = 1;
    						this.cart_list.push(dishObj);
    				}
    				this.saveCart();
    		},
    	  saveCart(){
    	      const parsed = JSON.stringify(this.cart_list);
    	      localStorage.setItem('cart', parsed);
    		}
    }
    ```
    

### removeDishFromCart(dishIndex)

<aside>
🛒 Questa funzione andrà inserità dentro il component del carrello

</aside>

> Questa funzione ci servirà per eliminare un piatto dal carrello o diminuirne la quantità;
> 

Passiamo ora alla funzione `removeDishFromCart(dishIndex, finalQuantity)`;

La funzione verrà chiamata all’interno del carrello e dovrà essere  passata con due attributi, per prima cosa l’index del piatto dentro il localStorage, facilmente ottenibile dal v-for con cui creerete l’elenco di elementi nel carrello.

> Se la cosa crea problemi (es: non si è sicuri della validità dell0index del v-for perchè gli elementi si spostano al reload), passate semplicemente l’intero oggetto come nel fatto nel menù per la funzione `addDishToCart(dishObj)` e poi cercate tramite un semplce `findIndex()`;
> 

Come secondo attributo la quantità finale che il piatto dovrà avere (magari si può impostare tramite select aventi come opzioni da 0 alla quantità già presente);

Se la quantità passata sarà 0, procederemo con un semplice `splice()` per rimuovere l’elemento;

Altrimenti se si vuole semplicemente ridurre la quantità dobbiamo per prima cosa ottenere e copiare l’oggetto dentro `dishCopy`;

Andiamo poi a modificare la quantità con quella passata nella funzione;

Per ultima cosa eliminiamo il vecchio dish e pushiamo la copia modificata;

Terminiamo la funzione con un semplice `saveCart()`;

```jsx
data() {
    return {
	      dish_list: [/**Array di Oggetti contenti  Piatti**/]     
	      cart_list: [],
    }
 },
methods: {
		removeDishFromCart(dishIndex, finalQuantity){
				if(finalQuantity == 0){
						this.cart_list.splice(dishIndex, 1);
				} else {
						let dishCopy = this.cart_list[dishIndex];
						dishCopy.quantity = finalQuantity;
						this.cart_list.splice(dishIndex, 1);
						this.cart_list.push(dishCopy);
				}
				this.saveCart();
		},
	  saveCart(){
	      const parsed = JSON.stringify(this.cart_list);
	      localStorage.setItem('cart', parsed);
		}
}
```

## mounted()

L’ultima cosa che ci manca da fare è recuperare i dati dentro il localStorage e salvarli dentro la variabile `cart_list` che abbiamo usato fino ad’ora;

Per fare ciò impostiamo un sorta di chiamata Api per gestire anche il caso in cui i dati siano corrotti (il localStorage può essere facilmente manomesso dall’utente);

Se la session con chiave cart esiste, chiamiamo il localStorage:

-se la chiamata va a buon fine salviamo i dati del localStorage dentro la variabile `cart_list` nei data riconvertondoli prima con un parse;

-se la chiamata da errori o i dati sono corrotti, rimuoviamo tutti i dati presenti nella sessione con chiave `cart`;

```jsx
 mounted() {
    if(localStorage.getItem('cart')){
      try {
        this.cart_list = JSON.parse(localStorage.getItem('cart'));
      } catch(e) {
        localStorage.removeItem('cart');
      }
    }
    // localStorage.clear();
  },
```

## Conclusion

Il modo migliore di impostare la cosa è tramite creando uno store.js che conservi cart_list, in modo da non complicarsi con props ed emit varie tra il carrello ed il menù.

Ecco il template finale utilizzano lo store.js.

## Final Example

### Menu’ Component

```jsx
<script>
import { store } from './store'
import Cart from './components/Cart.vue'
export default {
  name: 'Menu',
  data() {
    return {
      dish_list: [/**Array di Oggetti contenti  Piatti**/]     
      store,
    }
  },
  methods: {
		addDishToCart(dishObj){
				let cart = JSON.parse(localStorage.getItem('cart')) || [];
				if( (cart.find(item) => item.name === dishObj.name)) !== undefined ){
						let dish = cart.find((item) => item.name === dishObj.name);
		        dishObj.quantity = dish.quantity + 1;
						let dishIndex = cart.findIndex((item) => item.name === dishObj.name);
		        store.cart_list.splice(dishIndex, 1);
		        store.cart_list.push(dishObj);
				else{
						dishObj.quantity = 1;
						store.cart_list.push(dishObj);
				}
				this.saveCart();
		}<div class="cart">
    <h1>cart</h1>
    <li v-for="(dish, index) in store.cart_list">
      <span>{{ dish.name }} </span>
      <span>{{ dish.price }} € </span>
      <span> {{ dish.quantity }} X</span>
      <button type="button" @click="removeDishFromCart(index, 0)"> Delete</button>
    </li>
  </div><div class="cart">
    <h1>cart</h1>
    <li v-for="(dish, index) in store.cart_list">
      <span>{{ dish.name }} </span>
      <span>{{ dish.price }} € </span>
      <span> {{ dish.quantity }} X</span>
      <button type="button" @click="removeDishFromCart(index, 0)"> Delete</button>
    </li>
  </div>
    saveCart(){
      const parsed = JSON.stringify(this.cart_list);
      localStorage.setItem('cart', parsed);
    }
  },
  mounted() {
    if(localStorage.getItem('cart')){
      try {
        store.cart_list = JSON.parse(localStorage.getItem('cart'));
      } catch(e) {
        localStorage.removeItem('cart');
      }
    }
    // localStorage.clear();
  },
  components: {
    HelloWorld
  }
}
</script>

<template>
<nav>
  <h1>DeliveBoo</h1>
  <Cart />
</nav>
<ul>
  <li v-for="dish in dish_list" @click="addDishToCart(dish)">
    <span>{{ dish.name }}</span>
    <img :src="dish.photo" alt="" style="height: 50px; aspect-ratio: 1/1;">
    <span>{{ dish.price }} €</span>
  </li>
</ul>
</template>
```

### Cart Component

```jsx
<script>
import { store } from './store'
export default {
  name: 'Cart',
  data() {
    return {   
      store,
    }
  },
  methods: {
		removeDishFromCart(dishIndex, finalQuantity){
				if(finalQuantity == 0){
						this.cart_list.splice(dishIndex, 1);
				} else {
						let dishCopy = this.cart_list[dishIndex];
						dishCopy.quantity = finalQuantity;
						this.cart_list.splice(dishIndex, 1);
						this.cart_list.push(dishCopy);
				}
				this.saveCart();
		},
    saveCart(){
      const parsed = JSON.stringify(this.cart_list);
      localStorage.setItem('cart', parsed);
    }
  }
}
</script>

<template>
		<div class="cart">
		    <h1>cart</h1>
		    <li v-for="(dish, index) in store.cart_list">
			      <span>{{ dish.name }} </span>
			      <span>{{ dish.price }} € </span>
			      <span> {{ dish.quantity }} X</span>
			      <button type="button" @click="removeDishFromCart(index, 0)"> Delete</button>
		    </li>
	  </div>
</template>
```

### Store.js

```jsx
import { reactive } from 'vue';

export const store = reactive({
    cart_list:  [],
});
```

Contact me to learn more!

---

<aside>
🛒 **👋 Thank you for your interest in this template!**
Any questions or thoughts? You can contact me at timonciniDev**@gmail.com.**

</aside>

Table of Contents

---