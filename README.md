# Programacion asincrona
La programacion hoy en dia es mas asincrona y debe ser pensado de manera **Offline first**
## Hilo 
Un programa es un proceso dividido en varios hilos.

![Hilo](./img/hilo.png)<br>
El blocked es un estado que se activa cuando esta esperando unos datos de tal manera que quedaria asi
![alt text](img/unsolohilo.png)<br>
para solucionar que se pare los programas se hacen **multihilos**
![alt text](img/multihilo.png)<br>
pero esto es muy complejo y algunos procesadores no pueden hacerlo 
* Callback: La idea es pasar una funcion a otra como parametro <br>
  ```kotlin
  fun postItem(item: Item) { 
    preparePostAsync { token ->
    submitPostAsync(token, item) { post -> 
    processPost(post)
            }
        }
    }

  ```
* Promesas: encapsula el callback y el manejo de errores es dificil<br>
  ```kotlin
  
    fun preparePostAsync(): Promise<Token> {
        // makes request and returns a promise that is completed later
        return promise
    }

  ```
  ```kotlin
  fun postItem(item: Item) { 
    preparePostAsync()
		.thenCompose { token -> submitPostAsync(token, item) }
		.thenAccept { post -> processPost(post) }
		…
    }
  ```
* Corutinas: es lo que se utiliza en kotlin y es la mas correcta
  ```kotlin
  suspend fun postItem(item: Item) {
        val token = preparePost()
        val post = submitPost(token, item)
        processPost(post)
    }
  ```
## Corrutinas
### Estados 
estos son los estados tiene una corrutina 
![alt text](./img/estadoscorutina.png)
### Scopes
Las funciones suspendidas **solo se pueden ejecutar desde una corrutina** (ejemplo hecho con IA sujeto a fallos)
```kotlin
import kotlinx.coroutines.*

// 1. Definición de una función suspendida
suspend fun realizarTarea(nombre: String) {
    println("[$nombre] Iniciando tarea en el hilo: ${Thread.currentThread().name}")
    // Simula una operación de larga duración sin bloquear el hilo
    delay(1000L) 
    println("[$nombre] Tarea completada. Hilo actual: ${Thread.currentThread().name}")
}

fun main() {
    println("[Main] Iniciando la aplicación...")
    
    // ERROR DE COMPILACIÓN: 
    // Las funciones 'suspend' solo pueden ser llamadas desde una corrutina 
    // (o desde otra función 'suspend').
    // realizarTarea("Tarea 1") // Si descomentas esta línea, el código no compilará.
    
    // SOLUCIÓN: Se debe ejecutar dentro de un ámbito de corrutina.
    runBlocking {
        println("[runBlocking] Corrutina lanzada en el hilo: ${Thread.currentThread().name}")
        realizarTarea("Tarea Corrutina 1")
        realizarTarea("Tarea Corrutina 2")
    }

    println("[Main] Aplicación finalizada.")
}
```