# Laboratorio 9 Escalamiento en Azure con Máquinas Virtuales

### Escuela Colombiana de Ingeniería Julio Garavito
### Arquitecturas de Software - ARSW

**Integrantes**
- Mariana Malagón
- Paula Lozano

**Repositorio:** https://github.com/marianamalagon11/ARSW_LOAD-BALANCING_AZURE

> El enunciado del laboratorio está en [README.md](README.md).

---

## Tabla de contenido

- [Parte 1: Escalabilidad vertical](#parte-1-escalabilidad-vertical)
  - [Checklist](#checklist-parte-1)
  - [Evidencias](#evidencias-parte-1)
  - [Preguntas](#preguntas-parte-1)
- [Parte 2: Escalabilidad horizontal](#parte-2-escalabilidad-horizontal)
  - [Desviaciones respecto a la guía](#desviaciones-respecto-a-la-guía)
  - [Evidencias](#evidencias-parte-2)
  - [Preguntas](#preguntas-parte-2)
- [Diagrama de despliegue](#diagrama-de-despliegue)
- [Conclusiones](#conclusiones)
- [Limpieza de recursos](#limpieza-de-recursos)

---

# Parte 1: Escalabilidad vertical

## Checklist: Parte 1

**Creación de la VM**
- [ ] Crear VM: Resource Group `SCALABILITY_LAB`, nombre `VERTICAL-SCALABILITY`, imagen `Ubuntu Server`, tamaño `Standard B1ls`, usuario `scalability_lab`, autenticación **SSH public key**
- [ ] Conectarse por SSH: `ssh scalability_lab@<IP_PUBLICA>`
- [ ] Instalar Node.js con NVM
- [ ] Clonar el repositorio y ejecutar `npm install` dentro de `FibonacciApp`
- [ ] Ejecutar la aplicación: `node FibonacciApp.js`
- [ ] Crear la **Inbound port rule** para el puerto `3000` en la sección *Networking*
- [ ] Verificar `http://<IP_PUBLICA>:3000/fibonacci/6` debe responder `The answer is 8`

**Mediciones con tamaño B1ls**
- [ ] Documentar tiempos de respuesta desde la consola del browser para: 1000000, 1010000, 1020000, 1030000, 1040000, 1050000, 1060000, 1070000, 1080000, 1090000
- [ ] Capturar la gráfica de consumo de CPU de la VM en Azure (tarda ~5 min en aparecer)
- [ ] Instalar newman en una máquina **distinta** a la VM: `npm install newman -g`
- [ ] En `FibonacciApp/postman/part1/[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambiar `VM1` por la IP de la VM
- [ ] Ejecutar la prueba de carga con 2 procesos en paralelo y capturar el resumen

**Escalamiento vertical**
- [ ] Cambiar el tamaño de la VM a `B2ms` (sección *Size*)
- [ ] Repetir tiempos de respuesta, gráfica de CPU y prueba de newman
- [ ] Evaluar si se cumple el escenario de calidad
- [ ] Ejecutar newman con **4 procesos en paralelo** (pregunta 11)
- [ ] **Devolver la VM al tamaño inicial `B1ls`** para evitar cobros

Comando de carga, desde `FibonacciApp/postman/part1`:

```bash
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e "[ARSW_LOAD-BALANCING_AZURE].postman_environment.json" -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e "[ARSW_LOAD-BALANCING_AZURE].postman_environment.json" -n 10
```

> En **PowerShell** el `&` no lanza procesos en segundo plano. Usar Git Bash / WSL, o una ventana por ejecución:
>
> ```powershell
> Start-Process newman -ArgumentList 'run','ARSW_LOAD-BALANCING_AZURE.postman_collection.json','-e','[ARSW_LOAD-BALANCING_AZURE].postman_environment.json','-n','10'
> ```

## Evidencias: Parte 1

### Creación de la VM

### Aplicación corriendo

### Tiempos de respuesta con B1ls

### Consumo de CPU con B1ls

### Prueba de newman con B1ls

### Cambio de tamaño a B2ms

### Tiempos de respuesta con B2ms

### Consumo de CPU con B2ms

### Prueba de newman con B2ms

### Prueba de newman con 4 procesos en paralelo

## Preguntas: Parte 1

**1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?**

**2. ¿Brevemente describa para qué sirve cada recurso?**

**3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?**

**4. Adjunte tabla de tiempos e interprete por qué la función tarda tanto tiempo.**

**5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.**

**6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete los tiempos de ejecución de cada petición y, si hubo fallos, documéntelos y explíquelos.**

**7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?**

**8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario? ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?**

**9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?**

**10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?**

**11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?**

---

# Parte 2: Escalabilidad horizontal

**Recursos desplegados**

| Recurso | Nombre |
|---|---|
| Resource Group | `SCALABILITY_LAB` |
| Load Balancer | `SCALABILITY_LAB_LOAD_BALANCER` |
| IP pública del LB | `SCALABILITY_LAB_LOAD_BALANCER_IP` |
| Backend Pool | `SCALABILITY_LAB_BACKEND_POOL` |
| Health Probe | `SCALABILITY_LAB_HEALTH_PROBE` |
| Load Balancing Rule | `SCALABILITY_LAB_LOAD_BALANCING_RULE` |
| Virtual Network | `SCALABILITY_LAB_VIRTUAL_NETWORK` |
| Subnet | `SCALABILITY_LAB_SUBNET` |
| Network Security Group | `LOAD_BALANCER_NSG` |
| VMs | `VM1`, `VM2`, `VM3` (y `VM4` en el punto 3) |

> Todos los recursos deben quedar en la **misma región**, de lo contrario el LB, la VNet y las VMs no se pueden asociar entre sí.
## Desviaciones respecto a la guía

El enunciado es de 2019 y la suscripción usada es *Azure for Students*, lo que obligó a cuatro ajustes:

| Aspecto | Guía | Implementado | Razón |
|---|---|---|---|
| Región | North Europe | **Canada Central** | La política `Allowed resource deployment regions` limita la suscripción a 5 regiones, y solo Canada Central y Chile Central ofrecen `B1ls` con 3 zonas de disponibilidad. |
| IP pública por VM | Una por VM | **Ninguna** | La suscripción permite máximo 3 IPs públicas por región y el balanceador usa una. Las VMs salen a internet por el SNAT del balanceador. |
| Imagen del SO | Ubuntu 18.04 | **Ubuntu 22.04** | La 18.04 llegó a fin de vida y ya no está en el Marketplace. |
| Gestor del proceso | `forever` | **`systemd`** | La instalación se automatizó de forma no interactiva. `systemd` cumple la misma función y además reinicia el proceso si falla. |

Ninguna altera la arquitectura evaluada: sigue siendo un balanceador público que distribuye tráfico entre nodos idénticos repartidos en zonas de disponibilidad distintas.

## Evidencias: Parte 2

### Grupo de recursos

![Grupo de recursos SCALABILITY_LAB en Canada Central](images/entrega/part2/00-resource-group.png)

Contenedor de toda la solución. Agrupa 13 recursos en **Canada Central**: el balanceador y su IP pública, la red virtual, el grupo de seguridad de red, y por cada VM su máquina, su interfaz de red y su disco. Tenerlos en un solo grupo permite eliminarlos todos con una sola operación al terminar.

### Virtual Network

![Red virtual con espacio de direcciones 10.1.0.0/16](images/entrega/part2/01-virtual-network.png)

Red privada aislada donde viven los nodos. El **espacio de direcciones `10.1.0.0/16`** reserva 65.536 direcciones privadas. Es la frontera de red del sistema: lo que está dentro se comunica por IP privada sin salir a internet.

### Subnet

![Subred SCALABILITY_LAB_SUBNET con rango 10.1.0.0/24](images/entrega/part2/02-subnet.png)

Segmento de la red virtual con rango **`10.1.0.0/24`**, es decir 256 direcciones de las cuales Azure reserva 5. De aquí salieron las IPs privadas de las VMs: `10.1.0.4` a `10.1.0.7`. En la captura se ve marcada la opción de **subred privada**, que elimina el acceso de salida predeterminado; no afectó porque la salida se resuelve por el balanceador.

### Load Balancer

![Balanceador Standard con IP de frontend 20.48.168.44](images/entrega/part2/03-load-balancer.png)

El punto de entrada único del sistema. Lo importante de esta pantalla: **SKU Standard** (requisito para usar zonas de disponibilidad), tipo **público**, y **dirección IP de frontend `20.48.168.44`**, que es la única IP que conoce el cliente. También se ven ya enlazados el backend pool, la regla y el sondeo.

### Backend Pool

![Backend pool con VM1, VM2 y VM3 en zonas 1, 2 y 3](images/entrega/part2/04-backend-pool.png)

Conjunto de nodos que atienden el tráfico. Se ven las tres VMs con su **IP privada** y su **zona de disponibilidad: 1, 2 y 3**. Este es el recurso que hace posible la elasticidad: agregar o quitar nodos aquí no obliga a cambiar la IP pública ni la regla de balanceo.

### Health Probe

![Health probe TCP sobre el puerto 3000 cada 5 segundos](images/entrega/part2/05-health-probe.png)

Comprueba la salud de cada nodo. Apunta al **puerto 3000**, que es donde escucha la aplicación, **no al 80** del frontend. Con intervalo de **5 segundos** y umbral de **2 fallos**, un nodo caído sale de rotación en unos 10 segundos.

### Load Balancing Rule

![Regla de balanceo TCP/80 hacia TCP/3000](images/entrega/part2/06-load-balancing-rule.png)

La pieza que une todo: traduce **TCP/80 a TCP/3000**, asocia el backend pool con el sondeo de estado, y es lo que permite consumir el servicio como `http://20.48.168.44/fibonacci/6` aunque la aplicación viva internamente en el 3000.

### Network Security Group

![NSG con las reglas de entrada SSH 22 y Port_3000](images/entrega/part2/07-network-security-group.png)

Firewall aplicado a las interfaces de red. Las reglas propias son **1000 (SSH, puerto 22)** y **1010 (Port_3000)**. Debajo se ven las reglas por defecto de Azure, entre ellas **`AllowAzureLoadBalancerInBound` (65001)**, que es la que permite el paso de los health probes, y **`DenyAllInBound` (65500)**, que bloquea todo lo demás. Sin la regla del 3000, el sondeo fallaría y el balanceador vaciaría el pool completo.

### VM1 (zona 1)

![VM1 en Canada Central zona 1, Standard B1ls](images/entrega/part2/08-vm1-zona1.png)

Primer nodo. Los datos relevantes: **Canada Central (Zona 1)**, tamaño **Standard B1ls** con 1 vCPU y 0.5 GiB de RAM, IP privada `10.1.0.4` y **sin IP pública**. El único vCPU es la razón por la que una sola petición satura la máquina.

### VM2 (zona 2)

![VM2 en Canada Central zona 2, Standard B1ls](images/entrega/part2/09-vm2-zona2.png)

Segundo nodo, idéntico al anterior salvo por la **zona 2** y la IP `10.1.0.5`. Estar en otra zona significa otro centro de datos, con energía y red independientes.

### VM3 (zona 3)

![VM3 en Canada Central zona 3, Standard B1ls](images/entrega/part2/10-vm3-zona3.png)

Tercer nodo, en **zona 3** con IP `10.1.0.6`. Con los tres repartidos en zonas distintas, la caída de un centro de datos deja el sistema operando al 66 % en lugar de tumbarlo.

### Aplicación instalada en las VMs

La instalación se automatizó con `az vm run-command invoke`, que ejecuta un script dentro de la VM sin necesidad de abrir una sesión SSH. El script instala Node.js 20, clona el repositorio, instala dependencias, registra el servicio `fibonacci.service` en systemd y verifica el resultado.

Salida de la verificación en cada nodo:

```
VM1 -> servicio: active | puerto 3000: escuchando | respuesta: The answer is 8
VM2 -> servicio: active | puerto 3000: escuchando | respuesta: The answer is 8
VM3 -> servicio: active | puerto 3000: escuchando | respuesta: The answer is 8
VM4 -> servicio: active | puerto 3000: escuchando | respuesta: The answer is 8
```

Los cuatro nodos quedaron con Node.js `v20.20.2`, el servicio activo y habilitado al arranque, y respondiendo correctamente en `localhost:3000`.

### Verificación funcional del balanceador

![Respuesta del balanceador en /fibonacci/6](images/entrega/part2/12-balanceador-funcionando.png)

Prueba de extremo a extremo. El navegador consulta `http://20.48.168.44/fibonacci/6` y recibe `The answer is 8`. Esa única petición recorre toda la arquitectura: entra por el **puerto 80** a la IP pública del balanceador, la regla la traduce al **puerto 3000**, el hash de 5-tupla escoge uno de los nodos sanos del backend pool, y la aplicación responde desde una IP privada que no es alcanzable desde internet.

Que funcione confirma simultáneamente que el balanceador, la regla, el health probe, el NSG, la red virtual y la aplicación están bien configurados. Cualquier pieza mal puesta habría dado un error aquí.

Se verificó también el reparto: con **18 peticiones usando conexión nueva cada una**, el tráfico se distribuyó en **6, 6 y 6** entre VM1, VM2 y VM3.

### Prueba de newman con 2 procesos en paralelo

Comando ejecutado desde `FibonacciApp/postman/part2`, con `loadbalancer = 20.48.168.44`:

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e "[ARSW_LOAD-BALANCING_AZURE].postman_environment.json" -n 10   (x2 en paralelo)
```

| Métrica | Ejecución 1 | Ejecución 2 |
|---|---|---|
| Iteraciones ejecutadas | 10 | 10 |
| Peticiones | 10 | 10 |
| **Fallidas** | **0** | **0** |
| Duración total | 2m 4.4s | 2m 7.9s |
| Tiempo de respuesta promedio | 12.3s | 12.7s |
| Mínimo | 12.2s | 12.2s |
| Máximo | 12.4s | 13.5s |
| Desviación estándar | 54 ms | 458 ms |
| Datos recibidos | 2.09 MB | 2.09 MB |

**Tasa de éxito: 20 de 20 peticiones, 100%.**

Resumen de newman en consola, correspondiente a una ejecución individual contra el balanceador:

![Resumen de la ejecución de newman](images/entrega/part2/11-newman-resumen-consola.png)

Se leen las tres cifras que importan: **10 iteraciones ejecutadas y 0 fallidas**, duración total de la corrida y tiempo de respuesta promedio con su mínimo, máximo y desviación estándar. La columna `failed` en cero en todas las filas es la evidencia directa de que el sistema respondió a la totalidad de las peticiones.

#### Distribución real de la carga

Contando en los logs de cada VM (`journalctl -u fibonacci | grep 'calculate 1000000'`) cuántas peticiones pesadas atendió cada nodo:

| VM | Zona | Peticiones `n=1000000` atendidas |
|---|---|---|
| VM1 | 1 | 10 |
| VM2 | 2 | 1 |
| VM3 | 3 | 10 |

Cada proceso de newman quedó fijado a **una sola VM** durante sus 10 iteraciones. Esto no es un defecto de la configuración: Azure Load Balancer distribuye mediante un **hash de 5-tupla** (IP origen, puerto origen, IP destino, puerto destino, protocolo) que se evalúa **por conexión TCP**, no por petición. Como newman mantiene la conexión HTTP viva (keep-alive) durante toda su ejecución, las 10 peticiones viajan por el mismo socket y aterrizan siempre en el mismo backend.

La consecuencia práctica es que **el número de nodos que se ejercitan está acotado por el número de conexiones TCP del cliente**, no por el tamaño del backend pool. Con 2 procesos solo se usaron 2 de las 3 VMs.

Esto se verificó por contraste: al lanzar 18 peticiones con `--no-keepalive`, es decir, abriendo una conexión nueva cada vez, el reparto fue **exactamente 6, 6 y 6** entre VM1, VM2 y VM3.

| Escenario | VM1 | VM2 | VM3 |
|---|---|---|---|
| 18 peticiones, conexión nueva cada una | 6 | 6 | 6 |
| 15 peticiones reutilizando conexión | 0 | 0 | 15 |

### Consumo de CPU de las VMs

Métrica *Percentage CPU*, agregación máximo, muestreo por minuto, durante la prueba de carga con 2 procesos en paralelo.

| VM | Zona | Pico de CPU | Peticiones atendidas |
|---|---|---|---|
| VM1 | 1 | **95.70 %** | 10 |
| VM2 | 2 | 1.1 % durante la prueba | 1 |
| VM3 | 3 | **95.06 %** | 10 |

Serie por minuto durante la ventana de la prueba:

```
VM1:  0.3  1.0  0.3  95.7  91.7
VM2:  1.1  0.6  1.0   0.4   0.5
VM3:  0.7 29.5 95.1  95.0   0.4
```

![CPU máxima de VM1 durante la prueba de carga](images/entrega/part2/13-cpu-vm1.png)

![CPU máxima de VM2 durante la prueba de carga](images/entrega/part2/14-cpu-vm2.png)

![CPU máxima de VM3 durante la prueba de carga](images/entrega/part2/15-cpu-vm3.png)

Las gráficas abarcan un rango mayor que la prueba, por lo que el valor de VM2 (66.34 %) corresponde a una medición manual aislada previa y no al periodo de la prueba de newman, durante el cual permaneció por debajo del 1.1 %.

#### Créditos de ráfaga

`Standard_B1ls` es una VM **burstable**: dispone de un rendimiento base bajo y acumula créditos de CPU mientras está ociosa, que gasta al superar esa base. Si los créditos se agotan, Azure estrangula la máquina a su rendimiento base.

![Créditos de CPU restantes en VM1](images/entrega/part2/16-cpu-credits-vm1.png)

Durante la prueba los créditos de VM1 se mantuvieron alrededor de **30**, con un descenso leve y recuperación posterior. La conclusión es que **la prueba fue demasiado corta para agotarlos**, así que los picos del 95 % reflejan saturación real del núcleo y no degradación por falta de créditos.

Esto importa para interpretar bien los resultados: bajo una carga **sostenida** en el tiempo, y no en ráfagas de dos minutos, estos nodos añadirían un segundo modo de falla, porque al vaciarse los créditos el rendimiento caería al nivel base y los tiempos de respuesta se dispararían aún más.

#### Evaluación del escenario de calidad

El escenario exige dos cosas a la vez: que **todas las peticiones sean respondidas** y que el **consumo de CPU no supere el 70 %**.

| Criterio | Resultado | ¿Se cumple? |
|---|---|---|
| Todas las peticiones respondidas | 20 de 20, 0 fallos | **Sí** |
| CPU del sistema por debajo del 70 % | Picos de 95.7 % y 95.1 % | **No** |

El escenario **se cumple parcialmente**. La explicación está en la combinación de tres factores:

1. **La aplicación es CPU-bound y monohilo.** Node.js procesa una petición por vez en el event loop, y el cálculo de Fibonacci con `big-integer` consume el núcleo completo mientras dura.
2. **`Standard_B1ls` tiene un solo vCPU.** Basta una petición concurrente para saturarlo al 100 %. No hay margen para quedarse por debajo del 70 % mientras se atiende tráfico pesado.
3. **La carga no se repartió entre los 3 nodos**, sino entre 2, por el hash de 5-tupla explicado arriba. VM2 permaneció ociosa al 0.4 % mientras VM1 y VM3 estaban al 95 %.

Lo relevante es **qué sí resolvió el escalamiento horizontal**: la disponibilidad. Aunque cada nodo individual se saturó, **ninguna petición falló ni se perdió**, porque el trabajo se repartió entre máquinas independientes en lugar de acumularse en una sola. Con escalamiento vertical, todas las peticiones habrían competido por el mismo núcleo.

Para cumplir también el criterio de CPU harían falta, o más nodos de los que hay peticiones concurrentes, o un tamaño de VM con varios núcleos, o reescribir la función de Fibonacci con un algoritmo eficiente, que es la causa raíz real del problema.

### Informe comparativo con la Parte 1

### VM4 creada

Se agregó un cuarto nodo, `VM4`, en la zona de disponibilidad 1, con IP privada `10.1.0.7` y el mismo tamaño `Standard_B1ls`.

El punto a destacar es que **la incorporación se hizo en caliente**: se creó la interfaz de red ya asociada al `SCALABILITY_LAB_BACKEND_POOL`, y en cuanto el health probe detectó el puerto 3000 escuchando, el balanceador empezó a enviarle tráfico. **No hubo que detener el servicio, ni cambiar la IP `20.48.168.44`, ni modificar la regla de balanceo, ni tocar nada del lado del cliente.** Esa es, en la práctica, la propiedad que define al escalamiento horizontal.

![VM4 en Canada Central zona 1](images/entrega/part2/17-vm4-zona1.png)

Backend pool con los cuatro nodos registrados y en ejecución:

![Backend pool con VM1, VM2, VM3 y VM4](images/entrega/part2/18-backend-pool-4vms.png)

### Prueba de newman con 4 procesos en paralelo

Cuatro procesos simultáneos, 10 iteraciones cada uno, contra los 4 nodos.

| Métrica | Ejec. 1 | Ejec. 2 | Ejec. 3 | Ejec. 4 |
|---|---|---|---|---|
| Iteraciones | 10 | 10 | 10 | 10 |
| **Fallidas** | **0** | **0** | **0** | **0** |
| Duración total | 2m 45.8s | 2m 25.9s | 2m 4.5s | 2m 3.3s |
| Tiempo promedio | 16.4s | 14.5s | 12.3s | 12.2s |
| Mínimo | 15.9s | 14.4s | 12.2s | 12.1s |
| Máximo | 18.0s | 14.7s | 12.6s | 12.5s |
| Desviación estándar | 563 ms | 98 ms | 89 ms | 117 ms |

**Tasa de éxito: 40 de 40 peticiones, 100 %.**

#### Distribución de la carga

| VM | Zona | Peticiones atendidas |
|---|---|---|
| VM1 | 1 | 10 |
| VM2 | 2 | 10 |
| VM3 | 3 | 10 |
| VM4 | 1 | 10 |

Esta vez el reparto fue **perfecto**: cada proceso de newman estableció su propia conexión TCP y el hash de 5-tupla la asignó a un nodo distinto. Con 4 conexiones y 4 backends se ejercitó el 100 % del pool, frente al 66 % de la prueba anterior, donde 2 conexiones solo alcanzaron 2 de 3 nodos.

Esto confirma lo analizado en la pregunta 4: **el paralelismo efectivo lo determina el número de conexiones del cliente**, no el tamaño del backend pool.

#### Comparación entre las dos pruebas

| | Prueba con 2 procesos | Prueba con 4 procesos |
|---|---|---|
| Nodos disponibles | 3 | 4 |
| Peticiones totales | 20 | **40** |
| Peticiones fallidas | 0 | **0** |
| Tasa de éxito | 100 % | **100 %** |
| Tiempo de respuesta | 12.3s y 12.7s | 12.2s a 16.4s |
| Nodos efectivamente usados | 2 de 3 (66 %) | **4 de 4 (100 %)** |
| Pico de CPU | 95.7 % y 95.1 % | 95.9 %, 90.2 %, 96.0 %, 96.2 % |

**Al duplicar la carga, la tasa de éxito se mantuvo en el 100 % y el tiempo de respuesta se conservó en el mismo orden de magnitud.** El sistema absorbió el doble de trabajo sin degradarse de forma apreciable.

La variación entre 12.2s y 16.4s es consistente y bimodal: dos procesos rondaron los 12.2s y los otros dos los 14.5s y 16.4s. Se descartó que fuera agotamiento de créditos de ráfaga, porque la métrica `CPU Credits Remaining` se mantuvo entre **26.8 y 31.9** en las cuatro máquinas, lejos de cero.

La explicación más probable es la **heterogeneidad del hardware subyacente**: las cuatro VMs son del mismo tamaño nominal, pero Azure las aprovisiona sobre hosts físicos que pueden tener generaciones de procesador distintas. Para una carga puramente CPU-bound, esa diferencia se traduce directamente en tiempo de respuesta. Es una limitación conocida de los tamaños económicos, donde no se garantiza un modelo de CPU específico.

#### Por qué aumenta la tasa de éxito con escalamiento horizontal

Tres razones, en orden de importancia:

1. **La capacidad crece con los nodos.** Cada VM aporta un núcleo completo. Como la aplicación es monohilo y CPU-bound, cada nodo atiende una petición a la vez de forma eficiente. Con 4 nodos hay 4 peticiones en curso simultáneamente, cada una a velocidad plena.

2. **Sin escalamiento horizontal, la carga se serializa.** En una sola máquina, las 40 peticiones competirían por el mismo núcleo: el event loop de Node las encolaría y cada una tardaría tanto como la suma de las que tiene delante. A 12 segundos por petición, las últimas esperarían varios minutos y muchas morirían por timeout del cliente. El fallo no viene de que el servidor rechace, sino de que no alcanza a responder a tiempo.

3. **Los fallos quedan aislados.** Si un nodo se cae o se satura, el health probe lo saca de rotación en 10 segundos (2 fallos × 5 s) y sus peticiones se redistribuyen. En una sola máquina, su caída es la caída del sistema completo.

El contraste con el escalamiento vertical es claro: subir de `B1ls` a `B2ms` multiplica los recursos de **una** máquina, pero mantiene el punto único de falla y choca contra el techo del tamaño más grande disponible. Agregar nodos no tiene ese techo, y además permite repartirlos en zonas de disponibilidad distintas, lo que añade tolerancia a fallos de centro de datos.

### Consumo de CPU de las 4 VMs

Métrica *Percentage CPU*, agregación máximo, muestreo por minuto, durante la prueba con 4 procesos.

| VM | Zona | Pico de CPU |
|---|---|---|
| VM1 | 1 | **95.9 %** |
| VM2 | 2 | **90.2 %** |
| VM3 | 3 | **96.0 %** |
| VM4 | 1 | **96.2 %** |

Serie por minuto:

```
VM1:   0.4   0.4  95.9  95.5
VM2:  56.2  90.2  89.0
VM3:  32.7  96.0  93.6   0.4
VM4:  47.4  28.9  96.2  96.2  55.1  0.4
```

![CPU máxima de VM1 durante la prueba con 4 procesos](images/entrega/part2/19-cpu-4x-vm1.png)

![CPU máxima de VM2 durante la prueba con 4 procesos](images/entrega/part2/20-cpu-4x-vm2.png)

![CPU máxima de VM3 durante la prueba con 4 procesos](images/entrega/part2/21-cpu-4x-vm3.png)

![CPU máxima de VM4 durante la prueba con 4 procesos](images/entrega/part2/22-cpu-4x-vm4.png)

Las cuatro máquinas se saturaron de forma prácticamente simultánea, lo que confirma que la carga se repartió de manera uniforme. Ninguna quedó ociosa, a diferencia de VM2 en la prueba anterior. En las gráficas se aprecia además el pico menor y anterior de la prueba con 2 procesos, donde solo VM1 y VM3 registraron actividad.

El criterio de CPU por debajo del 70 % del escenario de calidad **sigue sin cumplirse**, y es coherente: cada nodo tiene un solo vCPU y una petición CPU-bound basta para saturarlo. Agregar nodos aumenta el **throughput** del sistema y su disponibilidad, pero no reduce el uso de CPU de cada nodo individual. Para bajar del 70 % harían falta más nodos que peticiones concurrentes, VMs con varios núcleos, o corregir el algoritmo de Fibonacci, que es la causa de fondo.

## Preguntas: Parte 2

**1. ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian? ¿Qué es SKU, qué tipos hay y en qué se diferencian? ¿Por qué el balanceador de carga necesita una IP pública?**

Azure ofrece cuatro servicios de balanceo, que se diferencian por la **capa del modelo OSI** en la que operan y por su **alcance geográfico**:

| Servicio | Capa | Alcance | Decide según | Caso de uso |
|---|---|---|---|---|
| **Azure Load Balancer** | 4 (TCP/UDP) | Regional | IP y puerto | Tráfico TCP/UDP genérico dentro de una región. Es el que usa este laboratorio. |
| **Application Gateway** | 7 (HTTP/S) | Regional | URL, host, cabeceras, cookies | Enrutamiento por ruta, terminación SSL, afinidad por cookie y firewall de aplicación (WAF). |
| **Traffic Manager** | DNS | Global | Resolución DNS | Reparte entre regiones devolviendo distintas IPs. No ve el tráfico, solo resuelve nombres. |
| **Front Door** | 7 | Global | URL y ubicación del cliente | Distribución global con caché de CDN, aceleración y WAF. |

La diferencia esencial es que el **Load Balancer no entiende HTTP**: toma la decisión mirando únicamente la cabecera TCP. Esto lo hace muy rápido y agnóstico al protocolo, pero le impide enrutar por URL o leer cookies. Para este laboratorio es suficiente y es lo correcto, porque solo necesitamos repartir conexiones hacia un mismo servicio.

**SKU** (*Stock Keeping Unit*) es el nivel de servicio del recurso: define capacidades, límites y precio.

| SKU | Estado | Características |
|---|---|---|
| **Basic** | **Retirado el 30/09/2025** | Gratuito, sin soporte de zonas de disponibilidad, hasta 300 instancias, sin SLA. |
| **Standard** | Vigente | Soporta zonas de disponibilidad, hasta 1000 instancias, SLA del 99.99 %, reglas de salida configurables y es *seguro por defecto*: todo el tráfico se bloquea salvo que un NSG lo permita explícitamente. |
| **Gateway** | Vigente | Para encadenar appliances virtuales de red (NVA) de terceros. |

En este laboratorio el SKU **Standard era obligatorio** por dos razones: Basic ya no existe, y es el único que permite distribuir las VMs en zonas de disponibilidad, que es un requisito del ejercicio.

**Por qué necesita una IP pública:** porque se configuró como balanceador de tipo *Public*, es decir, el punto de entrada del sistema desde internet. La configuración de frontend necesita una dirección enrutable públicamente para que los clientes puedan alcanzarla; en nuestro caso `20.48.168.44`. Las VMs detrás solo tienen IPs privadas (`10.1.0.4` a `10.1.0.7`), que no son alcanzables desde fuera de la red virtual.

En esta implementación esa IP cumple **una segunda función**: como las VMs no tienen IP pública propia, el balanceador les provee la salida a internet mediante SNAT a través de su frontend. Fue lo que permitió que las VMs descargaran Node.js y clonaran el repositorio.

**2. ¿Cuál es el propósito del *Backend Pool*?**

El backend pool es el **conjunto de instancias que atenderán el tráfico** que entre por el balanceador. En esta implementación, `SCALABILITY_LAB_BACKEND_POOL` agrupa las interfaces de red de VM1, VM2, VM3 y VM4.

Su valor arquitectónico es el **desacoplamiento**: el cliente conoce una sola dirección estable (la IP del frontend) y nunca sabe cuántos nodos hay detrás ni cuáles son. Agregar o quitar máquinas es una operación sobre el pool que no obliga a tocar la regla de balanceo, ni la IP pública, ni la configuración del cliente.

Eso es exactamente lo que hace posible la escalabilidad horizontal, y se comprobó en la práctica: **VM4 se agregó al sistema en caliente**, sin interrumpir el servicio, sin cambiar la IP `20.48.168.44` y sin reconfigurar nada del lado del cliente.

**3. ¿Cuál es el propósito del *Health Probe*?**

El health probe **verifica periódicamente que cada instancia del backend pool esté viva** y saca de rotación a las que no respondan, para que el balanceador no les envíe tráfico.

Configuración usada:

| Parámetro | Valor | Significado |
|---|---|---|
| Protocolo | TCP | Verifica que el puerto acepte conexiones |
| Puerto | **3000** | El puerto donde escucha la aplicación, no el 80 del frontend |
| Intervalo | 5 segundos | Frecuencia de la comprobación |
| Umbral | 2 fallos | Comprobaciones fallidas consecutivas antes de marcar la instancia como no sana |

El detalle importante es que **el probe apunta al 3000, no al 80**. El puerto 80 es donde escucha el balanceador de cara a internet; dentro de la red virtual la aplicación escucha en el 3000. Si el probe apuntara al 80, fallaría en todas las instancias y el balanceador vaciaría el pool completo.

Se comprobó su funcionamiento de forma involuntaria durante el despliegue: mientras VM2 y VM3 todavía estaban instalando la aplicación, **el balanceador siguió respondiendo correctamente** porque el probe las tenía marcadas como no sanas y enviaba todo el tráfico a VM1, la única que ya tenía el servicio arriba. Sin health probe, un tercio de las peticiones habría fallado durante ese periodo.

**4. ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?**

La regla de balanceo es la que **une todas las piezas**: asocia una configuración de frontend y un puerto de entrada con un backend pool y un puerto de destino, usando un health probe como criterio de salud.

La regla `SCALABILITY_LAB_LOAD_BALANCING_RULE` define: TCP, puerto de frontend **80**, puerto de backend **3000**, pool `SCALABILITY_LAB_BACKEND_POOL`, probe `SCALABILITY_LAB_HEALTH_PROBE`. Es lo que permite consumir el servicio como `http://20.48.168.44/fibonacci/6` aunque internamente la aplicación viva en el 3000.

**Tipos de sesión persistente en Azure Load Balancer:**

| Tipo | Hash | Comportamiento |
|---|---|---|
| **None** (predeterminado) | 5-tupla: IP origen, puerto origen, IP destino, puerto destino, protocolo | Cada conexión nueva puede ir a un nodo distinto. Es la mejor distribución. |
| **Client IP** | 2-tupla: IP origen, IP destino | Todas las conexiones de una misma IP cliente van siempre al mismo nodo. |
| **Client IP and protocol** | 3-tupla: IP origen, IP destino, protocolo | Igual que la anterior, diferenciando por protocolo. |

**Por qué importa:** si la aplicación guarda estado en memoria del servidor (una sesión de usuario, un carrito de compras), las peticiones de ese usuario deben llegar siempre al mismo nodo o el estado se pierde. La persistencia resuelve eso.

**Cómo afecta la escalabilidad:** negativamente, y de forma severa. Con *Client IP*, todo el tráfico que venga detrás de un mismo NAT corporativo o universitario se concentra en un solo nodo, dejando el resto ociosos. Peor aún: al escalar, los nodos nuevos solo reciben clientes nuevos, nunca descargan a los que ya están saturados. Y si un nodo cae, sus usuarios pierden la sesión.

Por eso se configuró **None**: la aplicación de Fibonacci es *stateless*, no guarda nada entre peticiones, así que no necesita afinidad y conviene repartir lo máximo posible.

**Hallazgo experimental relevante:** incluso con persistencia en *None*, se observó afinidad de facto. El hash de 5-tupla se evalúa **por conexión TCP**, no por petición HTTP. Como `newman` mantiene la conexión viva (keep-alive) durante sus 10 iteraciones, las 10 peticiones viajaron por el mismo socket y cayeron todas en el mismo nodo. El resultado fue que **con 2 procesos solo se usaron 2 de las 3 VMs**, mientras la tercera quedaba ociosa al 0.4 % de CPU.

La conclusión práctica es que, en un balanceador de capa 4, **el paralelismo real está limitado por el número de conexiones TCP que abra el cliente**, no por el tamaño del backend pool. Se verificó por contraste: lanzando 18 peticiones con `--no-keepalive`, el reparto fue exactamente 6, 6 y 6. Un Application Gateway, al operar en capa 7, sí distribuiría petición por petición sobre una misma conexión.

**5. ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?**

Una **Virtual Network (VNet)** es una red privada y aislada lógicamente dentro de Azure. Funciona como el equivalente en la nube de una red local: los recursos que viven dentro se comunican entre sí por direcciones privadas, sin pasar por internet, y están aislados del tráfico de otros clientes de Azure. Aquí es `SCALABILITY_LAB_VIRTUAL_NETWORK`.

Una **Subnet** es una subdivisión de esa red. Sirve para segmentar: agrupar recursos con necesidades similares de conectividad y de seguridad, y aplicarles políticas distintas (NSG, tablas de rutas). En una arquitectura real se separarían la capa web, la de aplicación y la de base de datos en subredes distintas. En este laboratorio basta una, `SCALABILITY_LAB_SUBNET`, porque los 4 nodos son idénticos y cumplen la misma función.

El **address space** es el rango total de direcciones IP privadas que la VNet reserva, expresado en notación CIDR. Aquí es `10.1.0.0/16`, que abarca de `10.1.0.0` a `10.1.255.255`, es decir 65.536 direcciones.

El **address range** es la porción de ese espacio que se asigna a una subred concreta. Aquí es `10.1.0.0/24`, de `10.1.0.0` a `10.1.0.255`, 256 direcciones, de las cuales Azure reserva 5 (la primera, la última y tres para servicios internos), dejando 251 utilizables.

Su propósito es doble: **evitar colisiones de direcciones**, tanto entre subredes como con redes locales en escenarios híbridos conectados por VPN, y **dimensionar** la capacidad de crecimiento. En este despliegue las VMs recibieron `10.1.0.4`, `10.1.0.5`, `10.1.0.6` y `10.1.0.7` dentro de ese rango.

**6. ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas? ¿Qué significa que una IP sea *zone-redundant*?**

Una **Availability Zone** es una ubicación físicamente separada dentro de una misma región de Azure, con **alimentación eléctrica, refrigeración y red independientes**. Cada región que las soporta tiene al menos tres, separadas por una distancia suficiente para que un desastre localizado no afecte a más de una, pero conectadas por fibra de baja latencia para que se comporten como una sola red.

**Por qué 3 zonas distintas:** para eliminar el punto único de falla a nivel de centro de datos. Si las tres VMs estuvieran en el mismo edificio, un corte de energía las tumbaría todas a la vez y el sistema caería completo, por más balanceador que hubiera. Repartidas en 3 zonas, la caída de una deja el sistema operando al 66 % de capacidad: el health probe detecta las instancias caídas en 10 segundos (2 fallos × 5 s de intervalo) y las saca de rotación, y el tráfico sigue fluyendo a las sobrevivientes.

Es la diferencia entre escalar para **rendimiento** y escalar para **disponibilidad**. El backend pool da lo primero; distribuirlo en zonas da lo segundo.

En este despliegue: VM1 en zona 1, VM2 en zona 2, VM3 en zona 3 y VM4 en zona 1.

**Que una IP sea *zone-redundant*** significa que Azure la sirve **simultáneamente desde las tres zonas** en lugar de anclarla a una. Si una zona cae, la dirección sigue siendo alcanzable desde las otras dos, sin cambio de IP ni intervención manual.

Esto es imprescindible aquí: de nada serviría tener las VMs repartidas en 3 zonas si la IP del balanceador, que es el único punto de entrada, viviera en una sola. La caída de esa zona dejaría el sistema inalcanzable aunque todos los nodos siguieran sanos. Por eso `SCALABILITY_LAB_LOAD_BALANCER_IP` se creó como **Standard, Static y Zone-redundant**. Nótese que solo el SKU Standard admite esta propiedad.

**7. ¿Cuál es el propósito del *Network Security Group*?**

El NSG es un **firewall distribuido con estado** que filtra el tráfico de red mediante reglas de 5-tupla (origen, puerto origen, destino, puerto destino, protocolo). Se puede asociar a una subred o a una interfaz de red; aquí `LOAD_BALANCER_NSG` está asociado a las 4 NICs.

Reglas de entrada configuradas:

| Prioridad | Nombre | Puerto | Propósito |
|---|---|---|---|
| 1000 | `default-allow-ssh` | 22 | Administración remota de las VMs |
| 1010 | `Port_3000` | 3000 | Tráfico de la aplicación, que es el que reenvía el balanceador |

Las reglas se evalúan **en orden de prioridad ascendente** y la primera que coincide gana. Además de las personalizadas, Azure añade reglas por defecto en prioridades altas, entre ellas `AllowAzureLoadBalancerInBound` (65001), que permite el tráfico del balanceador, **incluidos los health probes**, y `DenyAllInBound` (65500), que bloquea todo lo demás.

Por qué es necesario aquí: el SKU **Standard es cerrado por defecto**. Sin una regla explícita que abra el 3000, el health probe no podría alcanzar la aplicación, marcaría las cuatro instancias como no sanas y el balanceador devolvería error aunque la aplicación estuviera funcionando perfectamente. El NSG es, por tanto, un requisito funcional del sistema, no solo una medida de seguridad.

**8. Informe de newman 1 (Punto 2)**

Los resultados completos están en la sección [Prueba de newman con 2 procesos en paralelo](#prueba-de-newman-con-2-procesos-en-paralelo) de las evidencias. Resumen:

| Métrica | Valor |
|---|---|
| Peticiones totales | 20 (2 procesos × 10 iteraciones) |
| Peticiones exitosas | **20 (100 %)** |
| Peticiones fallidas | **0** |
| Tiempo de respuesta promedio | 12.3 s y 12.7 s |
| Rango | 12.2 s a 13.5 s |
| Desviación estándar | 54 ms y 458 ms |
| Pico de CPU en los nodos con carga | 95.7 % y 95.1 % |

La interpretación está desarrollada en [Evaluación del escenario de calidad](#evaluación-del-escenario-de-calidad): el requisito de **responder todas las peticiones se cumple**, el de **mantener la CPU por debajo del 70 % no**, y la causa de fondo es el algoritmo de Fibonacci, no la infraestructura.

---

# Diagrama de despliegue

```mermaid
graph TD
    CLI["Cliente<br/>newman / navegador"]

    subgraph REGION["Azure · Región Canada Central"]
        PIP["SCALABILITY_LAB_LOAD_BALANCER_IP<br/>20.48.168.44<br/>Standard · Static · Zone-redundant"]
        LB["SCALABILITY_LAB_LOAD_BALANCER<br/>SKU Standard · Público<br/>Rule: TCP 80 a TCP 3000<br/>Probe: TCP 3000 cada 5s, umbral 2"]

        subgraph VNET["SCALABILITY_LAB_VIRTUAL_NETWORK · 10.1.0.0/16"]
            subgraph SUBNET["SCALABILITY_LAB_SUBNET · 10.1.0.0/24<br/>NSG LOAD_BALANCER_NSG · puertos 22 y 3000"]
                VM1["VM1 · Zona 1 · 10.1.0.4<br/>Ubuntu 22.04 · B1ls<br/>FibonacciApp :3000"]
                VM2["VM2 · Zona 2 · 10.1.0.5<br/>Ubuntu 22.04 · B1ls<br/>FibonacciApp :3000"]
                VM3["VM3 · Zona 3 · 10.1.0.6<br/>Ubuntu 22.04 · B1ls<br/>FibonacciApp :3000"]
                VM4["VM4 · Zona 1 · 10.1.0.7<br/>Ubuntu 22.04 · B1ls<br/>FibonacciApp :3000"]
            end
        end
    end

    CLI -->|HTTP puerto 80| PIP
    PIP --> LB
    LB -->|TCP 3000| VM1
    LB -->|TCP 3000| VM2
    LB -->|TCP 3000| VM3
    LB -->|TCP 3000| VM4
    VM1 -.->|SNAT de salida| LB
    VM2 -.->|SNAT de salida| LB
    VM3 -.->|SNAT de salida| LB
    VM4 -.->|SNAT de salida| LB
```

**Nodos y artefactos desplegados**

| Nodo | Artefacto | Puerto | Zona |
|---|---|---|---|
| VM1 | FibonacciApp sobre Node.js 20, servicio `fibonacci.service` | 3000 | 1 |
| VM2 | FibonacciApp sobre Node.js 20, servicio `fibonacci.service` | 3000 | 2 |
| VM3 | FibonacciApp sobre Node.js 20, servicio `fibonacci.service` | 3000 | 3 |
| VM4 | FibonacciApp sobre Node.js 20, servicio `fibonacci.service` | 3000 | 1 |

**Flujos de comunicación**

- **Entrada** (línea continua): el cliente llega a la IP pública por el puerto 80. La regla de balanceo reenvía al puerto 3000 de un nodo del backend pool, elegido por hash de 5-tupla sobre la conexión TCP.
- **Salida** (línea punteada): como los nodos no tienen IP pública propia, su tráfico saliente se traduce (SNAT) a través del frontend del balanceador. Es lo que les permitió descargar Node.js y clonar el repositorio durante el aprovisionamiento.
- **Supervisión**: el health probe abre una conexión TCP al puerto 3000 de cada nodo cada 5 segundos. Tras 2 fallos consecutivos, el nodo sale de rotación y deja de recibir tráfico.

---

# Conclusiones

**1. El escalamiento horizontal resolvió la disponibilidad, no la saturación.**

En las dos pruebas de carga se respondió el **100 % de las peticiones**, 20 de 20 y 40 de 40, sin un solo fallo. Pero el consumo de CPU de cada nodo con tráfico alcanzó picos del 95 %, muy por encima del 70 % que exige el escenario de calidad. El escenario, por tanto, **se cumple parcialmente**.

La distinción es importante: agregar nodos multiplica el **throughput** del sistema y aísla los fallos, pero no reduce el uso de CPU de cada máquina individual. Un nodo saturado sigue saturado aunque haya otros tres al lado.

**2. La causa raíz no es la infraestructura, es el algoritmo.**

Calcular el término 1.000.000 de Fibonacci iterando un millón de sumas de enteros de precisión arbitraria cuesta 12 segundos de CPU al 100 %. Ninguna topología de despliegue arregla eso: solo lo reparte. Escalar fue una forma de **comprar tiempo**, no de resolver el problema. La solución real sería un algoritmo eficiente, o cachear resultados, antes que añadir máquinas.

**3. El paralelismo real lo impone el cliente, no el backend pool.**

Fue el hallazgo más inesperado del laboratorio. Con la persistencia de sesión en `None`, se esperaba reparto uniforme; sin embargo, con 2 procesos de newman solo se usaron **2 de 3 nodos**, dejando uno ocioso al 0.4 % mientras los otros dos agonizaban al 95 %.

La explicación es que Azure Load Balancer opera en **capa 4**: su hash de 5-tupla se evalúa por conexión TCP, no por petición HTTP. Como newman mantiene keep-alive, sus 10 peticiones viajan por el mismo socket y aterrizan en el mismo nodo. Al repetir con `--no-keepalive` el reparto fue exacto: **6, 6 y 6**. Y con 4 procesos sobre 4 nodos, **10, 10, 10 y 10**.

La lección de diseño: para una aplicación HTTP con clientes que reutilizan conexiones, un **Application Gateway** (capa 7), que distribuye petición por petición, repartiría mejor que un Load Balancer.

**4. La elasticidad es real y verificable.**

VM4 se incorporó al sistema **sin interrumpir el servicio**: sin cambiar la IP pública, sin tocar la regla de balanceo y sin reconfigurar el cliente. El health probe la detectó y empezó a enviarle tráfico automáticamente. Eso es precisamente lo que el escalamiento vertical no puede ofrecer, porque redimensionar una VM exige reiniciarla y provoca downtime.

**5. Las restricciones reales del entorno moldean la arquitectura.**

El laboratorio no se pudo ejecutar tal como estaba escrito. La suscripción impuso una política que limita a 5 regiones, de las cuales solo 2 ofrecían `B1ls` con zonas de disponibilidad; un tope de 3 IPs públicas por región que obligó a dejar las VMs sin IP propia; y el retiro del SKU Basic. Cada restricción exigió una decisión de diseño: usar Canada Central, resolver la salida a internet por el SNAT del balanceador, y usar Standard.

Ninguna alteró la arquitectura evaluada, pero todas confirman que **una arquitectura no se diseña en el vacío**: las capacidades y los límites del proveedor son parte del problema.

**6. Disponibilidad por zonas.**

Distribuir los nodos en 3 zonas de disponibilidad y usar una IP zone-redundant protege contra la caída de un centro de datos completo. Con el health probe configurado a 5 segundos de intervalo y umbral de 2, el sistema detecta y aísla un nodo caído en unos **10 segundos**, y sigue operando con capacidad reducida en lugar de caer por completo.

---

# Limpieza de recursos

- [ ] Portal de Azure → *Resource groups* → `SCALABILITY_LAB` → **Delete resource group**

Esto elimina en una sola operación el Load Balancer, la IP pública, la Virtual Network, el NSG, las VMs, sus discos y sus NIC.
