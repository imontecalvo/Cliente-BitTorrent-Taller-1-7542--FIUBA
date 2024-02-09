# 22C1 - Cliente BitTorrent - PROYECTO TALLER I
## Objetivo
El objetivo del proyecto es implementar un Cliente BitTorrent con funcionalidades acotadas, que se detallan en el enunciado. El objetivo secundario del proyecto consiste en el desarrollo de un proyecto real de software de mediana envergadura aplicando buenas prácticas de desarrollo de software, incluyendo entregas y revisiones usando un sistema de control de versiones. Se presente emular, en la medida de lo posible, el proceso de desarrollo de la Industria de Software.

## DEMO
https://github.com/imontecalvo/Taller-1-7542--FIUBA/assets/82344492/02a92877-19eb-438e-b3e4-92675a9f1ee3

## Dependencias
En caso de error al ejecutar en Ubuntu, asegurarse de tener instalado: `libssl-dev` y `libgtk-3-dev`
```
sudo apt install libssl-dev
sudo apt install libgtk-3-dev
```

## Ejecución
Se debe ejecutar el siguiente comando, estando en `/bittorrent_client`:
```
cargo run <ruta .torrent / directorio> <ruta archivo configuración>
```
### Parámetros
* Ruta .torrent / directorio: Admite una ruta específica a un archivo .torrent como así también a un directorio. En el segundo caso, recorrerá todo el directorio obteniendo los archivos .torrents localizados para posteriormente descargarlos
* Ruta archivo configuración: Se puede usar un archivo personalizado o el archivo por defecto localizado en `/bittorrent_client/settings.txt`

## Funcionamiento
### Flujo general
1. Parseo de la configuración.
2. Escaneo de torrents (parsear torrents y detectar si ya hay piezas descargadas pudiendo reanudar descargas previas)
3. Ejecución del servidor
4. Ejecución de el/los clientes (se intancia uno por cada torrent, como máximo pueden haber tres clientes en paralelo).
5. Logging de eventos
6. Interfaz gráfica

### Diagramas
#### Interacción general
<p align="center">
  <img src=https://github.com/imontecalvo/Taller-1-7542--FIUBA/assets/82344492/d05ebeb9-968e-41f9-8873-d200f823fc4e" />
</p>

#### Threads
<p align="center">
  <img src="https://github.com/imontecalvo/Taller-1-7542--FIUBA/assets/82344492/dd4cab3f-ea77-446f-9621-bd20fe6f81cd" />
</p>

### Descarga de piezas
La descarga de piezas sucede de forma simultánea (concurrente) en los threads de cada `PeerConnection`.
Estos threads comparten la** cola con las piezas restantes**, por lo que pedirán el lock e intentarán retirar una pieza de la misma.

Puede darse la situación de que un thread intenta tomar una pieza de la cola y esta se encuentra vacía. Que la cola se encuentre vacía no es indicador de que todas las piezas hayan sido descargadas, dado que puede ocurrir que algún thread que tomó una de las piezas para descargar no pueda hacerlo. 

Debido a eso es necesario un mecanismo de control para saber que piezas fueron correctamente descargadas. Para ello se utiliza el mismo bitfield representativo de las piezas del torrent, donde se distinguen las descargadas de las pendientes. En caso de que la cola esté vacía:
* Se chequea si la descarga finalizó fijándose si el bitfield de piezas descargadas posee todas las piezas en 1. En este caso, la conexión se termina.
* Si el peer no posee ninguna pieza de las restantes, también finalizará la conexión.
* En caso contrario, intentará nuevamente retirar una pieza de la cola llamando previamente a ”yield now”.

Si la cola no estaba vacía, se intenta descargar la pieza obtenida siguiendo el protocolo de BitTorrent. 
En caso de algún error, ya sea que la pieza es inválida, el peer nos bloqueó, no tiene la pieza o simplemente no nos contesta, la pieza extraída se devuelve a la cola de piezas pendientes. 
Por otro lado, si la pieza se descargó con éxito, la misma se escribe en disco y se notifica al cliente para que actualice el bitfield

<p align="center">
  <img src="https://github.com/imontecalvo/Taller-1-7542--FIUBA/assets/82344492/c0024799-d38b-4769-a116-fb898cb3721f" />
</p>


## Integrantes

* Luis Anibal Arillo
* Santiago Marczewski
* Ignacio Montecalvo
* Tomas Sabao
