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

**Antes de empezar**
- [ ] Eliminar el grupo de recursos de la Parte 1 (o usar uno limpio) para evitar cobros adicionales
- [ ] Guardar los resultados de la Parte 1 antes de borrarla: se necesitan para el informe comparativo
- [ ] Tener la **llave SSH pública** (`~/.ssh/id_rsa.pub`); si no existe, generarla con `ssh-keygen -t rsa -b 4096`
- [ ] Tener `newman` instalado en la máquina local: `npm install newman -g`

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

## Evidencias: Parte 2

### Load Balancer creado

### Backend Pool

### Health Probe

### Load Balancing Rule

### Virtual Network y Subnet

### Network Security Group

### VM1 (zona 1)

### VM2 (zona 2)

### VM3 (zona 3)

### Aplicación instalada en las VMs

### Backend Pool con las VMs asociadas

### Verificación funcional del balanceador

### Prueba de newman con 2 procesos en paralelo

### Consumo de CPU de las VMs

### Informe comparativo con la Parte 1

### VM4 creada

### Prueba de newman con 4 procesos en paralelo

### Consumo de CPU de las 4 VMs

## Preguntas: Parte 2

**1. ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian? ¿Qué es SKU, qué tipos hay y en qué se diferencian? ¿Por qué el balanceador de carga necesita una IP pública?**

**2. ¿Cuál es el propósito del *Backend Pool*?**

**3. ¿Cuál es el propósito del *Health Probe*?**

**4. ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?**

**5. ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?**

**6. ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas? ¿Qué significa que una IP sea *zone-redundant*?**

**7. ¿Cuál es el propósito del *Network Security Group*?**

**8. Informe de newman 1 (Punto 2)**

---

# Diagrama de despliegue

---

# Conclusiones

---

# Limpieza de recursos

- [ ] Portal de Azure → *Resource groups* → `SCALABILITY_LAB` → **Delete resource group**

Esto elimina en una sola operación el Load Balancer, la IP pública, la Virtual Network, el NSG, las VMs, sus discos y sus NIC.
