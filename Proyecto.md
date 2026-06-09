
#### Referencias

- https://www.3gpp.org/technologies/5g-system-overview
- https://5g.systemsapproach.org/arch.html
- https://www.ijcsejournal.org/wp-content/uploads/2025/11/IJCSE-V8I1P2.pdf
- https://softwaremind.com/blog/why-open5gs-plays-an-important-role-in-open-source-private-5g-network-development/
- https://www.youtube.com/watch?v=YCkstHOlbWM

**Configuración Fase 2:

- https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins
- https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- https://github.com/s5uishida/open5gs_5gc_srsran_sample_config
- https://github.com/s5uishida/build_srsran_5g_zmq
- https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup

Objetivo : Instalar una red 5G operativa con srsRAN como mecanismo de acceso por radio y el core de red con servicios tipo NFV u otras opciones SDN.

# Documentación 
## Overview : 5G Technology

Esta nueva generación se destaca principalmente por su alta tasa de transmisión que permite (llegando incluso hasta 10Gbps)  y por la capacidad de soportar multiples conexiones con dispositivos finales (Hata 1 millon por metro cuadrado), lo que da pie a trabajar con demandas altas de sectores como :

- Iot
- Carros autodirigidos
- Smart cities
- Smart Homes

## Arquitectura 5G (Un pequeño resumen)

los estandares de 5G se  empezaron  a documentar desde el realease 15, en donde se definen protocolos e interfaces de red que abarcan  la tecnología : llamadas y control de session, manejo de mobilidad, provisión del servicio,etc.

![[Pasted image 20260421071600.png]]

La arquitectura base se compone de :

***UE (User Equipment) :***

Corresponde a el usuario final que se comunica a través de una interfaz llamada NR-Uu,  en ese orden de ideas podemos encontrar dispositivos como : 

- Telefonos celulares 
- Tablets
- Carros
- drones 
- Maquinas industriales
- Maquinaria médica

***Acceso a la red por Radio (RAN) :***

En este caso se realiza a través de la tecnología NG-RAN , y cumple la función de lo que conocemos como estación base o Node-B , la cual se divide en :

- gNB-CU (Central Unit) : Maneja RRC, PDCP , y la lógica de control
- gNB- DU (Distributed Units) : Maneja RLC, MAC y PHY (Lo que esta más cerca de la antena)

***5GC

El core 5G se compone de :

- UPF (User Plane Function):
- AMF (Access and Mobility Managment Function)
- NG es  el grupo de  interfaces a través de las cuales se comunica el componente de acceso por radio y el core.

La arquitectura del Core 5G se estructura en referencia al framework SBA (Service  Based Architecture) en donde los elementos se denominan en terminos de NFs (Network Functions)

![[Pasted image 20260421074303.png]]

Los NF's presentes en este esquema son :

- DN (Data network)
- AF -> Application Function , funciona en conjunto con el servicio de control de sesión (SMF) y el UPF
- SMF -> Session Management Function, Maneja llamadas y sesiones , en base a esto se comunica con el UPF.
- UDM -> Unified Data Managment , actua similar a el HSS(Home suscriber server) en 4G en donde se proveen servicios de suscribción a una entidad (IMSI ) , autenticación de suscriptores, etc.
- PCF -> Policy control function, controla el trafico del usuario y que este no exceda el limite negociado.
- NRF -> Network Repository Function, se encarga de recojer información acerca de los NF's, provisionando soporte de registro para nuevos NF's , actualización de servicios y eliminación de NF's. 
- NF's de seguridad :
	- SEAF -> Security Anchor Functionality
	- NEF -> Network Exposure Function
	- AUSF ->  Authentication Server Function
- NSSF -> Network Slice Selection Function 

#### IMSI

IMSI : Código que identifica a un suscriptor específico dentro de las redes móviles

Esta identidad se compone de tres partes clave:

- **MCC (Código de País Móvil):** Los primeros 2 o 3 dígitos que indican el país. 
    
- **MNC (Código de Red Móvil):** Los siguientes 2 o 3 dígitos que identifican al operador de telecomunicaciones. 
    
- **MSIN (Número de Identificación de Suscriptor):** La parte final que asigna el operador para identificar al usuario de forma única.

A diferencia del **IMEI** (que identifica el dispositivo físico) y el **ICCID** (que identifica la tarjeta SIM física), la IMSI identifica la **línea o servicio** del abonado, siendo fundamental para la autenticación en la red y el enrutamiento de llamadas, especialmente cuando se está en roaming internacional
### Interfaces que conectan todo


- NG-C (Control Plane entre gNB y AMF) : Usal el protocolo de aplicación NGAP
- NG-U (User Plane entre gNB y UPF): usa GTP-U sobre UDP/IP.
- Xn-C (Control plane entre gNBs) : SCTP sobre IP, protocolo XnAP. Se encarga de la transferencia de contexto o RAN paging)
- Xn-U(User plane entre gNBs) : usa GTP-U sobre UDP/IP.

![[Pasted image 20260510151240.png]]


>  ***Ahora si entramos en materia llendo abarcando a profundidad cada componente de la red 5G y como se relacionan entre sí cada subsistema para proveer el servicio.


### UE

El UE hace uso de las primeras 3 capas del modelo OSI, en las cuales maneja los siguientes protocolos respectivamente :

![[Pasted image 20260525183459.png]]

#### Capa de protocolos (Radio)

***Reiteración clave : Hay dos planos que corren en paralelo dentro de la red móvil 5G

- Plano de usuario (User Plane) : Lleva datos de usuario - video, páginas web, llamadas. Va desde la aplicación hasta internet.
- Plano de control (Control plane) : lleva señalización - Mensajes que la red y el UE intercambian para gestionar el acceso y la conexión a los servicios dentro de esta.(**NO LLEVA DATOS DE USUARIO)

EL Mecanismo de separación de estos dos planos se implementa por medio de un Bearer, que literalmente es una etiqueta que se le asigna a cada flujo para indicar a que plano pertenece y qué prioridad tiene.

y aquí hay dos tipos ya mencionados anteriormente : 

**SRB (Signaling Radio Bearer):

- El SRB0 : Mensaje RRC antes de que haya conexión
- El SRB1 : Mensaje RRC y NAS encapsulado en RRC.
- El SRB2 : Mensaje NAS de menor prioridad que SRB1.

**DRB (Data Radio Bearer): Transporta datos del plano de usuario


Cuando hablamos de la capa de protocolos que existen entre el gNB y el UE, tenemos el siguiente esquema:

![[Pasted image 20260511112807.png]]
Explicaremos desde arriba (Capas superiores) hacia abajo (Capas inferiores) el flujo entre UE y gNB.

**SDAP -> Capa que habla el idioma del core

Cuando el 5GC establece una sesión no simplemente nos da internet, si no que da uno o varios QoS flows, flujos de datos con características de calidad garantizada con latencia baja. Sin embargo, la capa de PDCP no entiende por QoS flows en cambio hace uso de DRBs, ya mencionados como etiqueta de datos del plano de usuario.

En resumen el SDAP gestiona una sesión de datos, cabe aclarar que si se tienen dos sesiones simultaneas (por ejemplo : Datos móviles e IMS para voz), habrían dos entidades SDAP.


***PDCP -> Capa que protege y comprime

Allí se suelen realizar tres procesos clave para la transmisión de paquetes  entre gnB y UE :

- Compresión : Un paquete de voz VoIP puede tener un header de aprox 40 bytes (IP + UDP +RTP), sin embargo de allí las cabeceras (IP + UDP + RTP (protocolo de transporte en tiempo real, opera sobre UDP)) equivalen a 20 bytes de este paquete y el payload de audio puede ser solo los 20 bytes restantes, por tanto, se esta gastando el doble de espectro simplemente en cabeceras y no en datos reales. 
	- El mecanismo de compresión es conocido como ROHC que comprime esos 401 bytes a unos 4, lo cual duplica la eficiencia efectiva del canal.
- Cifrado : Depende de el tipo de red.
	- NEA 0 -> sin cifrado, para redes de emergencia
	- NEA 2 
	- NEA 3
- Protección de integridad : Se calcula un MAC-I sobre cada PDU, si  el receptor recalcula el MAC-I y no coincide , el paquete se descarta (en el plano de usuario es opcional, pero en el de control es obligatorio)


***RLC -> Adapta el tamaño de las PDUs en base a el tamaño de bloque que puede transmitir la capa física , esto se específica en el RLC scheduler.

Modos :

**TM (Transparent Mode)**: el RLC no hace nada — pasa la PDU sin tocarla. Sin header, sin segmentación, sin control. Se usa para canales donde la simplicidad es crítica: el SRB0 para mensajes RRC antes de que haya conexión establecida, y los canales de broadcast como el BCCH que lleva el SIB1.

**UM (Unacknowledged Mode)**: puede segmentar PDUs grandes en fragmentos más pequeños y rensamblarlos en el receptor, pero si un fragmento se pierde, no lo retransmite. El receptor descarta la PDU incompleta. Se usa para VoIP porque en voz es peor recibir audio con 200ms de retardo por retransmisión que simplemente perder 20ms de audio — el oído humano tolera mejor la pérdida que el retardo.

**AM (Acknowledged Mode)**: segmenta, reensamble, y además implementa **ARQ (Automatic Repeat reQuest)**. El receptor envía status reports indicando qué PDUs llegaron y cuáles no. El transmisor retransmite las faltantes. Se usa para tráfico TCP — navegación web, descargas — donde perder un byte significa retransmitir todo el segmento TCP, lo cual es mucho más costoso.


**MAC -> Gestiona el acceso al medio

- Multiplexación : Define cuantos y que tipo de datos se envían en una sola transmisión.
- Scheduling : El sheduler del gNB decide cada slot , además : 
	- Reporta quien transmite
	- en que PRBS
	- El UE reporta : CQI (Channel quality indicator) que indica que tan buena es la señal y el BSR (Buffer status Report) que dice cuantos datos tiene el UE esperando en buffer para el uplink.
	- El scheduler define la prioridad entre las UEs y su prioridad de QoS.
- HARQ : Es el mecanismo de retransmisión a nivel físico, cuando un TB (Transport block) llega con errores, en lugar de descartarlo el receptor lo guarda en un buffer. Cuando llega la retransmisión, se combinan las dos versionespara decodificar correctamente lo que no pudo. (No retransmite desde cero)

**NAS -> Controla la selección PLMN , intercambiando información entre el UE y el Plano de control correspondiente a la identificación y autenticación.

**GW(Gateway) -> Es responsable de la creación y el mantenimiento de la interfaz virtual TUN .


#### Estados del UE y la señalización de control 

Un celular no está siempre activamente transmitiendo. La mayoría del tiempo está esperando — una notificación, una llamada, un mensaje. Si mantuviera una conexión de radio completa todo el tiempo gastaría batería constantemente y ocuparía recursos de red que otros UEs necesitan.

Los estados RRC son la solución: definen cuántos recursos mantiene activos el UE según lo que está haciendo. Piénsalo como los estados de una persona en una conversación telefónica.

**RRC_IDLE** es como estar dormido con el teléfono en silencio. El UE no tiene ningún contexto en la red — si apagan la celda a la que estaba conectado, no pasa nada. El UE monitorea periódicamente el canal de paging para ver si hay una llamada entrante, y entre esos momentos de escucha duerme completamente. La red solo sabe en qué **Tracking Area** está el UE — una zona geográfica que puede abarcar decenas de celdas.

**RRC_INACTIVE** es el estado más inteligente y es nuevo en 5G — no existía en LTE. Funciona como estar en pausa en una llamada: la conexión existe, el contexto está guardado en el gNB y en el UE, pero la radio está apagada. Cuando llega un dato, la reconexión es casi instantánea porque no hay que hacer todo el proceso de RACH y establecimiento de contexto desde cero. El UE pertenece a una **RNA (RAN Notification Area)** — un grupo de celdas — y solo avisa a la red si sale de esa zona. Esto es perfecto para IoT: un sensor que manda un dato cada 30 segundos no tiene sentido que mantenga una conexión completa, pero necesita reconectarse rápido cuando tiene algo que enviar.

**RRC_CONNECTED** es la conexión plena. El scheduler del gNB gestiona activamente los recursos del UE. La red sabe exactamente en qué celda está. Si el UE se mueve a otra celda, la red ejecuta un handover — el UE no decide, obedece.


#### El System Information: cómo la celda se presenta al mundo

El System Information es lo que la celda transmite continuamente sin que nadie lo pida — como un faro que emite constantemente para que los barcos sepan dónde está. Cualquier UE que esté cerca puede recibirlo sin estar conectado.

**El MIB** es el mensaje más pequeño y más crítico. Cabe en el PBCH dentro del SSB y lleva exactamente tres cosas útiles: el número de trama del sistema para que el UE se sincronice con el tiempo de la red, el bit `cellBarred` que indica si la celda acepta conexiones, y la configuración del **CORESET#0** que le dice al UE en qué frecuencias y con qué formato buscar el canal de control (PDCCH) para encontrar el SIB1. Sin CORESET#0 el UE no sabe dónde buscar nada más — es la dirección de la siguiente parada.

**El SIB1** es el directorio completo de la celda. Lleva el PLMN ID (qué operador es esta red), el TAC (Tracking Area Code, la zona geográfica), los parámetros completos del RACH para que el UE pueda intentar acceso, y el scheduling del resto de SIBs. A diferencia del MIB que va en un canal fijo (PBCH), el SIB1 va en el PDSCH con scheduling dinámico — por eso el UE necesita CORESET#0 para encontrar el PDCCH que le dice dónde está el SIB1 en ese slot.

**Los SIBs adicionales** son opcionales y específicos: SIB2 le dice al UE los criterios para reseleccionar celda (cuándo conviene cambiarse a otra), SIB3 y SIB4 le indican qué frecuencias medir para posible handover, SIB5 aplica cuando hay LTE en la misma red. Estos pueden transmitirse periódicamente o solo cuando un UE los solicita explícitamente — eso ahorra capacidad de broadcast cuando nadie los necesita


## RAN

### Capa física

En este subsitema de la arquitectura 5G se realizan principalmente los procesos de :

- Manejo del espectro y uso eficiente de este por medio de los  componentes de radio
- Requerimientos Qos

#### Interfaces de radio

- Para el downlink , es decir, para el NR que conecta a la red de el UE se usa OFDM con Cyclic Prefix (CP) y para el Uplink (UE hacia la RAN) se hace uso de DFT-s-OFDM

***OFDM : Usa diferentes subportadoras espaciadas  entre sí en el dominio de la frecuencia   para transportar datos, teniendo como restricción mátematica que entre las portadoras "vecinas" se cumpla el principio de ortogonalidad, esto permite eliminar la interferencia entre portadoras.

![[Pasted image 20260506221021.png]]

Para Uplink : Hace uso de SC-FDMA
Para Downlink : Hace use de OFDMA

**Diferencias entre FDMA Y OFDMA

![[Pasted image 20260506221141.png]]

- En OFDM las subportadoras estan sobrepuestas entre sí 
- Es más eficiente espectralmente OFDM
- en FDMA tenemos una "Guard band" para asegurarnos de que las subportadoras no se sobrepongan.


**Representación de OFDM en el dominio del tiempo y domio de la frecuencia

![[Pasted image 20260506221526.png]]

El espacio entre subportadoras esta definido por el termino "subcarrier spacing" el cual equivale a $\delta f$ .  

Cada subportadora tiene una frecuencia central indicada entre $f_i$  y $f_s$  , en donde el periodo $T_u$ es el mismo para cada una.
![[Pasted image 20260506222308.png]]

**OFDM Y OFDMA

![[Pasted image 20260506224403.png]]


**Cyclic Prefix

El CP es una copia del final del símbolo que se pega al principio. Tiene una duración fija en tiempo. Cuando el retardo multipath del canal es menor que la duración del CP, el receptor puede ignorar la interferencia completamente — es como si el símbolo hubiera llegado "limpio".

El problema es que el CP consume tiempo útil sin llevar datos. Si el símbolo dura 66.7 µs y el CP dura 4.7 µs, estás gastando ~7% del tiempo en "overhead". Y ese porcentaje es fijo respecto al símbolo — cuando reduces el SCS a la mitad, el símbolo se hace más largo y el CP también se escala, lo que te da mayor tolerancia al multipath.

![[Pasted image 20260506224300.png]]
![[Pasted image 20260506223220.png]]

Cuando transmites datos por radio, el canal no es ideal. Existe un fenómeno llamado **dispersión multipath**: la señal sale de la antena, rebota en edificios, el suelo, objetos, y le llega al receptor por múltiples caminos con distintos retardos. El receptor recibe la misma señal varias veces pero desalineada en tiempo. Esto se llama **Inter-Symbol Interference (ISI)** — los símbolos se solapan entre sí y se vuelven indistinguibles.

La solución clásica era usar un solo símbolo muy largo para que el retardo multipath sea despreciable comparado con la duración del símbolo. Pero eso limita la velocidad.

OFDM resuelve esto de manera elegante: en lugar de transmitir en una sola portadora, divide el canal en **cientos o miles de subportadoras angostas** en paralelo, cada una transmitiendo a velocidad baja. Para protegerlas de la ISI, agrega un **Cyclic Prefix (CP)** al inicio de cada símbolo — una copia del final del símbolo que actúa como buffer temporal. Mientras el retardo multipath sea menor que la duración del CP, no hay ISI.

#### Numerología o Portadoras de frecuencia

El espacio entre subportadoras es definido de la siguiente manera por el 3GPP $\delta f = 2^ µ × 15 kHz$ 

![[Pasted image 20260506225438.png]]

- A mayor µ, menor duración del símbolo, lo que reduce latencia pero requiere mayor ancho de banda de procesamiento.
- Un **PRB (Physical Resource Block)** siempre tiene 12 subportadoras consecutivas. Hasta 275 PRBs son soportados en un carrier.
	- Dentro de un PRB hay tres conceptos anidados que no debes confundir:

	**Resource Element (RE)**: la celda más pequeña de la grilla. Es la intersección de exactamente una subportadora con exactamente un símbolo OFDM. Lleva un número de bits que depende del esquema de modulación — con QPSK lleva 2 bits, con 64-QAM lleva 6 bits, con 256-QAM lleva 8 bits.
	
	**PRB**: el bloque de 12 subportadoras consecutivas. Tiene 12 × 14 = 168 REs por slot. Esta es la unidad que el scheduler asigna — nunca asigna subportadoras sueltas.
	
	**Resource Grid**: el conjunto de todos los PRBs del carrier en un slot. Con 100 MHz y µ=1 por ejemplo tienes 66 PRBs disponibles. El carrier puede tener hasta 275 PRBs según el estándar.
	
	***Por qué 12 y no otro número ?
	
	No es arbitrario. Tiene que ver con la transformada de Fourier que se usa para generar la señal OFDM. La IFFT que convierte el dominio frecuencia al dominio tiempo trabaja más eficientemente con potencias de 2, y 12 es el número que balancea entre granularidad de asignación y overhead de señalización. Viene heredado de LTE, donde también se usa.

- Las portadoras van desde 400 MHz hasta 100GHz
- Las licenciadas van desde 600 MHz hasta 39 GHz

Para despliegues terrestres :

- 1GHz : Mejora en la propagación , puede cubrir grandes areas, por ende es apto para implementaciones rurales .**El ancho de banda maximo para una portadora es de 100 M Hz**
- 1 - 6 GHz : Ideal para implementaciones urbanas o sub urbanas. **El ancho de banda por portadora es de 100 MHz**
- + 6 GHz : La capacidad de propagación es baja pero el ancho de banda disponible es alto, contando con **más de 400 MHz por portadora**


	 Cuando un dispositivo móvil entra en una zona que nunca antes había visitado...
	- No sabe en que frecuencia ésta la celda 
	- No sabe que hora es en términos de tramas de radio
	- No sabe a que celda conectarse

Aquí es donde el SSB cumple un rol importante, el cual se provee a través de un bloque de 4 símbolos OFDM que la **celda** transmite periódicamente.
#### SSB (Synchronization Signal Block)


Es lo primero que el UE debe encontrar para acceder a la red. Contiene:

- PSS (Primary Synchronization Signal) y SSS (Secondary Synchronization Signal) : Estos bloques se encargan de sincronizar el tiempo/frecuencia y detección del PCI (Physical Cell ID).
	- El PSS es una secuencia mátematica predefinida (hay 3 posibles secuencias : #0, #1 , #2). El UE inicialmente no sabe cual se èsta usando en la celda así que prueba las tres correlativamente sobre la señal recibida (Sincronización temporal).
	- Cuando el UE identifica la secuencia se le da un valor que es uno de los componentes del PCI  N_ID (2) , y de hecho aquí el UE puede estimar el offset de frecuencia entre su oscilador local y el de la celda.
	- Este proceso o bloque de PSS vive exclusivamente en el gNB (RAN).
	- Después de que el UE encontró el PSS, sabe donde buscar el SSS, que es una secuencia de longitud 127 y hay 336 posibilidades de secuencias.
	- El UE correlaciona la señal recibida contra estas 336 posibilidades y obtiene el índice N_ID (1) .
	- ya teniendo estos dos indices se puede formar el PCI completo
		$PCI = 3 × N_{ID}^1 + N_{ID}^2$

***El resultado es un número entre 0 y 1007 como PCI. Su importancia se resume en : Un gNB puede tener varias celdas (por sectores o por bandas diferentes), y cada una tiene su propio PCI. El PCI importa por dos razones: primero, determina las secuencias de referencia que usa esa celda para que el UE pueda demodular la señal correctamente. Segundo, se usa para planificación de red — dos celdas vecinas no pueden tener el mismo PCI porque causarían confusión en el UE durante la medición de vecinas. Es un parámetro que el operador asigna durante el despliegue.***



- PBCH (Physical Broadcast channel) : Su función es darle a la UE la información necesaria para seguir el proceso de acceso.
	- Allí se comparte el MIB, que contiene específicamente el número de trama (SFN) y un bit llamado `cellBarred` que indica si la celda esta disponible o bloqueada para el acceso.

	 El número de SSBs que se transmiten dentro de una misma trama depende de la banda:
		- FR1 ≤ 3 GHz: hasta 4 SSBs
		- FR1 > 3 GHz: hasta 8 SSBs
		- FR2: hasta 64 SSBs


- RMSI y CD-SSB
	- El RMSI es tambien conocido como el S1B1 y contiene los parametros del RACH (Información mínima para que la celda emita broadcast y así los usuarios se puedan conectar), parametros del RACH :
		- PLMN ID
		- Tracking Area
		- Schedulling del resto de los SIBs.
- CD-SSB: Esta asociado a un S1B1, en 5G una celda puede transmitir múltiples SSBs apuntando en distintas direcciones (beam sweeping), pero solo uno de esos SSBs — el CD-SSB — tiene un SIB1 asociado. Los demás SSBs sirven para cobertura direccional pero no definen la celda en términos de acceso inicial. Cuando el UE detecta el CD-SSB, sabe que puede buscar el SIB1 en los recursos indicados por el PBCH/MIB. Si detecta un SSB que no es el CD-SSB, no tiene SIB1 directo y no puede iniciar el acceso por ese SSB.

Para el bloque siguiente ya el UE debe tener el SSB, el MIB y el S1B1 (para llamar a la puerta), sin embargo , en este punto el  UE no existe así que no puede asignarle recursos.

#### RACH : El primer contacto del UE con la red

![[Pasted image 20260507084808.png]]

**Fuente : Generación con Claude (IA)

El Random Access Channel es el mecanismo por el cual el UE accede a la red sin recursos asignados, el proceso se resume en los siguientes pasos:

- MSG1 : El UE transmite un preamble RACH en recursos PRACH (Sin coordinación y con riesgo de colisión)
- MSG2 : La red responde con un RAR (Random Access response)
- MSG3  : El UE transmite su identidad en los recursos indicados
- MSG4: La red resuelve la contención y asigna C-RNTI permanente

***Terminos de los MSG

##### TA — Timing Advance

El canal de radio tiene un problema de física básica: la señal tarda tiempo en viajar. Un UE a 10 km del gNB tardará ~33 microsegundos en que su señal llegue. Si el gNB tiene múltiples UEs transmitiendo en uplink "al mismo tiempo", pero cada uno a distinta distancia, las señales llegarán desalineadas y se interferirán.

El TA es un valor de corrección que el gNB le envía al UE en el MSG2 diciéndole: "transmite tus símbolos UL con este adelanto de tiempo, de modo que cuando lleguen al gNB estén perfectamente alineados con los de los demás UEs". Es un ajuste en tiempo expresado en unidades de Tc (la unidad mínima de tiempo en 5G). Sin el TA no es posible la comunicación coherente en uplink.

##### UL Grant

Un Grant es una autorización de transmisión en uplink. El gNB le dice al UE exactamente en qué recursos de frecuencia y tiempo puede transmitir el MSG3. Contiene: los PRBs asignados, el MCS (esquema de modulación y codificación) a usar, y la redundancy version del HARQ. Sin este Grant el UE no sabe dónde transmitir su identidad — si transmitiera sin coordinación en cualquier frecuencia, colisionaría con otros UEs.

##### TC-RNTI — Temporary Cell Radio Network Temporary Identifier

Aquí está el concepto de identidad en radio. En 5G, cuando el gNB quiere hablar con un UE específico en la interfaz de radio, no puede gritar su número de teléfono — eso desperdiciaría espectro y violaría privacidad. En cambio, asigna a cada UE un identificador numérico de 16 bits llamado RNTI. Todo mensaje del gNB en downlink va etiquetado con un RNTI, y el UE decodifica solo los mensajes dirigidos a él.

El **TC-RNTI (Temporary Cell RNTI)** es una identidad provisional que el gNB asigna en el MSG2. Es temporal porque todavía no se sabe si el acceso tendrá éxito — puede haber colisión. El UE usa el TC-RNTI para identificar el MSG4 que le corresponde. Si el MSG4 confirma que el acceso fue exitoso para ese UE específico, el TC-RNTI se promueve a **C-RNTI permanente**.

##### C-RNTI permanente

Es la identidad definitiva del UE en esa celda durante toda la conexión RRC_CONNECTED. Cada UE activo en una celda tiene un C-RNTI único entre 0 y 65535. El gNB lo usa para:

- Dirigir los DCI (Downlink Control Information) en el PDCCH — el UE monitorea el PDCCH buscando mensajes con su C-RNTI
- Identificar los recursos de uplink que el UE puede usar
- Referenciar el contexto del UE en toda la lógica del scheduler

Cuando el UE hace handover a otra celda, el C-RNTI cambia — es local a la celda, no global. Cuando el UE cae a RRC_IDLE, el C-RNTI se libera. Cuando vuelve a conectarse, hace otro RACH y recibe uno nuevo.

***Parametros configurables del RACH (Importante)

- **prach-ConfigurationIndex**: define la configuración de tiempo/frecuencia de las ocasiones RACH
- **preamble format**: determina longitud de secuencia (139, 571, 839, 1151) y subcarrier spacing del PRACH
- **ra-ResponseWindow**: tiempo en slots que el UE espera el MSG2
- **preambleTransMax**: número máximo de intentos antes de declarar fallo
- **powerRampingStep**: incremento de potencia entre intentos (en dB)
- **msg3-transformPrecoding**: habilita DFT-spreading en MSG3

### gNB-CU (Central Unit) : Maneja RRC, PDCP , y la lógica de control

### gNB- DU (Distributed Units) : Maneja RLC, MAC y PHY (Lo que esta más cerca de la antena)

## Especificaciones de redes 5G

**Network Slicing**

- **Concepto:** Es la capacidad de instanciar múltiples **Redes Lógicas** sobre una única **Infraestructura Física**.
    
- **Fundamento Técnico:** Cada "Slice" (rebanada) es un conjunto completo de funciones de red (NF) configuradas para un propósito específico. El NSSF (Network Slice Selection Function) es el encargado de dirigir al usuario hacia la rebanada adecuada según sus credenciales y requerimientos (ej. latencia, ancho de banda, seguridad).
    
- **Impacto:** Permite garantizar **SLA (Service Level Agreements)** distintos. Por ejemplo, una rebanada de red para servicios críticos de salud (baja latencia garantizada) puede coexistir con una rebanada de red para consumo de video masivo (alta capacidad), sin que el tráfico de una interfiera con la otra.

**Network Function Virtualization (NFV)**

- **Concepto:** Desacoplamiento del software (las funciones de red como AMF, UPF, SMF) respecto del hardware propietario.
    
- **Fundamento Técnico:** En 5G, las funciones de red se diseñan como **Microservicios** que se ejecutan en contenedores (ej. Docker/Kubernetes). Al comunicarse a través de interfaces comunes basadas en servicios (SBA - Service Based Architecture), estas funciones pueden residir en servidores genéricos (_COTS - Commercial Off-The-Shelf_), ya sea en la nube central o en el borde.
    
- **Impacto:** Flexibilidad extrema. Si una función de red colapsa o necesita más recursos, el orquestador puede desplegar una nueva instancia en segundos. Esto simplifica drásticamente el mantenimiento y la actualización de la red.

**Edge Computing**

- **Concepto:** Descentralización del procesamiento de datos, moviendo la capacidad de cómputo hacia el **Borde (Edge)** de la red, lo más cerca posible del usuario (cerca de la gNB).
    
- **Fundamento Técnico:** Históricamente, todos los datos viajaban hasta un Core centralizado. En aplicaciones de latencia crítica (ej. realidad aumentada, vehículos autónomos), el tiempo de ida y vuelta al Core central es inaceptable. Con Edge Computing, el **UPF (User Plane Function)** se despliega cerca de la antena, permitiendo que el tráfico local termine allí mismo sin tener que atravesar toda la red de transporte hasta el núcleo central.
    
- **Impacto:** Reducción drástica de la latencia y ahorro de ancho de banda en la red troncal (_backbone_), al evitar que el tráfico local sature los enlaces centrales.


## 5GC

Tiene la responsabilidad de : 

- Autenticar los dispositivos que se conectan por medio del enlace por radio
- Proveer Internet para servicios de voz y datos
- Garantizar que los requerimientos de QoS se cumplen.
- Registrar la mobilidad de el usuario final para garantizar la no interrupción del servicio
- Revisar el tráfico en base a las politicas de uso del servicio.


Al tratarse de una implementación de core basada en SDN , tendremos una separación importante de el 5GC :

- User Plane
- Control Plane


Open5GS es una herramienta de software que nos permitira implementar el core de 5G y sus respectivas funcionalidades como :

- **AMF (Access and Mobility managment Function)** : Es el responsable de mantener la subscribción del usuario mientras esta en movimiento, el equivalente a MME (Mobility Managment Entity) en el core de 4G.
- **SMF (Session Managment Function)** : Maneja la sesión entre el usuario y la red 5G.Además implementa mecanismos de control de trafico.
- **UPF**: Facilita la transmisión de paquetes entre el gNodeB y la red. Este interactua directamente con SMF para registrar y mantener la sesión de usuario , además de garantizar una transferencia de datos que concuerda con los requerimientos de QoS.
- **AUSF**: autentica el usuario dentro de la red 5G, verifica su identidad y da la autorización para hacer uso de la red.
- **UDM(Unified Data Managment)**: Maneja la subscribción de datos a la red 5G, provee al SMF información acerca de los perfiles de servicios, preferencias, politicas , entre otra información necesaria para establecer manejar y controlar las sesiones del subscriptor.
	- Tambien provee datos necesarios a amf  como credenciales, perfil del subscriptor, etc.
	- Es el equivalente a HSS (Home Subscriber Server) para redes 4G.
- **PCF(Policy and charging Function)**: Mantiene la sesión bajos las politicas establecidas de :
	- Priorización de trafico
	- Manejo de ancho de banda
	- Limitación de velocidad
	- Equivalente a PCRF en 4G
- **SCP(Service Communication Proxy):** Maneja la comunicación entre las NF's y las aplicaciones o servicios externos (Aplicaciones que corren sobre el UPF, ej : Acceso a un CDN con Kiwix).
- **NRF (NF Repository Function)**: Almacena la información de los NF's, actua como recurso central de información de los elementos de la arquitectura 5G SA.
- **NSSF(Network Slice Function)** : El NSSF permite la virtualización de redes físicas para ofrecer diferentes SLAs (Service Level Agreements) sobre el mismo hardware físico".
	- **BSF (Binding Support Function)**: El BSF es un "gestor de contexto" esencial cuando un usuario tiene múltiples sesiones activas. La BSF garantiza la integridad del plano de control de políticas, asegurando que todas las solicitudes de una misma sesión de usuario sean servidas por la instancia correcta del PCF, evitando inconsistencias en la gestión de calidad (QoS) y tarificación


Las opciones de despliegue del core , se resume en arquitecturas Standalone y non-standalone.

#### Standalone

Para que una arquitectura sea **5G Standalone**, la condición definitiva es que el dispositivo (UE - User Equipment) **no necesite depender de una red 4G (EPC) para ninguna función de control, señalización o plano de usuario.**


- El EPC (Evolved Packet Core) de 4G fue diseñado para manejar señalización basada en protocolos que no entienden las nuevas capacidades de 5G (como el _Network Slicing_ o la latencia ultra baja del URLLC).
    
- Al cambiar al 5GC, obtienes una **arquitectura basada en servicios (SBA - Service Based Architecture)**, que es lo que realmente permite las promesas del 5G.

Funcionalidades principales:

- CUPS : Separación del plano de control del plano de usuario.
- Soporte Multi-Gigabit
- URLLC
- Consumo energetico bajo


#### Non-Standalone

En esta modelo, el core de 5G es desplegado a lo largo de infraestructura existente de 4G LTE , por tanto el core es basado en el core de 4G (EPC ) .

![[Pasted image 20260422112716.png]]

En ese orden de ideas, el RAN 5G es compatible para conectarse a el core de 4G (EPC) en este arquitectura, sin embargo, el eNB es el que ejecuta las funcionalidades del plano de control , por ende la comunicación entre el RAN 5G y 4G LTE core debe ser mediada por el eNB.

El 5G RAN cumple la función de plano de usuario ,además del acceso principal de este a el servicio.


***Ventajas:

- La relación costo-beneficio  es bastante buena ya que la red 5G es desplegada en infraestructura existente.
- Es de facil despliegue, porque el sector teleco ya esta familiarizado con la tecnología 4G.

***Desventajas:

- La latencia puede variar debido a la infraestructura de 4G LTE presente.
- Hay más consumo de energía al implementarse con dos variantes de red celular (eNB y gNB)
- Las expectativas para los servicios que ofrece 5G  puro no son exactamente satisfechas por esta arquitectura.



***Arquitecturas Hibridas NSA y SA

![[Pasted image 20260422115902.png]]


#### Pila de protocolos

En 5G, la comunicación se divide en dos planos fundamentales: el **Plano de Control (Control Plane)**, que gestiona "las reglas" de la conexión, y el **Plano de Usuario (User Plane)**, que transporta "los datos reales".

##### **1. Plano de Control (Control Plane): Gestión y Señalización**

Su función es que la red sepa quién eres, dónde estás y qué permisos tienes. Los protocolos aquí no transportan el tráfico de internet, sino las instrucciones para establecerlo.

- **NAS (Non-Access Stratum):** Es la señalización directa entre el UE (móvil) y el Core (AMF/SMF). Se llama "Non-Access" porque es transparente para la antena (gNB); la antena solo actúa como un puente.
    
    - **NAS-MM (Mobility Management):** El "portero". Gestiona el registro del usuario, la movilidad y la seguridad (cifrado).
        
    - **NAS-SM (Session Management):** El "administrador de servicios". Gestiona las sesiones PDU (ej. crear el túnel para que el móvil acceda a la red).
        
- **NG-AP (NG Application Protocol):** El lenguaje que usa la antena (gNB) para hablar con el AMF. Es el protocolo de aplicación en la interfaz **N2**.
    
- **SCTP (Stream Control Transmission Protocol):** El protocolo de transporte para el plano de control. A diferencia de TCP, permite múltiples flujos de datos sin bloqueos, garantizando que si un mensaje de señalización se pierde, se retransmita (fiabilidad).
    

> **Concepto clave:** La comunicación **N2 SM information** es una "traducción" donde el AMF empaqueta información del SMF dentro de mensajes NG-AP para que la radio la entienda, manteniendo la independencia entre capas.


![[Pasted image 20260423074742.png]]

##### **2. Plano de Usuario (User Plane): Transporte de Datos**

Su función es entregar paquetes IP o tramas Ethernet desde el móvil hasta la red de datos (DN - Data Network).

- **PDU Layer (Protocol Data Unit):** Es el tráfico "puro" del usuario. Si navegas en Google, aquí viajan tus paquetes IPv4/v6. Es la carga útil final que la red debe entregar.
    
- **GTP-U (GPRS Tunnelling Protocol - User Plane):** Es el "contenedor" universal. El 5G encapsula los paquetes del usuario dentro de túneles GTP para transportarlos a través de la infraestructura interna (interfaces **N3** y **N9**).
    
    - **Marcado QoS:** El GTP-U no solo transporta el dato; incluye una marca (_QoS Flow Identifier_) que le indica a la red qué prioridad debe darle a ese paquete específico.
        
- **5G-AN Protocol Stack (L1/L2):** Las capas de radio (Capa Física y Enlace). Es el hardware de la red procesando las ondas de radio.
    
- **UDP/IP:** La infraestructura física de transporte (backbone). A diferencia del plano de control que usa SCTP, el plano de usuario usa UDP porque prioriza la velocidad sobre la retransmisión estricta (para evitar retardos en servicios sensibles a latencia).
![[Pasted image 20260423074856.png]]



# Simulación 

**Fase 2:
***Objetivos : 

- Implementar Core y Red de acceso 
- Implementar software de mensajería NAS/AS

**Fase 3:
***Objetivos :

- Integrar SDR (B200)
- Integrar Terminal comercial

## Fase 2 

### Diseño de red (Maquinas virtuales, segmentación, componentes y tecnologías y automatización con vagrant)

***Red entre VMs

La red Host-Only (`192.168.56.0/24`) es completamente interna a VirtualBox. Existe independientemente de si el PC anfitrión está conectado a WiFi, cable, otra red, o sin Internet. VirtualBox crea un switch virtual (`vboxnet0`) que vive dentro del hypervisor y conecta las VMs entre sí. 

Enlace a el digrama : https://lucid.app/lucidchart/f6828243-fb41-413b-873a-08dc2334c561/edit?viewport_loc=-5049%2C-2735%2C4797%2C2475%2C0_0&invitationId=inv_24b99a44-c534-4288-a6c9-a94d0a334f9d

![[Diagrama-virtual-siscomII.jpeg]]

La creación de dichas maquinas y su configuración de red individual para cada una sera realizada con la herramienta Vagrant la cual nos permite automatizar estos procesos.

**VagrantFile

```yaml
# =============================================================================

# Vagrantfile — Simulación 5G SA

# Proyecto: Open5GS + srsRAN Project (gNB) + srsRAN 4G (srsUE)

#

# Este archivo SOLO define:

# - Recursos de cada VM (RAM, CPU)

# - Interfaces de red (NAT + Host-Only con IP estática)

#

# Todo lo demás (instalaciones, configuraciones, compilaciones)

# se hace manualmente dentro de cada VM.

#

# Topología:

# VM1 — Open5GS — 192.168.56.111

# VM2 — srsRAN gNB + UE — 192.168.56.131

#

# Uso:

# vagrant up — crea las 2 VMs

# vagrant ssh vm1 — accede a VM1

# vagrant ssh vm2 — accede a VM2

# vagrant halt — apaga sin destruir

# vagrant destroy -f — elimina todo

# =============================================================================

  

Vagrant.configure("2") do |config|

  

# Box base: Ubuntu 22.04 Server (sin desktop)

config.vm.box = "ubuntu/jammy64"

config.vm.box_check_update = false

  

config.vm.provider "virtualbox" do |vb|

vb.gui = false

vb.linked_clone = true

end

  

# ---------------------------------------------------------------------------

# VM1 — Open5GS Control Plane + User Plane

# Componentes a instalar manualmente: NRF, SCP, AMF, SMF, AUSF, UDM, UPF

# ---------------------------------------------------------------------------

config.vm.define "vm1" do |vm1|

vm1.vm.hostname = "open5gs-cp"

  

# Adaptador 1: NAT (automático) — Internet y apt install

# Adaptador 2: Host-Only — comunicación entre VMs

vm1.vm.network "private_network",

ip: "192.168.56.111",

name: "vboxnet0"

  

vm1.vm.provider "virtualbox" do |vb|

vb.name = "5G-VM1-CP"

vb.memory = 2048

vb.cpus = 2

end

  

# Solo configura la IP estática de eth1 — nada más

vm1.vm.provision "shell", inline: <<-SHELL

cat > /etc/netplan/99-5g-lab.yaml << 'EOF'

network:

version: 2

ethernets:

enp0s8:

dhcp4: false

addresses:

- 192.168.56.111/24

EOF

netplan apply

echo "[VM1] Red configurada. IP Host-Only: 192.168.56.111"

SHELL

end

  
  

# ---------------------------------------------------------------------------

#vm2 — srsRAN Project (gNB) + srsRAN 4G (srsUE)

# Compilar manualmente: srsRAN_Project y srsRAN_4G

# ZeroMQ entre gNB y srsUE usa 127.0.0.1 (loopback, misma VM)

# N3 GTP-U hacia VM2 usa 192.168.56.131 (Host-Only)

# ---------------------------------------------------------------------------

config.vm.define vm2" do vm2|

vm2.vm.hostname = "srsran"

  

vm2.vm.network "private_network",

ip: "192.168.56.131",

name: "vboxnet0"

  

vm2.vm.provider "virtualbox" do |vb|

vb.name = "5Gvm2-RAN"

vb.memory = 4096

vb.cpus = 5

end

  

vm2.vm.provision "shell", inline: <<-SHELL

cat > /etc/netplan/99-5g-lab.yaml << 'EOF'

network:

version: 2

ethernets:

enp0s8:

dhcp4: false

addresses:

- 192.168.56.131/24

EOF

netplan apply

echo "vm2] Red configurada. IP Host-Only: 192.168.56.131"

SHELL

end

  

end
```

Algunas de las tecnologías que han surgido y que se mantienen fucnionales en la actualidad, nos ayudaran a simular los componentes de una red celular 5G (UE, RAN, 5GC), a continuación se muestra una table que compara las tecnologías de software en relación a las funcionalidades de la tecnología 5G que incluyen :

![[Pasted image 20260509170509.png]]

Por temas de simplicidad y compatibilidad, haremos uso de dos tecnologías especificas :

#### Open5Gs

Es una implementación de código abierto del core de 5G, orientada a cumplir las funciones primordiales de red central con alta eficiencia, brindando compatibilidad con 5G Standalone, además permite su despliegue en equipos de hardware con bajos recursos. Posee una comunidad activa y soporte frecuente de actualizaciones , además de disponer paquetes precompilados y una interfaz web opcional para administración.


#### SrsRAN

Es una suite de código abierto centrada en la implementación del componente de RAN . Incluye herramientas para despliegue de eNodeB (4G) , gNodeB (5G) y también una simulación del UE en software. El soporte para 5G inicialmente era en modo NSA, sin embargo en 2022 con srsRAN project surgió el soporte para la implementación con 5G Stand-Alone completa, soportando gNodeB y UE 5G sin necesidad de ancla LTE.


### Instalación de cada uno de los programas (srsUE, srsRAN  ,Open 5GS)


#### srsUE

La solución disponible para el despliegue del proyecto 5G SA por parte de ***srsRAN Project*** no incluye el prototipo del UE, sin embargo , srsRAN 4G si lo incluye y por ende vamos a instalar esta herramienta en especifico para nuestra configuración con ZMQ.

***Limitaciones del UE (Según la documentación de srsRAN) :

- Limited to 15 kHz Sub-Carrier Spacing (SCS), which means only FDD bands can be used.

- Limited to 5, 10, 15 or 20 MHz Bandwidth (BW)

- Handover is not supported


***Instalar librerías necesarias

```
sudo apt-get install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev
```

Posteriormente debemos instalar la librería de ZMQ , el cual nos permitira emular la comunicación radio entre gnB y UE , por medio de muestras de radio que viajan a través de la interfaz de loopback.

**Instalación ZeroMQ

```
sudo apt-get install libzmq3-dev
```


**Descargar y construir srsRAN 4G

```
git clone https://github.com/srsRAN/srsRAN_4G.git
cd srsRAN_4G
mkdir build
cd build
cmake ../ -DENABLE_RF_PLUGINS=OFF
make
make test
```


En un setup donde el UE simulado y el gNB corren en la misma máquina que tiene acceso directo a la red donde vive el 5GC, el kernel de Linux podría enrutar el tráfico del UE directamente por la interfaz física (`eth1`) hacia el core, saltándose completamente el stack radio simulado (ZeroMQ → gNB → GTP-U → UPF). El network namespace `ue1` resuelve esto aislando el proceso srsUE en un entorno de red completamente separado donde la única interfaz visible es `tun_srsue` — obligando a que todo el tráfico del UE atraviese el camino correcto y completo de la red 5G simulada, lo que garantiza que las mediciones y verificaciones reflejen el comportamiento real del sistema.

Los pasos de creación del namespace son : 

- `sudo ip netns add ue1` : Crea el namespace
- `sudo ip netns list` : Lista los namespace creados


**IMPORTANTE : Cada que la VM se reinicie se debe volver a crear el namespace o automatizar su creación.

***Obtener archivo de configuración con ZMQ

```
wget https://docs.srsran.com/projects/project/en/latest/_downloads/fbb79b4ff222d1829649143ca4cf1446/ue_zmq.conf
```

![[Pasted image 20260513233212.png]]


#### srsRAN

**Instalar srsRAN Project

```
git clone https://github.com/srsRAN/srsRAN_Project.git

sudo apt install -y libyaml-cpp-dev
```

**Crear el codigo base :

```
cd srsRAN_Project
mkdir build
cd build
cmake ../ -DENABLE_EXPORT=ON -DENABLE_ZEROMQ=ON
make -j $(nproc)
make test -j $(nproc)
```

**Obtener archivo de configuración gnB

```
wget https://docs.srsran.com/projects/project/en/latest/_downloads/a7c34dbfee2b765503a81edd2f02ec22/gnb_zmq.yaml
```

![[Pasted image 20260511210234.png]]

#### Open5GS y WebUI

**Obtener MogoDB

```
$ sudo apt update
$ sudo apt install gnupg
$ curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
```

```
$ echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

**Instalar paquetes de MongoDB

```
sudo apt update
	sudo apt install -y mongodb-org
```


**Añadir el repositorio para OPEN5GS

```
sudo add-apt-repository ppa:open5gs/latest

sudo apt update

sudp apt install open5gs


```


***Instalación del WebUI


La WebUI es la interfaz gráfica de administración de Open5GS. Su función principal  es **registrar el suscriptor** — es decir, darle a la red 5G los datos del UE simulado para que pueda autenticarlo cuando intente conectarse.

```
# Download and import the Nodesource GPG key
$ sudo apt update
$ sudo apt install -y ca-certificates curl gnupg
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

# Create deb repository
$ NODE_MAJOR=20
$ echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

# Run Update and Install
$ sudo apt update
$ sudo apt install nodejs -y
```

**Instalar dependencias para ejecutar el WebUI

```
$ cd webui
$ npm ci
```

### Configuración de los servicios

#### srsUE


```
[rf]
freq_offset = 0
tx_gain = 50
rx_gain = 40
srate = 23.04e6
nof_antennas = 1
device_name = zmq
device_args = tx_port=tcp://127.0.0.1:2001,rx_port=tcp://127.0.0.1:2000,base_srate=23.04e6

[rat.eutra]
dl_earfcn = 2850
nof_carriers = 0

[rat.nr]
bands = 3
nof_carriers = 1
nof_prb = 106

[pcap]
enable = none
mac_filename = /tmp/ue_mac.pcap
mac_nr_filename = /tmp/ue_mac_nr.pcap
nas_filename = /tmp/ue_nas.pcap

[log]
all_level = info
phy_lib_level = none
all_hex_limit = 32
filename = /tmp/ue.log
file_max_size = -1

[usim]
mode = soft
algo = milenage
opc  = E8ED289DEBA952E4283B54E88E6183CA
k    = 465B5CE8B199B49FAA5F0A2EE238A6BC
imsi = 001010000000000
imei = 353490069873319

[rrc]
release = 15
ue_category = 4

[nas]
apn = internet
apn_protocol = ipv4

[gw]
netns = ue1
ip_devname = tun_srsue
ip_netmask = 255.255.255.0

[gui]
enable = false
```



#### srsRAN

ZeroMQ no es una interfaz de red. Es una librería de mensajería que crea sockets TCP internos dentro del sistema operativo. Cuando el gNB y el srsUE se comunican por ZeroMQ usando `127.0.0.1:2000` y `127.0.0.1:2001`, ese tráfico nunca sale de la VM3 — viaja por el loopback interno del kernel de Linux, igual que cuando dos programas en tu PC se comunican por localhost.


```

cu_cp:
  amf:
    addr: 192.168.56.111
    bind_addr: 192.168.56.131

```

- `addr` es la IP  donde está escuchando el AMF y el gNB establece la conexión NGAP hacia esta dirección. Por tanto esta debe coincidir con la ip definida para la VM1 (5G-CP)
- `bind_addr` : Es la interfaz a través de la cual sale el tráfico hacia el AMF , es decir , nuestra propia maquina 

```
supported_tracking_areas:
  - tac: 1
    plmn_list:
      - plmn: "00101"
        tai_slice_support_list:
          - sst: 1
```

- `tac` es el Tracking Area Code. Debe coincidir exactamente con el TAC configurado en el AMF de Open5GS (`amf.yaml`). Si no coinciden, el AMF rechaza la conexión del gNB con causa "TA not supported".
- **`plmn: "00101"`** identifica la red: los primeros 3 dígitos son el MCC (001 = red de prueba) y los últimos 2 son el MNC (01). Este valor debe coincidir en tres lugares simultáneamente: aquí en el gNB, en el Open5GS, y en el USIM del srsUE. Si cualquiera de los tres difiere, el UE no reconoce la celda como su operador.
- **`sst: 1`** es el Slice/Service Type. SST=1 es eMBB (datos móviles estándar). 

**El SST hace parte de un identificador unico conocido como S-NSSAI (Single - Network Slice Selection Assistance Information) que se compone de dos campos, el SD (Service Diffrentiator) como opcional y el SST que EL COMPORTAMIENTO ESPERADO DE EL SLICE DE RED EN TERMINOS DE CARACTERÍSTICAS Y SERVICIOS.

![[Pasted image 20260512225947.png]]



![[Pasted image 20260512225610.png]]

Fuente SST : https://5gworldpro.com/blog/2023/05/28/5g-slicing-and-s-nssai/

```
ru_sdr:
  device_driver: zmq
  device_args: tx_port=tcp://127.0.0.1:2000,rx_port=tcp://127.0.0.1:2001,base_srate=23.04e6
  srate: 23.04
  tx_gain: 75
  rx_gain: 75

```

- **`device_driver: zmq`** le dice al gNB que en lugar de usar hardware SDR real, use ZeroMQ como canal de radio virtual. Sin esto el gNB busca un USRP o similar y falla.

- **`device_args`** define los sockets ZeroMQ:

- `tx_port=tcp://127.0.0.1:2000` — el gNB **transmite** (DL) por el puerto 2000
- `rx_port=tcp://127.0.0.1:2001` — el gNB **recibe** (UL) por el puerto 2001

El srsUE debe tener la configuración **inversa**: transmite al puerto 2001 (lo que el gNB recibe) y recibe del puerto 2000 (lo que el gNB transmite). Si los puertos están cruzados, la comunicación es en un solo sentido o no funciona.

- **`base_srate=23.04e6`** es la tasa de muestras del canal ZeroMQ — debe ser idéntica en gNB y srsUE.

- **`srate: 23.04`** es la tasa de muestreo en MHz. Para 20 MHz de ancho de banda con µ=0 (15 kHz SCS), la tasa mínima es 23.04 MHz. Si se cambia  el ancho de banda del carrier, esta tasa debe ajustarse.

```
cell_cfg:
  dl_arfcn: 368500
  band: 3
  channel_bandwidth_MHz: 20
  common_scs: 15
  plmn: "00101"
  tac: 1
```

**`dl_arfcn`** es el NR-ARFCN — el número que identifica la frecuencia central del carrier de downlink en la escala de frecuencias 5G NR. El valor 368500 corresponde a la Banda n3 (1.8 GHz). En ZeroMQ no transmites por el aire, pero el srsUE necesita este valor para simular que está buscando la celda en esa frecuencia — si no coincide con el ARFCN configurado en el srsUE, el UE "no encuentra" la celda.

Calculadora ARFCN : https://5g-tools.com/5g-nr-arfcn-calculator/
Info de interes :  https://www.nrexplained.com/band



#### Open5GS y WebUI

- Detener procesos correspondientes a LTE
- Crear Scripts de activación y detención de los servicios


***Detener procesos correspondientes a LTE

- `sudo systemctl stop open5gs-pcrfd`
- `sudo systemctl stop open5gs-mmed`
- `sudo systemctl stop open5gs-hssd`
- `sudo systemctl stop open5gs-sgwud`
- `sudo systemctl stop open5gs-sgwcd`

***Crear scripts de activación y detención de los servicios 

```
cd /usr/bin

sudo touch 5gc

sudo touch stop-5gc 

```


```bash
#####start 5GC#########

#!/usr/bin/zsh

sudo rm /var/log/open5gs/*
sudo systemctl restart open5gs-smfd
sudo systemctl restart open5gs-amfd
sudo systemctl restart open5gs-upfd
sudo systemctl restart open5gs-nrfd
sudo systemctl restart open5gs-scpd
sudo systemctl restart open5gs-ausfd
sudo systemctl restart open5gs-udmd
sudo systemctl restart open5gs-pcfd
sudo systemctl restart open5gs-nssfd
sudo systemctl restart open5gs-bsfd
sudo systemctl restart open5gs-udrd
sudo systemctl restart open5gs-webui


```

```bash
#####stop 5GC#########

#!/usr/bin/zsh

sudo rm /var/log/open5gs/*
sudo systemctl stop open5gs-smfd
sudo systemctl stop open5gs-amfd
sudo systemctl stop open5gs-upfd
sudo systemctl stop open5gs-nrfd
sudo systemctl stop open5gs-scpd
sudo systemctl stop open5gs-ausfd
sudo systemctl stop open5gs-udmd
sudo systemctl stop open5gs-pcfd
sudo systemctl stop open5gs-nssfd
sudo systemctl stop open5gs-bsfd
sudo systemctl stop open5gs-udrd
sudo systemctl stop open5gs-webui


```

```
sudo chmod +x 5gc
sudo chmod +x stop-5gc
```

**Validar si el script funciona

```
ps aux | grep open5gs
```

Al ejecutar los servicios de open5GS , debemos observar que se crea una interfaz -OGSTUN , en el caso de no ser así, debe crearse manualmente de la siguiente manera :

```
sudo ip tuntap add name ogstun mode tun
sudo ip addr add 10.45.0.1/16 dev ogstun
sudo ip link set ogstun up
```


**IMPORTANTE : Cada que la VM se reinicie se debe volver a crear el namespace o automatizar su creación.

**Revisar que IPv4 este `enabled`

```bash
sudo sysctl -a | grep ip_forward

sudo sysctl -w net.ipv4.ip_forward=1 # En el caso de que no tenga el valor 1
```

***Configurar las reglas de enrutamiento para el NAT forwarding

MASQUERADE o SNAT dinamico , es una técnica de traducción de direcciones de red que modifica la dirección IP de origen de los paquetes salientes por una IP pública variable.

```bash
sudo iptables -L -n -v -t nat # Revisa si ya hay alguna configuración

# En el caso de que no este nada configurado ...

sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE 

```


- `-t nat` — opera en la tabla NAT
- `-A POSTROUTING` — actúa después de que el kernel decide por dónde sale el paquete
- `-s 10.45.0.0/16` — aplica solo a paquetes cuyo origen es el pool de UEs
- `! -o ogstun` — excepto cuando el paquete sale por `ogstun` mismo (tráfico interno UPF)
- `-j MASQUERADE` — sustituye la IP origen por la IP de la interfaz de salida dinámicamente

Cuando un paquete del UE sale hacia Internet, iptables cambia la IP origen `10.45.0.2` por la IP de `eth0` de VM2 (que sí es alcanzable, porque VirtualBox hace otro NAT hacia la red local). La respuesta llega a `eth0`, iptables la traduce de vuelta y la entrega al UPF que la reencapsula en GTP-U hacia el gNB.

***amf-smf-upf

Datos a modificar :

- logger -> level : debug
- Plmn Id : 001 01
- tac : 1
- sst : 1

***Web UI config

 - Ejecutar el 5GC
 - Acceder al localhost:9999 
 - copiar la siguiente info en el apartado de crear suscriptor

![[Pasted image 20260518111008.png]]

Deben quedar de esta manera :

![[Pasted image 20260518111234.png]]


### Ejecutar los servicios

**Hacer captura de tráfico antes de iniciar los servicios en maquina con Open5GS

```bash
sudo tcpdump -i  any -w 5gc.pcap
```

**1. Ejecutar el core 5G

```bash
5gc #script ya creado
ps aux | grep open5gs #Corroborar que el servicio esta en ejecución
sudo netstat -tulnp | grep open5gs
```

**2. Ejecutar el gnB

```bash
sudo ./gnb -c gnb_zmq.yaml # Apuntar a el nombre del archivo de configuración
```

**3. Ejecutar el UE

```bash
sudo ip netns exec ue1 ./src/srsue ue_zmq.conf # Apuntar a el nombre del archivo de configuración 

# Escribir t para ver las metricas 
```

**4. hacer test a la red

#### Ping

Antes de hacer ping debemos crear luna ruta dentro de el UE que apunte a el 5GC.

```bash
sudo ip ro add 10.45.0.0/16 via 192.168.56.111
```
##### Uplink

To test the connection in the uplink direction, use the following:

```bash
sudo ip netns exec ue1 ping 10.45.1.1
```

##### Downlink
To run ping in the downlink direction, use:

```bash
ping 10.45.1.2
```

The IP for the UE can be taken from the UE console output. This might change each time a UE reconnects to the network, so it is best practice to always double-check the latest IP assigned by reading it from the console before running the downlink traffic.


**Generar trafico con Iperf3

Iperf3 nos permite generar tráfico en uplink, por medio de UDP a 10Mbs por 60 segundos.

- Iniciar primero el server
- Segundo el cliente 

**5GCore (Iperf3-server)

```bash
iperf3 -s
```


**UE (Iperf3-Client)

```bash
iperf3 -c 192.168.56.111
```

Genera trafico en la dirección de uplink 

El tráfico se enviara desde el UE a el core, los outputs se mostraran en la consola de ambos componentes (UE y core)

***Ping a google para verficar conexión a internet

```
ping google.com -I tun_srsue -n
```

![[Pasted image 20260526080816.png]]


## Fase 3 (Pendiente por modificar)

https://www.youtube.com/watch?v=G5lV_kazVjI&t=303s

### Criterios para tener en cuenta respecto a el SDR

- Tipo de driver (Soapy, UHD,etc)
- Rango de frecuencia en la que opera 
- Ancho de banda disponible
- Clock rate
- Canales disponibles (MIMO, SISO, etc)
- Especificaciones de la FPGA

**Referencia a utilizar :  Ettus B200

**Arquitectura de hardware

![[Pasted image 20260603090718.png]]

**Características :

- RF integrado con cobertura de 70MHz - 6 GHz -> Importante para definir las bandas NR que se pueden usar en la implementación.
- Ancho de banda Analogo Variable (200kHz - 56 MHz) -> El B200 tiene un filtro analógico configurable antes del ADC, el cual define cuanto espectro pasa realmente al procesamiento digital, por tanto este filtro debe ser de almenos 20 MHz en nuestra implementación.
- Clock rate configurable -> Afecta a la tasa de sampleo `srate`
- Referencia externa de 10 MHz -> Es la referencia de frecuencia del oscilador. Afecta directamente a el terminal comercial para la sincronización en frecuencia.
- El PPS (Pulse per second) -> es la referencia de tiempo ,  especialmente importante si la banda que estamos usando es TDD donde DL y UL alternan en el tiempo y la sincronización debe ser exacta.

**Importante : De acuerdo a el tipo de banda que nos de el calculo de el arfcn va influir en si usamos TDD o FDD.
### Configuración de el SDR


**Arquitectura de la implementación 

https://app.eraser.io/workspace/7cUMYYptVfM4bPnuP2g3?origin=share


![[Pasted image 20260609145843.png]]


**Direccionamiento IP

![[Pasted image 20260609151603.png]]



**Parametros de la celda

| Parámetro | Valor |
|---|---|
| PLMN | 00101 (MCC: 001, MNC: 01) |
| Banda | n3 (FDD) |
| Frecuencia DL | 1842.5 MHz (ARFCN 368500) |
| Frecuencia UL | 1747.5 MHz |
| Ancho de banda | 5 MHz |
| SCS | 15 kHz |
| TAC | 1 |
| PCI | 1 |
| tx_gain | 76 dB |
| rx_gain | 40 dB |
| Aislamiento CPU | 4, 6, 8, 10 (isolcpus + nohz_full) |

**Caculo del arfcn y valores de ganancia , que implica TDD o FDD - Justificar elección (pendiente)


**Flujo de Registro 

El proceso completo desde que el teléfono busca la red hasta que tiene datos:

```
UE (Teléfono)          gNB (srsRAN)              Core (Open5GS)
     │                      │                          │
     │──── SSB / MIB ───────│                          │  gNB transmite beacon
     │──── SIB1 ────────────│                          │  UE lee parámetros de celda
     │                      │                          │
     │══ PRACH (Msg1) ══════│                          │  Random Access
     │◄═ RAR   (Msg2) ══════│                          │  gNB responde
     │══ RRC Setup Req ═════│                          │
     │◄═ RRC Setup    ══════│                          │
     │                      │                          │
     │── NAS: Registration Request ──────────────────►│  AMF
     │◄─ NAS: Identity Request ──────────────────────-│
     │── NAS: Identity Response (SUCI) ──────────────►│
     │                      │                ┌────────►│  AUSF / UDM
     │                      │                │ Auth    │
     │◄─ NAS: Authentication Request ────────┘────────│
     │── NAS: Authentication Response ───────────────►│  (SQN resync en 1er intento)
     │◄─ NAS: Security Mode Command ─────────────────-│
     │── NAS: Security Mode Complete ────────────────►│
     │◄─ NAS: Registration Accept ───────────────────-│
     │── NAS: Registration Complete ─────────────────►│
     │                      │                          │
     │── NAS: PDU Session Establishment Request ──────►│  SMF
     │                      │                ┌────────►│  UPF (PFCP)
     │                      │                │ Túnel   │
     │◄─ NAS: PDU Session Establishment Accept ────────│
     │                      │                          │
     │  IP: 10.45.0.2       │   GTP-U tunnel activo   │  Datos cursando
```

#### Validación Wireshark

**Conexión AMF con el gNB

- Protocolo de señalización :  N2
- Protocolo principal : NGAP

1. Conexión SCTP establecida y  NGSetupRequest/Response presentes en la captura (gNB se registra en el AMF)

![[Pasted image 20260609152557.png]]

2. Handshake sctp 
![[Pasted image 20260609153316.png]]


**Registro NAS del UE

1. Primer mensaje del UE que llega a el amf

![[Pasted image 20260609155241.png]]

2. Autenticación 

![[Pasted image 20260609155335.png]]

![[Pasted image 20260609155411.png]]


**Sesión PDU - IP asignada al UE

1. Ip asignada al dispositivo
![[Pasted image 20260609154936.png]]

![[Pasted image 20260609155002.png]]

2. Navegación del UE en google

![[Pasted image 20260609155557.png]]


**Sesión 5G completa 


Esta captura muestra una **sesión 5G completa y funcional**. Se pueden identificar claramente 3 fases:

1. Establecimiento de sesión PDU (Core 5G)

|Pkt|Protocolo|Evento|
|---|---|---|
|1214|HTTP/2 (SBI)|SMF recibe solicitud de sesión PDU del UE|
|1286|PFCP|SMF → UPF: solicitud de establecer sesión|
|1287|PFCP|UPF → SMF: respuesta OK|
|1289|HTTP/2 (SBI)|SMF notifica al AMF: sesión aceptada|
|1292|NGAP|AMF → gNB: `PDUSessionResourceSetupRequest`|
|1299|NGAP|gNB → AMF: `PDUSessionResourceSetupResponse`|
|1301|HTTP/2 (SBI)|SMF modifica contexto (actualiza parámetros)|
|1302-1303|PFCP|Modificación de sesión en el UPF|

Esto confirma que el **plano de control 5G funciona correctamente**.

2. Tráfico de usuario real (GTP-U)

A partir del paquete **1324**, el tráfico viaja encapsulado en **GTP** (túnel entre gNB y UPF):

- `10.45.0.2` → IP asignada al **UE** dentro del core
- `8.8.8.8` → DNS de Google (internet real)
- `8.8.4.4` → DNS secundario de Google

Se ven conexiones TCP (SYN, SYN-ACK) y consultas DNS a:

- `www.google.com`
- `conn-service-us-05.allawnos.com`
- `www.googleapis.com`


**1. Descarga de drivers

```bash
sudo yum install uhd uhd-devel
sudo uhd_images_downloader ## Firmware FPGA

```


Pruebas para saber si el PC reconoce la B200 :

```bash
uhd_find_devices
lsusb | grep Ettus # Verificar que Linux detecta el USB
```

**2. Sincronización de relojes (Se realiza en los archivos correspondientes a las dependencias del driver)

### Métricas

#### Rendimiento de la red

#### Iperf3

Configurar :

- Iperf server -> 5G-core
- Iperf client -> gNB

![[Pasted image 20260602170337.png]]




**Latencia

Es el tiempo que tarda un paquete en ir desde el emisor al receptor y volver.Es una métrica crítica porque determina la capacidad de respuesta de la red. Influyen factores como : 

- Los retardos en la red IP
- El tiempo de procesamiento en el núcleo de red (5GC)
- El tiempo de transmisión sobre la interfaz radio (RAN)

Valor teórico : < 1 ms

***Resultado : 28 ms



**Throughtput 

El rendimiento máximo es la cantidad de datos que la red puede trnasmitir en un tiempo determinado , representando la capacidad efectiva de la red para enviar y recibir datos.

El ancho de banda define  los datos que podemos transmitir por segundo y esta directamente relacionado con los PRBs (Physical Resource Blocks), cada PRB tiene 12 subportadoras y 7 simbolos en 0.5 ms, es decir, 84 símbolos por slot.

(Pendiente el resto de info)

Valor teórico : >10 Mbps
Valor teórico calculado : Pendiente



***Resultado :

- DL : 19.77 Mb/s
- UL : 23.50 Mb/s



**Jitter

Es la variación en la latencia entre paquetes consecutivos. Mide cuanto varía el tiempo de llegada de los paquetes respecto a un intervalo ideal.

Valor teórico: < 5 ms

***Resultado: No calculado en las herramientas para revisar el rendimiento



### Programación de la SIM

**1. Crear un entorno virtual de Python 3.12 para aislar las dependencias

En este entorno ejecutar los siguientes comandos :

![[Pasted image 20260602154537.png]]

![[Pasted image 20260602154554.png]]


**2. Instalar pySIM 

![[Pasted image 20260602154629.png]]

**3. Programción de la SIM (Agregar parametros : opc , mcc , mnc , imsi)

![[Pasted image 20260602154955.png]]

![[Pasted image 20260602155006.png]]

**4. Verificar que se han cambiado exitosamente los datos

```bash
pySim-read
```

Debe aparecer algo como : 

![[Pasted image 20260602155247.png]]



**Importante : Deshabilitar la búsqueda de redes LTE y por ende de funcionalidades que correspondan a este sistema


**Tambien se deben habilitar los servicios asociados a 5G

- 122 → Gestión de movilidad 5G 
- 123 → Seguridad 5G 
 - 124 → Privacidad del identificador (SUPI/SUCI)
![[Pasted image 20260602134740.png]]


**Desactivar el servicio :

- 125 → Cálculo de SUCI en la USIM

![[Pasted image 20260602134712.png]]



**Deshabilitar servicios no soportados por la SIM


**El archivo EF.Routing_Indicator debe contener un valor válido para que la funcionalidad SUCI funcione correctamente. Si el valor está vacío (por ejemplo ffffffff), es necesario actualizarlo. Para un valor típico:

![[Pasted image 20260602135132.png]]

Despues del cambio, verificar :

![[Pasted image 20260602135333.png]]


![[Pasted image 20260602140306.png]]
