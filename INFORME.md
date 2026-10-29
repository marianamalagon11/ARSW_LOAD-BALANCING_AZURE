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
  - [Checklist A: Balanceador de carga y red](#checklist-a-balanceador-de-carga-y-red)
  - [Checklist B: Máquinas virtuales (nodos)](#checklist-b-máquinas-virtuales-nodos)
  - [Checklist C: Pruebas de la infraestructura](#checklist-c-pruebas-de-la-infraestructura)
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

**Nombres de los recursos** (tomados de las imágenes guía del enunciado):

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

## Checklist A: Balanceador de carga y red

**A.1: Crear el Load Balancer** · guía: [part2-lb-create.png](images/part2/part2-lb-create.png)

- [ ] Resource group: `SCALABILITY_LAB`
- [ ] Name: `SCALABILITY_LAB_LOAD_BALANCER`
- [ ] Region: la misma para todo el laboratorio
- [ ] Type: **Public**
- [ ] SKU: **Standard**
- [ ] Public IP address: **Create new** con nombre `SCALABILITY_LAB_LOAD_BALANCER_IP`
- [ ] Assignment: **Static**
- [ ] Availability zone: **Zone-redundant**
- [ ] Add a public IPv6 address: **No**
- [ ] Anotar la IP pública asignada: `_______________`

**A.2: Crear el Backend Pool** · guía: [part2-lb-bp-create.png](images/part2/part2-lb-bp-create.png)

- [ ] *Settings → Backend pools → Add*
- [ ] Name: `SCALABILITY_LAB_BACKEND_POOL`
- [ ] IP version: **IPv4**
- [ ] Virtual network: `SCALABILITY_LAB_VIRTUAL_NETWORK`
- [ ] Dejarlo **sin VMs por ahora**, se asocian al crear cada VM

> Si Azure exige la Virtual Network para crear el backend pool, ejecutar primero el paso **A.5**.

**A.3: Crear el Health Probe** · guía: [part2-lb-hp-create.png](images/part2/part2-lb-hp-create.png)

- [ ] *Settings → Health probes → Add*
- [ ] Name: `SCALABILITY_LAB_HEALTH_PROBE`
- [ ] Protocol: **TCP**
- [ ] Port: **3000**
- [ ] Interval: **5** segundos
- [ ] Unhealthy threshold: **2** fallos consecutivos

**A.4: Crear la Load Balancing Rule** · guía: [part2-lb-lbr-create.png](images/part2/part2-lb-lbr-create.png)

- [ ] *Settings → Load balancing rules → Add*
- [ ] Name: `SCALABILITY_LAB_LOAD_BALANCING_RULE`
- [ ] IP Version: **IPv4**
- [ ] Frontend IP address: la IP del `LoadBalancerFrontEnd`
- [ ] Protocol: **TCP**
- [ ] Port (frontend): **80**
- [ ] Backend port: **3000**
- [ ] Backend pool: `SCALABILITY_LAB_BACKEND_POOL`
- [ ] Health probe: `SCALABILITY_LAB_HEALTH_PROBE (TCP:3000)`
- [ ] Session persistence: **None**
- [ ] Floating IP: **Disabled**

> Esta regla es la que permite consumir el servicio como `http://<IP_LB>/fibonacci/6` por el puerto 80, aunque la app escuche en el 3000 dentro de cada VM.

**A.5: Crear la Virtual Network** · guía: [part2-vn-create.png](images/part2/part2-vn-create.png)

- [ ] Name: `SCALABILITY_LAB_VIRTUAL_NETWORK`
- [ ] Address space: **10.1.0.0/16**
- [ ] Resource group: `SCALABILITY_LAB`
- [ ] Location: la misma región del LB
- [ ] Subnet name: `SCALABILITY_LAB_SUBNET`
- [ ] Subnet address range: **10.1.0.0/24**
- [ ] DDoS protection: **Basic** · Service endpoints: **Disabled** · Firewall: **Disabled**

## Checklist B: Máquinas virtuales (nodos)

Repetir este bloque **3 veces** (`VM1`, `VM2`, `VM3`), cambiando el **nombre**, el **usuario** y la **Availability Zone**.

| VM | Availability Zone | Username | IP pública | IP privada |
|---|---|---|---|---|
| VM1 | 1 | `vm1` | | |
| VM2 | 2 | `vm2` | | |
| VM3 | 3 | `vm3` | | |
| VM4 *(punto 3)* | | `vm4` | | |

**B.1: Pestaña Basics** · guía: [part2-vm-create1.png](images/part2/part2-vm-create1.png)

- [ ] Resource group: `SCALABILITY_LAB`
- [ ] Virtual machine name: `VM1` / `VM2` / `VM3`
- [ ] Region: la misma región del LB
- [ ] Availability options: **Availability zone**
- [ ] Availability zone: **1 para VM1, 2 para VM2, 3 para VM3**
- [ ] Image: **Ubuntu Server 18.04 LTS**
- [ ] Size: **Standard B1ls**
- [ ] Authentication type: **SSH public key**
- [ ] Username: `vm1` / `vm2` / `vm3`
- [ ] SSH public key: pegar la llave pública propia

> La Availability Zone se fija al crear la VM y **no se puede cambiar después**.

**B.2: Pestaña Networking** · guía: [part2-vm-create2.png](images/part2/part2-vm-create2.png)

- [ ] Virtual network: `SCALABILITY_LAB_VIRTUAL_NETWORK`
- [ ] Subnet: `SCALABILITY_LAB_SUBNET (10.1.0.0/24)`
- [ ] Public IP: **Create new** con nombre `VM1-ip`, SKU **Standard**, Assignment **Static**
- [ ] Public IP, Availability zone: **Zone-redundant**
- [ ] Accelerated networking: Off

**B.3: Network Security Group** · guía: [part2-vm-create3.png](images/part2/part2-vm-create3.png)

- [ ] NIC network security group: **Advanced**
- [ ] **Solo en VM1:** *Create new* con nombre `LOAD_BALANCER_NSG`
  - [ ] Regla `1000: default-allow-ssh` → Any, SSH (TCP/22)
  - [ ] Regla `1010: Port_3000` → Any, Custom (Any/**3000**)
- [ ] **En VM2, VM3 y VM4:** seleccionar el NSG `LOAD_BALANCER_NSG` ya creado, no crear uno nuevo

**B.4: Asociar la VM al Load Balancer** · guía: [part2-vm-create4.png](images/part2/part2-vm-create4.png)

- [ ] Place this virtual machine behind an existing load balancing solution: **Yes**
- [ ] Load balancing options: **Azure load balancer**
- [ ] Select a load balancer: `SCALABILITY_LAB_LOAD_BALANCER`
- [ ] Select a backend pool: `SCALABILITY_LAB_BACKEND_POOL`
- [ ] *Review + create* y luego **Create**

**B.5: Instalar la aplicación en la VM**

- [ ] Conectarse: `ssh vm1@<IP_PUBLICA_VM1>`
- [ ] Ejecutar el bloque de comandos

```bash
git clone https://github.com/marianamalagon11/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

> En VM2, VM3 y VM4 hay que cambiar `vm1` por el usuario correcto en la línea `source /home/vm1/.bashrc`, de lo contrario `nvm` no queda disponible y falla el `npm install`.

- [ ] Verificar que el proceso quedó arriba: `forever list`
- [ ] Verificar la VM directamente: `http://<IP_PUBLICA_VM>:3000/fibonacci/6` debe responder `The answer is 8`

**B.6: Verificar el Backend Pool**

- [ ] En el Load Balancer, sección *Backend pools*, confirmar que aparecen las VMs con su IP privada
- [ ] Confirmar que todas las VMs están **sanas** según el Health Probe

## Checklist C: Pruebas de la infraestructura

**C.1: Verificación funcional**

- [ ] `http://<IP_LB>/` responde `Hello World`
- [ ] `http://<IP_LB>/fibonacci/1` responde `The answer is 1`
- [ ] Refrescar varias veces y confirmar que el tráfico se reparte entre las VMs

**C.2: Prueba de carga comparativa (punto 2)**

- [ ] En `FibonacciApp/postman/part2/[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambiar `loadbalancer` por la IP pública del LB
- [ ] Ejecutar newman con **2 procesos en paralelo**, el mismo escenario de la Parte 1
- [ ] Capturar el resumen de newman
- [ ] Capturar la CPU de las 3 VMs durante la prueba
- [ ] Redactar el informe comparativo: tiempos de respuesta, peticiones exitosas y **costos** de ambas infraestructuras

```bash
cd FibonacciApp/postman/part2
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e "[ARSW_LOAD-BALANCING_AZURE].postman_environment.json" -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e "[ARSW_LOAD-BALANCING_AZURE].postman_environment.json" -n 10
```

**C.3: Cuarta VM y carga x4 (punto 3)**

- [ ] Crear `VM4` repitiendo el **Checklist B**, reutilizando el NSG existente
- [ ] Confirmar que `VM4` quedó en el backend pool y está sana
- [ ] Ejecutar newman con **4 procesos en paralelo**
- [ ] Capturar la CPU de las **4** VMs
- [ ] Explicar por qué la tasa de éxito aumentó con este estilo de escalabilidad

```bash
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e "[ARSW_LOAD-BALANCING_AZURE].postman_environment.json" -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e "[ARSW_LOAD-BALANCING_AZURE].postman_environment.json" -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e "[ARSW_LOAD-BALANCING_AZURE].postman_environment.json" -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e "[ARSW_LOAD-BALANCING_AZURE].postman_environment.json" -n 10
```

## Desviaciones respecto a la guía

El enunciado es de 2019 y la suscripción usada es *Azure for Students*. Estas son las diferencias entre lo que pide la guía y lo que se pudo implementar, con su justificación técnica.

| Aspecto | Guía | Implementado | Razón |
|---|---|---|---|
| Región | North Europe | **Canada Central** | Azure aplica a la suscripción la política `Allowed resource deployment regions` (efecto *Denegar*), que la limita a 5 regiones: `southcentralus`, `northcentralus`, `canadacentral`, `brazilsouth` y `chilecentral`. De esas, solo Canada Central y Chile Central ofrecen `Standard_B1ls` con las 3 zonas de disponibilidad. |
| IP pública por VM | Una IP pública Standard por VM | **VMs sin IP pública** | La suscripción tiene un límite de 3 IPs públicas por región y el balanceador consume una. Las VMs obtienen salida a internet mediante el SNAT saliente del Load Balancer Standard, verificado con `curl` (HTTP 200). La entrada sigue siendo exclusivamente por el balanceador, que es lo que exige la arquitectura. |
| Imagen del SO | Ubuntu Server 18.04 LTS | **Ubuntu Server 22.04 LTS** | La 18.04 llegó a fin de vida y ya no se publica en el Marketplace. |
| Gestor del proceso | `forever` | **Servicio `systemd`** | La instalación se automatizó con `az vm run-command`, que es no interactivo. `systemd` cumple la misma función que `forever` (mantener el proceso vivo al cerrar la sesión) y además lo reinicia automáticamente si falla. |
| Subred | Con acceso de salida predeterminado | **Subred privada** | Desde el 31/03/2026 Azure crea las subredes como privadas por defecto. No afecta el funcionamiento porque la salida se resuelve por el balanceador. |
| SKU del balanceador | Basic o Standard | **Standard** | Microsoft retiró el SKU Basic el 30/09/2025. Standard es además el único que soporta zonas de disponibilidad. |

Ninguna de estas desviaciones altera la arquitectura evaluada: sigue siendo un balanceador de carga público que distribuye tráfico entre nodos idénticos repartidos en 3 zonas de disponibilidad.

## Evidencias: Parte 2

### Grupo de recursos

![Grupo de recursos SCALABILITY_LAB en Canada Central](images/entrega/part2/00-resource-group.png)

### Virtual Network

![Red virtual con espacio de direcciones 10.1.0.0/16](images/entrega/part2/01-virtual-network.png)

### Subnet

![Subred SCALABILITY_LAB_SUBNET con rango 10.1.0.0/24](images/entrega/part2/02-subnet.png)

### Load Balancer

![Balanceador Standard con IP de frontend 20.48.168.44](images/entrega/part2/03-load-balancer.png)

### Backend Pool

![Backend pool con VM1, VM2 y VM3 en zonas 1, 2 y 3](images/entrega/part2/04-backend-pool.png)

### Health Probe

![Health probe TCP sobre el puerto 3000 cada 5 segundos](images/entrega/part2/05-health-probe.png)

### Load Balancing Rule

![Regla de balanceo TCP/80 hacia TCP/3000](images/entrega/part2/06-load-balancing-rule.png)

### Network Security Group

![NSG con las reglas de entrada SSH 22 y Port_3000](images/entrega/part2/07-network-security-group.png)

### VM1 (zona 1)

![VM1 en Canada Central zona 1, Standard B1ls](images/entrega/part2/08-vm1-zona1.png)

### VM2 (zona 2)

![VM2 en Canada Central zona 2, Standard B1ls](images/entrega/part2/09-vm2-zona2.png)

### VM3 (zona 3)

![VM3 en Canada Central zona 3, Standard B1ls](images/entrega/part2/10-vm3-zona3.png)

### Aplicación instalada en las VMs

### Verificación funcional del balanceador

![Respuesta del balanceador en /fibonacci/6](images/entrega/part2/12-balanceador-funcionando.png)

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

La variación entre 12.2s y 16.4s tiene una explicación plausible: VM1 y VM3 ya habían ejecutado la prueba anterior y tenían parte de sus créditos de ráfaga consumidos, mientras que VM2 y VM4 llegaban en reposo. No se verificó de forma concluyente a qué nodo fue cada proceso, así que queda como hipótesis.

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

---

# Conclusiones

---

# Limpieza de recursos

- [ ] Portal de Azure → *Resource groups* → `SCALABILITY_LAB` → **Delete resource group**

Esto elimina en una sola operación el Load Balancer, la IP pública, la Virtual Network, el NSG, las VMs, sus discos y sus NIC.
