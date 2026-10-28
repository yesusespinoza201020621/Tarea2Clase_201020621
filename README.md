# Laboratorio ACYE1 - Tarea No. 2: Probador Automático de Circuitos Integrados (7400, 7408, 7427, 7432)

## 👥 Datos del Estudiante
* **Nombre:** Yesus Rudy Espinoza Pivaral
* **Carnet:** 201020621
* **Universidad:** Universidad de San Carlos de Guatemala
* **Facultad:** Ingeniería
* **Escuela:** Ciencias y Sistemas
* **Curso:** Arquitectura de Computadores y Ensambladores 1
* **Catedrático:** Ing. Otto Escobar Leiva
* **Sección de Laboratorio:** A
* **Auxiliar:** Diego Josue Guevarra

---

## 🎯 1. Marco Formativo y Valores

### 🤝 Responsabilidad y Calidad de Ingeniería
Como futuro ingeniero en Ciencias y Sistemas de la USAC, asumo la responsabilidad absoluta de diseñar soluciones de hardware embebido de alta fidelidad. Esta entrega demuestra el rigor técnico en la manipulación directa de registros físicos, la optimización de los buses de datos del procesador y la correcta alineación de la pila bajo los estándares industriales, garantizando un código óptimo, modular y libre de fallas.

### 🧠 Competencias Desarrolladas (Capítulos 1 al 5 del Libro)
1. **Representación de Datos y Control de Registros (Capítulo 2):** Manipulación de flujos de 8 bits (`ldrb`) para la lectura ASCII de caracteres de control y registros empaquetados de 16 bits (`W10`) para el parsing de códigos binarios, evitando el desperdicio de recursos del bus de 64 bits.
2. **Direccionamiento Escalado por Hardware (Capítulo 3):** Acceso secuencial e indexado a los vectores contiguos de excitación y tablas de verdad mediante instrucciones de carga con escalado lógico (`ldrh w0, [x20, x23, lsl #1]`), multiplicando el índice de forma inmediata por 2 bytes debido al uso de constantes de media palabra (`.hword`).
3. **Aislamiento y Enmascaramiento de Bits (Capítulo 4):** Empleo de operaciones booleanas puras por hardware (`EOR` y `TST`) para contrastar las salidas físicas devueltas por los integrados contra las tablas de verdad esperadas, aislando pines específicos (`D9`, `D6`, `D5`, `D0`) sin alterar el estado del resto de las señales.
4. **Preservación del Contexto de la Pila (Capítulo 5):** Implementación estricta de la convención formal de llamadas de ARM64 (AAPCS) mediante pre-indexación y post-indexación (`stp` / `ldp`), garantizando el balanceo del Stack Pointer (`SP`) y protegiendo el Link Register (`X30`) en llamadas anidadas.

---

## 🧭 2. Arquitectura de Hardware y Mapeo de Pines

El probador empaqueta y desempaqueta todas las señales físicas del circuito integrado (entradas de excitación y salidas del buffer) utilizando únicamente el registro **`X0`** de 64 bits de forma *in-place*. El mapa de bits implementado para la correspondencia de pines es el siguiente:

| Bit en `X0` | `D11` | `D10` | `D9` | `D8` | `D7` | `D6` | `D5` | `D4` | `D3` | `D2` | `D1` | `D0` |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Pin Físico** | pin1 | pin2 | pin3 | pin4 | pin5 | pin6 | pin8 | pin9 | pin10 | pin11 | pin12 | pin13 |

### 🛠️ Justificación de las Syscalls Utilizadas (Linux ARM64 Puro)
* **`sys_write` (Syscall 64):** Envía flujos de caracteres ASCII directamente a la salida estándar (`stdout`) para renderizar los encabezados personales, el prompt interactivo y los diagnósticos globales (`IC OK` / `IC MAL`).
* **`sys_read` (Syscall 63):** Pausa el flujo de la CPU por hardware esperando que el usuario ingrese caracteres en el teclado (`stdin`), guardando los bytes en un búfer seguro de la sección `.bss`.
* **`sys_exit` (Syscall 93):** Realiza un cortocircuito síncrono al detectar la palabra de control `exit`, invocando la terminación segura del proceso para devolver el control a la terminal de Linux de forma limpia.

---

## 🛠️ 3. Guía de Compilación y Automatización

Para ensamblar y ejecutar el entorno interactivo del probador sin una sola advertencia y utilizando el enlazador estático puro (`ld`), ejecute los siguientes comandos en la terminal integrada de Visual Studio Code:

```bash
# 1. Ensamblar el código fuente puro
as -o tarea2_final.o tarea2_final_201020621.s

# 2. Enlazar estáticamente sin librerías externas de C
ld -o probador_ejecutar tarea2_final.o

# 3. Correr el programa autónomo interactivo
./probador_ejecutar
```

---

## 🖼️ 4. Casos de Prueba y Diagnóstico en Consola

El sistema embebido incluye un banco de pruebas interactivo que valida el comportamiento del chip ante múltiples escenarios extremos:

* **Entrada Inválida:** Si el usuario ingresa un código fuera de la matriz de la USAC (ej: `99`), el despachador intercepta el error e imprime: *“Código no válido. Intente con 00, 08, 27, 32 o salir con exit.”*.
* **IC OK:** Al ingresar un integrado en buen estado (ej: `08`), el bucle procesa los 4 vectores de verdad y, al no encontrar discrepancias lógicas (`W22 == 0`), imprime el éxito por pantalla.
* **Daño Parcial:** Al descomentar la línea de depuración de cortocircuito (`eor w0, w0, #(1 << 0)`), el enmascaramiento por hardware aísla el pin defectuoso y el intérprete notifica dinámicamente el fallo específico (ej: `COMPUERTA 4 DAÑADA`).
* **Exit:** Al ingresar el comando de escape `exit`, la CPU interrumpe el bucle indefinido y cierra el programa limpiamente sin causar fallos de segmentación.
# Tarea2Clase_201020621
