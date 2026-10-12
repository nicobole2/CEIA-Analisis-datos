# Reingreso hospitalario en pacientes diabéticos

**FIUBA · CEIA · Análisis de Datos 2026 · Trabajo Práctico Integrador · Grupo 1**

> ¿Podemos anticipar, al momento del alta, qué pacientes con diabetes volverán a internarse en menos de 30 días?

Este proyecto analiza 101.766 internaciones de 130 hospitales de EE.UU. (1999–2008) y deja los datos **listos y verificados** para construir un modelo que estime ese riesgo. El foco no está en el modelo: está en que los datos sean confiables y que cada decisión de preparación se pueda explicar.

---

## Por qué importa

Un reingreso temprano suele indicar un alta prematura o un seguimiento insuficiente. Los sistemas de salud lo usan como indicador de calidad y, en algunos casos, penalizan los reingresos evitables. Si pudiéramos priorizar a los pacientes de mayor riesgo, el equipo clínico podría reforzar el seguimiento (llamado, control ambulatorio, revisión de medicación) justo donde más se necesita.

## Lo que encontramos

| Hallazgo | Dato |
|---|---|
| Internaciones que terminan en reingreso antes de 30 días | **11,2 %** |
| Reingreso en pacientes sin internaciones previas | 8,4 % |
| Reingreso en pacientes con 5 o más internaciones previas | **36,4 %** (4,3 veces más) |
| Reingreso con alta a rehabilitación frente a alta a domicilio | 27,7 % frente a 9,3 % |
| Pacientes con más de una internación | ~23 % de los pacientes, ~46 % de las filas |

El **historial de internaciones** es la señal más clara. Son asociaciones observadas, no relaciones de causa y efecto.

## Qué hicimos con los datos

| Paso | Decisión | Por qué |
|---|---|---|
| **Limpieza** | De 101.766 a 100.111 internaciones (−1,63 %), sin perder ningún caso de reingreso | Se retiraron fallecidos o en hospicio (no pueden reingresar), registros con género inválido y columnas sin información |
| **Datos faltantes** | Categoría propia («Desconocido», «No medido») en lugar de inventar valores | Que un examen no se haya pedido es información. Rellenarlo con el valor más común fabricaría datos |
| **Descartes** | Se eliminan peso (96,9 % faltante) y código del pagador (39,6 % faltante, de carácter administrativo) | Eran poco confiables o de utilidad limitada, y el pagador puede introducir sesgo socioeconómico |
| **Códigos ocultos** | Se detectaron entre 4,6 % y 10,2 % de valores «nulos» disfrazados de códigos válidos | Usando el diccionario oficial del dataset |
| **Valores extremos** | Se limitan (no se eliminan) | Son pacientes reales y graves. Borrarlos nos dejaría sin los casos que más interesan |
| **Separación entrenamiento/prueba** | Por paciente, no por fila: 0 pacientes compartidos | Evita que el modelo «reconozca» a un paciente que ya vio y que los resultados parezcan mejores de lo que son |
| **Desbalance** | Ponderar la clase minoritaria, en lugar de generar casos sintéticos | No altera los datos reales ni crea pacientes inexistentes |
| **Reducción de variables** | De 132 a 118 variables por selección | Se descartó PCA: comprime poco y hace el modelo difícil de explicar |

## ¿Podemos confiar en estos datos?

Se aplicaron 16 verificaciones antes de dar por terminado el trabajo.

- **11 de 11 chequeos de integridad aprobados:** sin faltantes, sin identificadores colados, mismas variables en entrenamiento y prueba, sin pacientes compartidos.
- **Prueba de «etiquetas mezcladas»:** si mezclamos al azar quién reingresó, el rendimiento cae al nivel del azar (0,51). Con las etiquetas reales sube a 0,65. Es decir, hay señal real y no se detectaron fugas.
- **Estabilidad:** la diferencia entre entrenamiento y prueba es de 0,02 y el resultado casi no cambia entre particiones (0,007).

| Escenario | Capacidad de distinguir (AUC) |
|---|---|
| Azar | 0,50 |
| Etiquetas mezcladas | 0,51 |
| **Datos reales** | **0,65** |

**Lectura honesta:** la señal es real pero modesta, en línea con lo que reporta la literatura para este dataset. Sirve para **priorizar**, no para decidir por sí sola. Estas pruebas no garantizan la ausencia absoluta de fugas.

## Limitaciones que hay que tener presentes

1. **La etiqueta es imperfecta.** «No reingresó» mezcla a quienes no volvieron con quienes simplemente salieron de la ventana de observación. No se corrige limpiando datos.
2. **Los datos son de 1999 a 2008.** La práctica clínica cambió desde entonces.
3. **Variables sensibles** (raza, género, edad). Antes de cualquier uso real, hace falta una auditoría de equidad entre grupos.

## Próximos pasos

- Ajustar y validar el modelo agrupando por paciente.
- Calibrar las probabilidades para que «30 %» signifique realmente 30 %.
- Evaluar el costo-beneficio de dónde fijar el umbral de alerta.

---

## Cómo reproducir el análisis

**1. Preparar el entorno** (Python 3.13)

```bash
python -m venv .venv
source .venv/bin/activate        # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

**2. Descargar los datos**

Tomarlos de [UCI · Diabetes 130-US hospitals for years 1999-2008](https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008) y dejar estos dos archivos en la carpeta `content/`:

```
content/diabetic_data.csv
content/IDS_mapping.csv
```

**3. Ejecutar**

```bash
jupyter notebook Grupo1.ipynb
```

Al ejecutarlo completo (≈ 3 minutos) se reproducen todas las cifras de este documento.

### Opciones

Al comienzo del notebook, en la sección «Configuración»:

| Parámetro | Valor por defecto | Qué hace |
|---|---|---|
| `ENTRENAR_MODELO` | `True` | Entrena y compara modelos al final. El trabajo no lo requiere: con `False` se omite y todo lo demás queda igual |
| `GUARDAR_SALIDA` | `False` | Con `True` exporta las matrices finales (entrenamiento y prueba) a la carpeta `salida/` |
| `RANDOM_STATE` | `42` | Semilla para que los resultados sean reproducibles |

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `Grupo1.ipynb` | **Notebook principal**: análisis, preparación, verificaciones y modelo opcional, con cada decisión comentada |
| `content/` | Datos de entrada (ver «Descargar los datos») |
| `requirements.txt` | Dependencias con las versiones probadas |
| `Análisis de datos - Trabajo integrador - 4B2026.pdf` | Enunciado del trabajo |

