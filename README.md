---
title: "Como usar datos climaticos para modelar variables morfologicas y fisiologicas Caso"
author: "Jorge González Campos"
date: "2024-12-18"
output:
  html_document:
    toc: true            # Activa el índice
    toc_float: false      # Hace el índice flotante
    toc_position: left   # Posiciona el índice a la izquierda
    css: "styles.css"    # Enlaza un archivo CSS personalizado (opcional)
---

# **Introdución**

::: {align="justify"}
La modelación de caracteres morfológicos y fisiológicos en especies leñosas en relación con variables climáticas es fundamental para comprender su adaptación y respuesta al cambio ambiental.

Mediante modelos de nicho ecológico, técnicas de machine learning y simulaciones ecofisiológicas, es posible predecir patrones de crecimiento y distribución de las especies bajo diferentes escenarios climáticos.

Esta información es esencial para la gestión forestal, permitiendo seleccionar genotipos mejor adaptados, optimizar estrategias de silvicultura y diseñar planes de conservación eficaces. Además, la modelación facilita la identificación de áreas de alto riesgo por el cambio climático y contribuye a la planificación de corredores ecológicos que favorezcan la migración de especies en respuesta a nuevas condiciones ambientales. En un contexto de creciente variabilidad climática, integrar estos modelos en la toma de decisiones permite una gestión sostenible de los bosques y sus recursos genéticos.
:::

## **1. Rasgos de plantas**

::: {align="justify"}
Los rasgos funcionales de las plantas son características que definen su forma, función y estrategias ecológicas. Estan estrechamente vinculados a gradientes ambientales, como el clima y las propiedades del suelo

Estos rasgos son esenciales en modelos de vegetación y en modelos acoplados clima-vegetación, permitiendo un análisis adecuado de la dinámica de las pobalciones de una especie vegetal bajo el cambio global.
:::

### Caso de estudio: *Araucaria araucana*

::: {align="justify"}
El Pehuén o Araucaria (*Araucaria araucana* (Molina) K. Koch) es una de las coníferas nativas más longevas de los bosques templados de Chile y Argentina, reconocida por su incuestionable valor cultural, social y ecológico. Esta especie posee una distribución disyuntiva, con dos poblaciones localizadas en la Cordillera de la Costa y otras en la Cordillera de los Andes, abarcando un rango que va desde los 37° 20'S hasta los 40°20'S (Bekessy et al., 2004).

![Reserva Nacional Malalcahuello, Región de la La Araucanía, Chile.](images/IMG_20210518_221701_840.jpg)
:::

::: tip
📌 **Consejo 1: Organiza tu Directorio de Trabajo**

Mantener un directorio de trabajo bien estructurado es clave para ahorrar `tiempo` y evitar confusiones. Define una carpeta específica para tu proyecto y almacena en ella todos los archivos relacionados, como datos, scripts y resultados. Esto facilitará la gestión y evitará pérdidas de información en el futuro.
:::

Usa `setwd("ruta/del/proyecto")` en R para establecer tu directorio de trabajo.

```{r echo=FALSE, warning=FALSE, results='hide'}

rm(list = ls()) # Reiniciar el entorno
gc() # Limpiar memoria
```

```{r, echo=FALSE}

knitr::opts_knit$set(root.dir = "C:/Users/jmgon/OneDrive - Instituto Forestal/INFOR/6. Proyectos Finalizados/Proyecto SIMEF Araucaria araucana/5. Bases de Datos/Semillas")

```

## 🔍 Paso 1: Visualización del problema

::: {align="justify"}
Los datos generalmente están organizados en una tabla, ¿cierto? Seguramente tomaste varias mediciones de diferentes rasgos en individuos de las distintas poblaciones que visitaste. Estos rasgos, casi siempre, son de naturaleza morfológica y/o fisiológica.

| Pop  | Rasgo 1 | Rasgo 2 | Rasgo 3 | Latitud | Longitud |
|------|---------|---------|---------|---------|----------|
| pop1 | 10.5    | 3.4     | 8.1     | -38.5   | -73.2    |
| pop1 | 12.3    | 3.1     | 7.5     | -38.6   | -73.3    |
| pop1 | 11.8    | 3.3     | 8.2     | -38.7   | -73.4    |
| pop2 | 9.7     | 4.0     | 6.9     | -38.2   | -73.5    |
| pop2 | 8.9     | 4.1     | 7.0     | -38.1   | -73.6    |
| pop2 | 10.1    | 4.2     | 6.8     | -38.0   | -73.7    |
| pop3 | 13.2    | 5.5     | 9.0     | -39.0   | -74.0    |
| pop3 | 14.1    | 5.1     | 8.5     | -39.1   | -74.1    |
| pop3 | 12.8    | 5.3     | 9.1     | -39.2   | -74.2    |
:::

Para analizarlos de manera eficiente, es útil separar el flujo de trabajo en **dos procesos paralelos**:

1️⃣ **Análisis estadístico de tendencias en los rasgos** 2️⃣ **Procesamiento de datos geoespaciales** 📍

Estos dos enfoques se trabajarán por separado y luego se integrarán para una comprensión más completa.

## 🔍 Paso 2: Tendencias de los rasgos

::: {align="justify"}
Comencemos con el análisis estadístico de tendencias en los rasgos,. Si tus datos incluyen múltiples localidades o poblaciones 📍, es clave analizar cómo varían los rasgos en cada una. Para ello, un **gráfico de Ridgeline** 📈 es una excelente opción, ya que permite visualizar la distribución de los rasgos en cada población, mostrando su dispersión y diferencias. Puedes usar el paquete ggridges

```{r warning=FALSE}
library(ggridges)
```

Antes de realizar pruebas estadísticas, es fundamental comprender la distribución de los datos y la medida de tendencia central más adecuada🎯 :
:::

::: tip
📌 Consejo 2:

Visualizar la distribución de los rasgos te ayudará a tomar mejores decisiones sobre qué pruebas estadísticas utilizar, asegurando un análisis más preciso y robusto. 🚀
:::

```{r message=FALSE, warning=FALSE, echo=FALSE}
library(ggplot2)  # Para crear gráficos y visualizaciones
library(dplyr)    # Para manipulación de datos 
library(giscoR)   # Para descargar y trabajar con datos geoespaciales
library(maps)     # Para acceder a datos de mapas geográficos
library(readxl)   # Para leer archivos de Excel (.xlsx) en R

```

```{r, echo=FALSE, results='hide'}
## 1. Carga de datos
# Cargamos y revisamos los datos 
data_sem_imputado <- read_excel("C:/Users/jmgon/OneDrive - Instituto Forestal/INFOR/6. Proyectos Finalizados/Proyecto SIMEF Araucaria araucana/5. Bases de Datos/Semillas/data_sem_imputado.xlsx")

data_sem_imputado
```

```{r, echo=FALSE, results='hide'}
# Renombrar manualmente las columnas
data_sem_imputado <- data_sem_imputado %>%
  rename(
    rasgo1 = viabilidad_tz,
    rasgo2 = sem_semb,
    rasgo3 = sem_germe19,
    rasgo4 = porgerm_e19,
    rasgo5 = sem_germo18,
    rasgo6 = porgerm_o18,
    rasgo7 = ls_sem,
    rasgo8 = asm_sem,
    rasgo9 = abs_sem,
    rasgo10 = ps_sem,
    rasgo11 = sk_sem,
    rasgo12 = tam_embrio,
    rasgo13 = anch_embrio,
    rasgo14 = tam_anch_embrio
  )

# Verificar los nuevos nombres de las columnas
names(data_sem_imputado)

```

::: {align="justify"}
🌱 En este ejemplo, el rasgo en análisis corresponde al largo del embrión en las semillas de Araucaria araucana. Este parámetro morfológico es un indicador interno del desarrollo 🌟 que ocurre a lo largo de casi dos años. Por ello, las variaciones observadas en este rasgo pueden reflejar tanto la influencia de las condiciones ambientales locales 🏞️ como la diversidad intrapoblacional 🌿.

En el gráfico, es evidente que ni la media 🔵 ni la mediana 🔴 representan adecuadamente a las poblaciones costeras 🌊 ni a la población más al norte de los Andes 🏔️. Esto sugiere la presencia de patrones diferenciados entre los grupos.

🔍 Este hallazgo preliminar nos indica que es necesario separar los datos en dos subconjuntos para un análisis más detallado y específico. Por lo tanto, en este caso, procederemos a trabajar con las poblaciones de la costa 🌊 y los Andes 🏔️ de manera independiente, permitiendo una mejor comprensión de las variaciones dentro de macrozona.

Al ver todas las localidades en conjunto. Se observan las tendencias centrales s:

-   **Media**: 3.15 🔵
-   **Mediana**: 3.10 🔴
-   **Moda**: 2.77 🟣

La **distribución** del rasgo 12 muestra una ligera asimetría, con la **media** siendo algo más alta que la **mediana**, lo que indica una leve inclinación hacia los valores más altos. Es de esperar considerando que la media es afectada por valores extremos , tales como los que se aprencian en la figura.

La **moda** es considerablemente más baja que la media y la mediana, lo que sugiere que una gran parte de las observaciones están agrupadas en torno a un valor más b
:::

```{r, echo=FALSE, message=FALSE, warning=FALSE}
# Cargar librerías necesarias
library(ggplot2)
library(ggridges)  # Para gráficas de ridgeline
library(dplyr)
library(glue)

# Función para calcular la moda
get_mode <- function(x) {
  uniq_vals <- unique(x[!is.na(x)])  # Eliminar valores NA
  uniq_vals[which.max(tabulate(match(x, uniq_vals)))]  # Devolver el valor más frecuente
}


# Función de configuración
establecer_configuracion <- function() {
  list(
    orden_localidades = c("NAH", "VIL", "RAL", "CON", "NAL", "MAL", "LON", "ICA", "HUE", "VIR"),
    colores_localidades = setNames(
      c("#c7e9b4", "#7fcdbb", "#41b6c4", "#1d91c0", "#225ea8", "#feb24c", "#fd8d3c", "#f03b20", "#bd0026", "#800026"),
      c("NAH", "VIL", "RAL", "CON", "NAL", "MAL", "LON", "ICA", "HUE", "VIR")
    ),
    traits = paste0("rasgo", 1:14),
    font_family = "sans",
    bg_color = "white"
  )
}

# Función de preparación de datos
preparar_datos <- function(data, traits, orden_localidades) {
  data %>% 
    mutate(
      across(all_of(traits), ~as.numeric(as.character(.))),
      Localidad = factor(localidad2, levels = rev(orden_localidades))
    ) %>% 
    filter(if_all(all_of(traits), ~!is.na(.)))
}

# Función para generar gráfico
generar_grafico <- function(data, trait, colores_localidades, font_family, bg_color, general_mean, general_median, general_mode) {
  ggplot(data, aes(x = !!sym(trait), y = Localidad, fill = Localidad)) +
    geom_density_ridges(alpha = 0.7) + 
    geom_vline(xintercept = general_mean, color = "blue", linetype = "dashed", size = 1) +
    geom_vline(xintercept = general_median, color = "red", linetype = "dashed", size = 1) +
    geom_vline(xintercept = general_mode, color = "#8E66C1", linetype = "dashed", size = 1) +
    scale_fill_manual(values = colores_localidades) +
    labs(
      title = glue("Distribución de {trait} por localidad"),
      subtitle = glue("Media: {round(general_mean, 2)} | Mediana: {round(general_median, 2)} | Moda: {round(general_mode, 2)}"),
      x = trait,
      y = "Localidad"
    ) +
    theme_minimal(base_family = font_family) +
    theme(
      plot.background = element_rect(fill = bg_color),
      panel.grid = element_blank(),
      plot.title = element_text(size = 12),
      plot.subtitle = element_text(size = 10),
      axis.text.y = element_text(hjust = 0),
      legend.position = "right"
    )
}

# Preparar los datos
config <- establecer_configuracion()
data_prep <- preparar_datos(data_sem_imputado, config$traits, config$orden_localidades)

# Calcular las estadísticas (media, mediana, moda) para rasgo12
general_mean <- mean(data_prep$rasgo12, na.rm = TRUE)
general_median <- median(data_prep$rasgo12, na.rm = TRUE)
general_mode <- get_mode(data_prep$rasgo12)

# Generar gráfico para rasgo12
graficos_todos <- list()
graficos_todos[[12]] <- generar_grafico(
  data = data_prep, 
  trait = "rasgo12", 
  colores_localidades = config$colores_localidades, 
  font_family = config$font_family, 
  bg_color = config$bg_color, 
  general_mean = general_mean, 
  general_median = general_median, 
  general_mode = general_mode
)

# Mostrar gráfico
print(graficos_todos[[12]])

```
