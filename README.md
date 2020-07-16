# TSSOO-taller02

##### Autor: Ignacio Alvarado Torreblanca - Correo: ignacio.alvarado@alumnos.uv.cl

## 1. Diseño de Solución

En la siguiente figura se puede observar un diagrama de secuencia que explica el funcionamiento de los procesos que se ejecutarán para llegar a la solución del problema.
![Ialvarado](http://imgfz.com/i/3h680Hk.png)

Para resolver el problema será necesario conocer el lenguaje de programación c ++ junto a las librerías que se encarguen de la implementación threads, como también los comandos para medir los tiempos de ejecución de las ejecuciones del programa los cuales serán necesarios para el posterior análisis de estos.

Una vez ejecutado el programa este debe comenzar a crear el arreglo vacío, con la cantidad de hilos y el total de elementos agregados en los parámetros al momento de la ejecución. Para crear los hilos por cada módulo, se van a añadir, con un ciclo For, los distintos segmentos de los hilos para luego unirlos. El llenado se debe hacer mediante el encargado de generar números aleatorios en base a los límites propuestos a la hora de ejecutar el programa. Tras realizar el llenado del arreglo, se debe ejecutar el módulo de suma del arreglo compuesto de la otra mitad de hilos instanciados.
## 2. Estructura del Código

### 2.1 Parametros del script
Al usar el script desarrollado en bash, se pasan por parámetro distintos elementos que se necesitan para la ejecución del programa, para ingresar el tamaño del arreglo se ingresa después de "-N", para el numero de threads se ingresa después de  "-t", para el limite inferior del rango aleatorio se ingresa después de "-l", para el limite superior del rango aleatorio se ingresa después de "-L" y opcionalmente tenemos  "-h" para mostrar la forma de uso. A continuación se puede ver la variable que contiene la forma de uso del programa: 
```
const std::string descripcion = "Párametros:\n"
"\t-N : tamaño del arreglo.\n"
"\t-t : número de threads.\n"
"\t-l : limite inferior rango aleatorio.\n"
"\t-L : límite superior rango aleatorio.\n"
"\t[-h] : muestra la ayuda de uso y termina. \n";
```
En el archivo "checkArgs.hpp" están condicionadas todos los parámetros y condiciones de uso. En la siguientes lineas de código se puede ver como se almacenan los parámetros nombrados anteriormente :
```
checkArgs::checkArgs(int _argc , char **_argv){
parametros.tamProblema = 0;
parametros.numThreads = 0;
parametros.l_inferior = 0;
parametros.l_superior = 0;
argc = _argc;
argv = _argv;
}
```
### 2.2 Modulo de llenado en paralelo

 Para llenar el arreglo con números randomicos en los limites establecidos en los parámetros de entrada del programa, tenemos la siguiente función:

```
void fillArray(uint32_t l_superior, uint32_t l_inferior, size_t beginIndex,
size_t endIndex)
{
std::random_device device;
std::mt19937 rng(device());
std::uniform_int_distribution<> unif(l_inferior, l_superior);
for (size_t i = beginIndex; i < endIndex; ++i){
	paralelo[i] = unif(rng);
	}
}
```
Donde fillArray es el nombre de la función que será implementada en la función main, sus parámetros son los limites para generar el numero random y los limites que deben ser recorridos para poner los hilos uno tras otro (`beginIndex`y `endIndex`).
```
paralelo = new uint64_t[totalElementos];
for (size_t i = 0; i < numThreads; ++i)
{
	threads.push_back(new std::thread(fillArray,
	l_superior, l_inferior,
	i * (totalElementos) / numThreads,
	(i + 1) * (totalElementos) / numThreads));
	}
auto start = std::chrono::system_clock::now();
for (auto &ths : threads)
	{
	ths->join();
	}
auto end = std::chrono::system_clock::now();
std::chrono::duration<float, std::milli> duration = end - start;
auto tiempo_arreglo = duration.count();
```
El fragmento de código mostrado anteriormente corresponde al uso de hilos instancias en la clase main, `paralelo` es el arreglo que almacena los números randoms y que será ocupado con los hilos, luego tenemos un ciclo for que recorre la cantidad de hilos que están puestos como parámetro de entrada `(numThreads)`, y dentro de ese ciclo se tiene a la función push_back, con la cual se añaden elementos al final del vector, en este caso el vector es `threads` que esta instancia en la clase main y corresponde al vector que almacena los hilos, y con push_back se llama a la función fillArray donde `i * (totalElementos) / numThreads` es la parte inicial del hilo que se recorre en la función, esta corresponde a `beginIndex` en la función fillArray y `(i + 1) * (totalElementos) / numThreads)`, que corresponde a `endIndex` en la función fillArray,  es el inicio del siguiente hilo.
Por ultimo, tenemos un arreglo que une a los hilos con `ths->join();`. todo esto siendo cronometrado con la función `chrono` y almacenado en una variable `tiempo_arreglo` para ser mostrada por pantalla.



### 2.3 Modulo de suma en paralelo.
Al igual que en el modulo anterior, se tiene una función que luego es ocupada en a función main, la cual suma los elementos del arreglo anterior:
```
void sum_parcial(size_t pos,
size_t beginIndex,
size_t endIndex)
{
for (size_t i = beginIndex; i < endIndex; ++i)
	{
	sum_parciales[pos] += paralelo[i];
	}
}
```
Donde al igual que en el modulo anterior, se pasa por parámetro a los limites de los hilos que serán recorridos dentro de la función con el ciclo for, también tenemos un nuevo elemento `pos` que corresponde a la posición del arreglo `sum_parciales` que almacenara la suma de los elementos.

```
sum_parciales = new uint64_t[totalElementos];
for (size_t i = 0; i < numThreads; ++i)
	{
	threads2.push_back(new std::thread(sum_parcial, i,
	i * (totalElementos) / numThreads,
	(i + 1) * (totalElementos) / numThreads));
	}
start = std::chrono::system_clock::now();
for (auto &th : threads2)
{
	th->join();
}

for (size_t i = 0; i < numThreads; ++i)
{
band += sum_parciales[i];
}
end = std::chrono::system_clock::now();
std::chrono::duration<float, std::milli> duration6 = end - start;
auto tiempo_suma = duration6.count();
```
En el código mostrado, al igual que en el modulo de llenado en paralelo, se crea instancia un arreglo llamado `sum_parciales` que es el que almacenara los valores que se sumaran en la función. Luego tenemos el ciclo for con el push_back que hará lo mismo que el modulo anterior, con la diferencia que tenemos un vector distinto de hilos llamado `threads2` el cual es instancias en la misma función main, y como parámetro dentro del push_back pasamos los limites y la posición `i` que será necesaria para sumar los elementos del arreglo creado en el modulo anterior. Por ultimo, tenemos un ciclo for que une los hilos y otro ciclo for que pondrá todas las sumas realizadas por los hilos en una sola variable llamada `band`, todo este procesos de los ciclo for esta siendo cronometrado para su posterior análisis.
