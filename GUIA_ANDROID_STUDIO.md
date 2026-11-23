# Guía completa de Android Studio: tabla, base de datos SQLite y Navigation Drawer

**Tecnologías:** Java, XML Views, SQLiteOpenHelper y Material Components.  
**Nivel:** principiante-intermedio.  
**Objetivo:** aprender a crear una tabla, conectar una base de datos local SQLite y construir un menú lateral (Navigation Drawer), con ejemplos independientes y un anexo final para integrarlos.

> Todos los ejemplos usan una aplicación llamada `GuiaAndroid`, paquete `com.ejemplo.guiaandroid` y `minSdk 24`.

## Índice

1. [Preparación del proyecto](#1-preparación-del-proyecto)
2. [Sección 1: cómo hacer una tabla](#2-sección-1-cómo-hacer-una-tabla)
3. [Sección 2: conexión a base de datos SQLite](#3-sección-2-conexión-a-base-de-datos-sqlite)
4. [Sección 3: Navigation Drawer](#4-sección-3-navigation-drawer)
5. [Anexo: integrar las tres piezas](#5-anexo-integrar-las-tres-piezas)
6. [Pruebas y solución de problemas](#6-pruebas-y-solución-de-problemas)

---

## 1. Preparación del proyecto

### 1.1 Crear el proyecto

1. Abre Android Studio y elige **New Project**.
2. Selecciona **Empty Views Activity** (no confundir con *Empty Activity*, que crea un proyecto con Jetpack Compose).
3. Configura:
   - **Name:** `GuiaAndroid`
   - **Package name:** `com.ejemplo.guiaandroid`
   - **Language:** `Java`
   - **Minimum SDK:** `API 24`
   - **Build configuration language:** `Kotlin DSL (build.gradle.kts)` o Groovy, según tu instalación.
4. Pulsa **Finish** y espera a que termine la sincronización de Gradle.
5. Comprueba en la vista **Project → Android** que existen:
   - `app/java/com.ejemplo.guiaandroid/MainActivity.java`
   - `app/res/layout/activity_main.xml`
   - `app/manifests/AndroidManifest.xml`

> Si tu versión de Android Studio genera otro `compileSdk` o versiones más nuevas de las librerías, usa las que te proponga el asistente. Las versiones de esta guía son solo una referencia estable.

### 1.2 Dependencias

Abre `app/build.gradle.kts` y deja el bloque `dependencies` así:

```kotlin
dependencies {
    implementation("androidx.appcompat:appcompat:1.7.0")
    implementation("com.google.android.material:material:1.12.0")
    implementation("androidx.constraintlayout:constraintlayout:2.2.0")
    implementation("androidx.recyclerview:recyclerview:1.4.0")
    implementation("androidx.fragment:fragment:1.8.5")
    implementation("androidx.drawerlayout:drawerlayout:1.2.0")

    testImplementation("junit:junit:4.13.2")
    androidTestImplementation("androidx.test.ext:junit:1.2.1")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.6.1")
}
```

Si tu proyecto usa Groovy (`build.gradle`), la sintaxis es:

```groovy
implementation 'androidx.appcompat:appcompat:1.7.0'
implementation 'com.google.android.material:material:1.12.0'
implementation 'androidx.constraintlayout:constraintlayout:2.2.0'
implementation 'androidx.recyclerview:recyclerview:1.4.0'
implementation 'androidx.fragment:fragment:1.8.5'
implementation 'androidx.drawerlayout:drawerlayout:1.2.0'
```

Después pulsa **Sync Now**.

### 1.3 Tema sin ActionBar

El Navigation Drawer necesita controlar la barra superior con un `MaterialToolbar`. Por eso el tema principal no debe traer ActionBar.

Edita `app/src/main/res/values/themes.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="Base.Theme.GuiaAndroid" parent="Theme.Material3.DayNight.NoActionBar">
        <!-- Personaliza aquí los colores si lo deseas. -->
    </style>

    <style name="Theme.GuiaAndroid" parent="Base.Theme.GuiaAndroid" />
</resources>
```

> Si el archivo ya existe, conserva la estructura y cambia únicamente el `parent` por `Theme.Material3.DayNight.NoActionBar`.

### 1.4 Textos de toda la guía

Sustituye el contenido de `app/src/main/res/values/strings.xml` por este bloque. Incluye todos los textos que usaremos en las tres secciones:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Guía Android</string>
    <string name="abrir_menu">Abrir menú</string>
    <string name="cerrar_menu">Cerrar menú</string>

    <string name="menu_inicio">Inicio</string>
    <string name="menu_tabla">Tabla</string>
    <string name="menu_base_datos">Base de datos</string>
    <string name="menu_acerca">Acerca de</string>
    <string name="nav_header_subtitulo">Tablas · SQLite · Navigation Drawer</string>

    <string name="titulo_tabla">Tabla de productos</string>
    <string name="titulo_tabla_recycler">Tabla con RecyclerView</string>
    <string name="col_nombre">Producto</string>
    <string name="col_stock">Stock</string>
    <string name="col_precio">Precio</string>

    <string name="titulo_base_datos">Base de datos</string>
    <string name="hint_nombre">Nombre</string>
    <string name="hint_precio">Precio (€)</string>
    <string name="hint_stock">Stock</string>
    <string name="btn_guardar">Guardar</string>
    <string name="btn_actualizar">Actualizar</string>
    <string name="btn_cancelar">Cancelar</string>
    <string name="btn_eliminar">Eliminar</string>
    <string name="titulo_productos">Productos guardados</string>
    <string name="msg_selecciona_para_editar">Toca un producto para editarlo; mantén pulsado para eliminarlo.</string>
    <string name="msg_campos_obligatorios">Completa todos los campos.</string>
    <string name="msg_numero_invalido">El precio y el stock deben ser números válidos.</string>
    <string name="msg_guardado">Producto guardado.</string>
    <string name="msg_actualizado">Producto actualizado.</string>
    <string name="msg_error_guardar">No se pudo guardar el producto.</string>
    <string name="msg_eliminado">Producto eliminado.</string>
    <string name="titulo_eliminar">Eliminar producto</string>
    <string name="msg_eliminar">¿Quieres eliminar «%1$s»?</string>

    <string name="inicio_titulo">Guía de Android Studio</string>
    <string name="inicio_texto">Ejemplos independientes de tablas, conexión a SQLite con SQLiteOpenHelper y un Navigation Drawer. Usa el menú lateral para navegar entre las secciones.</string>
    <string name="placeholder_tabla">Aquí irá la tabla de la sección 1. En el anexo de integración tienes los pasos para incrustarla en este fragment.</string>
    <string name="placeholder_base_datos">Aquí irá el formulario y el listado de la sección 2. En el anexo de integración tienes los pasos para incrustarlos en este fragment.</string>
    <string name="placeholder_acerca">Guía de ejemplo creada con Java, XML, SQLiteOpenHelper y Material Components.</string>
</resources>
```

### 1.5 Estilos y borde de las celdas

Las tablas de la sección 1 comparten estos estilos. Crea `app/src/main/res/values/styles.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="Celda">
        <item name="android:layout_width">0dp</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:layout_weight">1</item>
        <item name="android:padding">10dp</item>
        <item name="android:textSize">14sp</item>
        <item name="android:gravity">center_vertical</item>
        <item name="android:background">@drawable/borde_celda</item>
    </style>

    <!-- El punto hace que Celda.Cabecera herede de Celda. -->
    <style name="Celda.Cabecera">
        <item name="android:textStyle">bold</item>
        <item name="android:textColor">?attr/colorOnSurfaceVariant</item>
    </style>
</resources>
```

Crea `app/src/main/res/drawable/borde_celda.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <stroke
        android:width="1dp"
        android:color="?attr/colorOutlineVariant" />
</shape>
```

> `?attr/colorOutlineVariant` se adapta al modo claro y oscuro. Si tu versión de Material no lo reconoce, cambia esa línea por un color fijo, por ejemplo `#BDBDBD`.

### 1.6 Estructura de archivos que tendrá el proyecto

Al terminar la guía tendrás esta estructura (las clases están en el paquete base para simplificar; en un proyecto real conviene separarlas por paquetes):

```text
app/src/main/
├── java/com/ejemplo/guiaandroid/
│   ├── MainActivity.java
│   ├── TablaActivity.java
│   ├── TablaRecyclerActivity.java
│   ├── BaseDatosActivity.java
│   ├── AdaptadorTabla.java
│   ├── Producto.java
│   ├── DBHelper.java
│   ├── InicioFragment.java
│   └── PlaceholderFragment.java
├── res/
│   ├── layout/
│   │   ├── activity_main.xml
│   │   ├── activity_tabla.xml
│   │   ├── activity_tabla_recycler.xml
│   │   ├── activity_base_datos.xml
│   │   ├── item_fila_tabla.xml
│   │   ├── fragment_inicio.xml
│   │   ├── fragment_placeholder.xml
│   │   └── nav_header.xml
│   ├── menu/drawer_menu.xml
│   ├── drawable/borde_celda.xml
│   └── values/strings.xml, styles.xml, themes.xml
└── AndroidManifest.xml
```

### 1.7 Cómo probar una pantalla aislada

Mientras avanzas por las secciones, la aplicación arranca en `MainActivity`. Para probar `TablaActivity` o `BaseDatosActivity` antes de terminar el menú tienes dos opciones:

- **Temporal:** mueve el `<intent-filter>` con `MAIN`/`LAUNCHER` al activity que quieras probar en el manifiesto. Recuerda devolverlo a `MainActivity` al terminar.
- **Con adb:** con el emulador o dispositivo conectado ejecuta:

```bash
adb shell am start -n com.ejemplo.guiaandroid/.TablaActivity
```

Al final de la guía podrás acceder a todas las pantallas desde el Navigation Drawer.

---

## 2. Sección 1: cómo hacer una tabla

En Android no existe un componente "tabla" universal. Las dos opciones más habituales con XML son:

| Enfoque | Cuándo usarlo |
|---|---|
| `TableLayout` + `TableRow` | Pocas filas, tabla fija o contenido que cabe en una pantalla. |
| `RecyclerView` con filas | Muchos datos, listas que crecen, filas recicladas y mejor rendimiento. |

Veremos las tres variantes: tabla estática, tabla rellenada desde Java y tabla con `RecyclerView`.

### 2.1 Tabla estática con TableLayout

`TableLayout` organiza los hijos en filas (`TableRow`) y columnas. Las columnas se reparten con `android:stretchColumns="*"` y con pesos (`layout_weight`) en cada celda.

#### Layout `activity_tabla.xml`

Crea `app/src/main/res/layout/activity_tabla.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.google.android.material.appbar.MaterialToolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        app:title="@string/titulo_tabla" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">

        <TableLayout
            android:id="@+id/tabla"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:padding="16dp"
            android:stretchColumns="*">

            <!-- Cabecera -->
            <TableRow android:background="?attr/colorSurfaceVariant">
                <TextView
                    style="@style/Celda.Cabecera"
                    android:text="@string/col_nombre" />

                <TextView
                    style="@style/Celda.Cabecera"
                    android:gravity="center_vertical|end"
                    android:text="@string/col_stock" />

                <TextView
                    style="@style/Celda.Cabecera"
                    android:gravity="center_vertical|end"
                    android:text="@string/col_precio" />
            </TableRow>

            <!-- Filas de ejemplo -->
            <TableRow>
                <TextView
                    style="@style/Celda"
                    android:text="Manzanas" />

                <TextView
                    style="@style/Celda"
                    android:gravity="center_vertical|end"
                    android:text="12" />

                <TextView
                    style="@style/Celda"
                    android:gravity="center_vertical|end"
                    android:text="1,20 €" />
            </TableRow>

            <TableRow>
                <TextView
                    style="@style/Celda"
                    android:text="Pan integral" />

                <TextView
                    style="@style/Celda"
                    android:gravity="center_vertical|end"
                    android:text="5" />

                <TextView
                    style="@style/Celda"
                    android:gravity="center_vertical|end"
                    android:text="2,10 €" />
            </TableRow>

            <TableRow>
                <TextView
                    style="@style/Celda"
                    android:text="Leche" />

                <TextView
                    style="@style/Celda"
                    android:gravity="center_vertical|end"
                    android:text="8" />

                <TextView
                    style="@style/Celda"
                    android:gravity="center_vertical|end"
                    android:text="0,95 €" />
            </TableRow>
        </TableLayout>
    </ScrollView>
</LinearLayout>
```

Puntos clave del layout:

- `stretchColumns="*"` hace que las columnas ocupen el ancho disponible.
- Cada celda tiene `layout_width="0dp"` y `layout_weight="1"` en el estilo `Celda`, así todas las columnas miden lo mismo.
- `ScrollView` permite desplazar la tabla verticalmente.
- El borde de cada celda lo aporta `@drawable/borde_celda`; `TableLayout` no dibuja rejillas por sí solo.

#### Activity `TablaActivity.java`

Crea `app/src/main/java/com/ejemplo/guiaandroid/TablaActivity.java`:

```java
package com.ejemplo.guiaandroid;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;

import com.google.android.material.appbar.MaterialToolbar;

public class TablaActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tabla);

        MaterialToolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(R.string.titulo_tabla);
        }
    }

    @Override
    public boolean onSupportNavigateUp() {
        finish();
        return true;
    }
}
```

Registra la actividad en `app/src/main/AndroidManifest.xml`, dentro de `<application>`:

```xml
<activity
    android:name=".TablaActivity"
    android:exported="false"
    android:parentActivityName=".MainActivity" />
```

Ejecuta la app y abre esa pantalla (ver el apartado 1.7 si necesitas lanzarla directamente).

### 2.2 Tabla dinámica: rellenar TableLayout desde Java

Cuando los datos vienen de una lista o de una base de datos, se construyen las filas en código. El siguiente método se puede añadir a `TablaActivity`, debajo de `onCreate`. Sustituye las filas de ejemplo por filas nuevas:

```java
private void cargarFilasDinamicas() {
    TableLayout tabla = findViewById(R.id.tabla);

    // Conserva la cabecera (índice 0) y elimina las filas de datos anteriores.
    while (tabla.getChildCount() > 1) {
        tabla.removeViewAt(1);
    }

    String[][] datos = {
            {"Manzanas", "12", "1,20 €"},
            {"Pan integral", "5", "2,10 €"},
            {"Leche", "8", "0,95 €"},
            {"Café", "3", "4,50 €"}
    };

    for (String[] fila : datos) {
        TableRow tr = new TableRow(this);
        for (int columna = 0; columna < fila.length; columna++) {
            tr.addView(crearCelda(fila[columna], columna));
        }
        tabla.addView(tr);
    }
}

private TextView crearCelda(String texto, int columna) {
    TextView celda = new TextView(this);

    // 0dp + weight 1 = las tres columnas se reparten el ancho.
    TableRow.LayoutParams params = new TableRow.LayoutParams(
            0,
            TableRow.LayoutParams.WRAP_CONTENT,
            1f
    );
    celda.setLayoutParams(params);

    int padding = (int) (10 * getResources().getDisplayMetrics().density);
    celda.setPadding(padding, padding, padding, padding);
    celda.setText(texto);
    celda.setTextSize(14f);
    celda.setBackgroundResource(R.drawable.borde_celda);

    int gravedad = (columna == 0)
            ? Gravity.START | Gravity.CENTER_VERTICAL
            : Gravity.END | Gravity.CENTER_VERTICAL;
    celda.setGravity(gravedad);

    return celda;
}
```

Añade los imports que falten:

```java
import android.view.Gravity;
import android.widget.TableLayout;
import android.widget.TableRow;
import android.widget.TextView;
```

Y llama al método al final de `onCreate`:

```java
cargarFilasDinamicas();
```

Notas:

- El método borra las filas de ejemplo del XML antes de insertar las nuevas.
- Para producir filas desde SQLite, cambia el `String[][]` por tu lista de objetos y recorre los objetos en el bucle.
- El ancho de las columnas debe ser coherente: si dos filas usan pesos distintos, las columnas dejarán de estar alineadas.
- Para combinar celdas usa `android:layout_span="2"` en un hijo de `TableRow`. Con pesos y estilos puede ser más predecible definir el ancho explícito de esa fila.

### 2.3 Tabla con RecyclerView (recomendada para listas largas)

`RecyclerView` recicla las vistas de las filas y está pensado para listas grandes. La cabecera se mantiene fija fuera del `RecyclerView` y cada fila es un layout independiente.

#### Layout `activity_tabla_recycler.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.google.android.material.appbar.MaterialToolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        app:title="@string/titulo_tabla_recycler" />

    <!-- Cabecera fija -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/colorSurfaceVariant"
        android:paddingStart="16dp"
        android:paddingTop="12dp"
        android:paddingEnd="16dp"
        android:paddingBottom="12dp">

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="2"
            android:text="@string/col_nombre"
            android:textStyle="bold" />

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="end"
            android:text="@string/col_stock"
            android:textStyle="bold" />

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="end"
            android:text="@string/col_precio"
            android:textStyle="bold" />
    </LinearLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_tabla"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:clipToPadding="false"
        android:paddingBottom="16dp" />
</LinearLayout>
```

#### Fila `item_fila_tabla.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:minHeight="48dp"
    android:gravity="center_vertical"
    android:orientation="horizontal"
    android:paddingStart="16dp"
    android:paddingTop="12dp"
    android:paddingEnd="16dp"
    android:paddingBottom="12dp">

    <TextView
        android:id="@+id/tv_nombre"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="2"
        android:ellipsize="end"
        android:maxLines="2"
        android:textSize="14sp" />

    <TextView
        android:id="@+id/tv_stock"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:gravity="end"
        android:textSize="14sp" />

    <TextView
        android:id="@+id/tv_precio"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:gravity="end"
        android:textSize="14sp" />
</LinearLayout>
```

> La cabecera y la fila usan exactamente los mismos pesos (`2`, `1`, `1`) y el mismo padding horizontal. Si cambias uno, cambia también el otro para que las columnas sigan alineadas.

#### Adaptador `AdaptadorTabla.java`

```java
package com.ejemplo.guiaandroid;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import java.util.List;

public class AdaptadorTabla extends RecyclerView.Adapter<AdaptadorTabla.FilaViewHolder> {

    private final List<String[]> filas;

    public AdaptadorTabla(List<String[]> filas) {
        this.filas = filas;
    }

    @NonNull
    @Override
    public FilaViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View vista = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_fila_tabla, parent, false);
        return new FilaViewHolder(vista);
    }

    @Override
    public void onBindViewHolder(@NonNull FilaViewHolder holder, int position) {
        String[] fila = filas.get(position);
        holder.nombre.setText(fila[0]);
        holder.stock.setText(fila[1]);
        holder.precio.setText(fila[2]);
    }

    @Override
    public int getItemCount() {
        return filas.size();
    }

    public static class FilaViewHolder extends RecyclerView.ViewHolder {
        final TextView nombre;
        final TextView stock;
        final TextView precio;

        public FilaViewHolder(@NonNull View itemView) {
            super(itemView);
            nombre = itemView.findViewById(R.id.tv_nombre);
            stock = itemView.findViewById(R.id.tv_stock);
            precio = itemView.findViewById(R.id.tv_precio);
        }
    }
}
```

En una aplicación real, cambia `String[]` por tu clase modelo (por ejemplo `Producto`) y usa sus getters en `onBindViewHolder`.

#### Activity `TablaRecyclerActivity.java`

```java
package com.ejemplo.guiaandroid;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.DividerItemDecoration;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.google.android.material.appbar.MaterialToolbar;

import java.util.ArrayList;
import java.util.List;

public class TablaRecyclerActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tabla_recycler);

        MaterialToolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(R.string.titulo_tabla_recycler);
        }

        RecyclerView recycler = findViewById(R.id.recycler_tabla);
        recycler.setLayoutManager(new LinearLayoutManager(this));
        recycler.addItemDecoration(
                new DividerItemDecoration(this, DividerItemDecoration.VERTICAL)
        );

        List<String[]> filas = new ArrayList<>();
        filas.add(new String[]{"Manzanas", "12", "1,20 €"});
        filas.add(new String[]{"Pan integral", "5", "2,10 €"});
        filas.add(new String[]{"Leche", "8", "0,95 €"});
        filas.add(new String[]{"Café", "3", "4,50 €"});
        filas.add(new String[]{"Arroz", "20", "1,35 €"});
        filas.add(new String[]{"Aceite de oliva", "4", "8,90 €"});

        recycler.setAdapter(new AdaptadorTabla(filas));
    }

    @Override
    public boolean onSupportNavigateUp() {
        finish();
        return true;
    }
}
```

Registra también esta actividad en el manifiesto:

```xml
<activity
    android:name=".TablaRecyclerActivity"
    android:exported="false"
    android:parentActivityName=".MainActivity" />
```

### 2.4 Buenas prácticas para tablas

- Usa `RecyclerView` cuando el número de filas pueda crecer; `TableLayout` está pensado para tablas pequeñas.
- Mantén los mismos pesos y padding en cabecera y filas para que las columnas queden alineadas.
- Para muchas columnas, envuelve la tabla en un `HorizontalScrollView`; en ese caso es más fiable usar anchos fijos (`android:width`) que pesos.
- Añade `android:ellipsize="end"` y `android:maxLines` a los textos largos.
- Para tablas con miles de filas, usa `ListAdapter` y `DiffUtil` en lugar de `notifyDataSetChanged`.
- No uses una tabla para contenido que en realidad es una lista de elementos con varias líneas; usa tarjetas o un `LinearLayout` vertical.

---

## 3. Sección 2: conexión a base de datos SQLite

### 3.1 Cómo funciona SQLite en Android

Android incluye SQLite, un motor de base de datos local. La forma clásica de usarlo sin librerías externas es `SQLiteOpenHelper`, que se encarga de:

- Crear la base de datos y las tablas la primera vez (`onCreate`).
- Ejecutar migraciones cuando cambia la versión (`onUpgrade`).
- Entregar objetos `SQLiteDatabase` para leer y escribir.

La base de datos es privada de la aplicación: se guarda en su directorio interno y **no necesitas permisos** en el manifiesto. Para este ejemplo usaremos una tabla `productos`:

| Columna | Tipo | Descripción |
|---|---|---|
| `id` | `INTEGER PRIMARY KEY AUTOINCREMENT` | Identificador único. |
| `nombre` | `TEXT NOT NULL` | Nombre del producto. |
| `precio` | `REAL NOT NULL DEFAULT 0` | Precio unitario. |
| `stock` | `INTEGER NOT NULL DEFAULT 0` | Unidades disponibles. |

> Esta sección conecta con una base de datos **local**. Una app no debe conectarse directamente a MySQL o PostgreSQL desde el dispositivo: se usa un servidor/API intermedio (por ejemplo con Retrofit) por seguridad y para no exponer credenciales.

### 3.2 Modelo `Producto.java`

Crea `app/src/main/java/com/ejemplo/guiaandroid/Producto.java`:

```java
package com.ejemplo.guiaandroid;

import androidx.annotation.NonNull;

import java.util.Locale;

public class Producto {

    private long id;
    private String nombre;
    private double precio;
    private int stock;

    public Producto(String nombre, double precio, int stock) {
        this(0, nombre, precio, stock);
    }

    public Producto(long id, String nombre, double precio, int stock) {
        this.id = id;
        this.nombre = nombre;
        this.precio = precio;
        this.stock = stock;
    }

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public double getPrecio() {
        return precio;
    }

    public void setPrecio(double precio) {
        this.precio = precio;
    }

    public int getStock() {
        return stock;
    }

    public void setStock(int stock) {
        this.stock = stock;
    }

    @NonNull
    @Override
    public String toString() {
        // Se usa en el ArrayAdapter del ListView.
        return nombre + " · "
                + String.format(Locale.getDefault(), "%.2f €", precio)
                + " · " + stock + " uds.";
    }
}
```

### 3.3 `DBHelper.java` con el CRUD

Crea `app/src/main/java/com/ejemplo/guiaandroid/DBHelper.java`:

```java
package com.ejemplo.guiaandroid;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.DatabaseUtils;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import java.util.ArrayList;
import java.util.List;

public class DBHelper extends SQLiteOpenHelper {

    private static final String NOMBRE_BD = "inventario.db";
    private static final int VERSION_BD = 1;

    public static final String TABLA_PRODUCTOS = "productos";
    public static final String COL_ID = "id";
    public static final String COL_NOMBRE = "nombre";
    public static final String COL_PRECIO = "precio";
    public static final String COL_STOCK = "stock";

    // Una sola instancia para toda la aplicación.
    private static DBHelper instancia;

    public static synchronized DBHelper getInstance(Context context) {
        if (instancia == null) {
            instancia = new DBHelper(context.getApplicationContext());
        }
        return instancia;
    }

    private DBHelper(Context context) {
        super(context, NOMBRE_BD, null, VERSION_BD);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        String sql = "CREATE TABLE " + TABLA_PRODUCTOS + " (" +
                COL_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
                COL_NOMBRE + " TEXT NOT NULL, " +
                COL_PRECIO + " REAL NOT NULL DEFAULT 0, " +
                COL_STOCK + " INTEGER NOT NULL DEFAULT 0" +
                ")";
        db.execSQL(sql);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // Ejemplo didáctico: se pierden los datos.
        // En producción, migra los datos con ALTER TABLE (ver 3.6).
        db.execSQL("DROP TABLE IF EXISTS " + TABLA_PRODUCTOS);
        onCreate(db);
    }

    // ---------- CREATE ----------

    public long insertar(Producto producto) {
        SQLiteDatabase db = getWritableDatabase();
        return db.insert(TABLA_PRODUCTOS, null, valoresDe(producto));
    }

    // ---------- READ ----------

    public List<Producto> obtenerTodos() {
        List<Producto> productos = new ArrayList<>();
        SQLiteDatabase db = getReadableDatabase();

        try (Cursor cursor = db.query(
                TABLA_PRODUCTOS,
                null,
                null,
                null,
                null,
                null,
                COL_NOMBRE + " COLLATE NOCASE ASC")) {

            int idxId = cursor.getColumnIndexOrThrow(COL_ID);
            int idxNombre = cursor.getColumnIndexOrThrow(COL_NOMBRE);
            int idxPrecio = cursor.getColumnIndexOrThrow(COL_PRECIO);
            int idxStock = cursor.getColumnIndexOrThrow(COL_STOCK);

            while (cursor.moveToNext()) {
                productos.add(new Producto(
                        cursor.getLong(idxId),
                        cursor.getString(idxNombre),
                        cursor.getDouble(idxPrecio),
                        cursor.getInt(idxStock)
                ));
            }
        }

        return productos;
    }

    public Producto obtenerPorId(long id) {
        SQLiteDatabase db = getReadableDatabase();

        try (Cursor cursor = db.query(
                TABLA_PRODUCTOS,
                null,
                COL_ID + " = ?",
                new String[]{String.valueOf(id)},
                null,
                null,
                null)) {

            if (cursor.moveToFirst()) {
                return new Producto(
                        cursor.getLong(cursor.getColumnIndexOrThrow(COL_ID)),
                        cursor.getString(cursor.getColumnIndexOrThrow(COL_NOMBRE)),
                        cursor.getDouble(cursor.getColumnIndexOrThrow(COL_PRECIO)),
                        cursor.getInt(cursor.getColumnIndexOrThrow(COL_STOCK))
                );
            }
        }

        return null;
    }

    public long contar() {
        SQLiteDatabase db = getReadableDatabase();
        return DatabaseUtils.queryNumEntries(db, TABLA_PRODUCTOS);
    }

    // ---------- UPDATE ----------

    public int actualizar(Producto producto) {
        SQLiteDatabase db = getWritableDatabase();
        return db.update(
                TABLA_PRODUCTOS,
                valoresDe(producto),
                COL_ID + " = ?",
                new String[]{String.valueOf(producto.getId())}
        );
    }

    // ---------- DELETE ----------

    public int eliminar(long id) {
        SQLiteDatabase db = getWritableDatabase();
        return db.delete(
                TABLA_PRODUCTOS,
                COL_ID + " = ?",
                new String[]{String.valueOf(id)}
        );
    }

    // ---------- OPERACIONES EN LOTE ----------

    public void insertarVarios(List<Producto> productos) {
        SQLiteDatabase db = getWritableDatabase();
        db.beginTransaction();
        try {
            for (Producto producto : productos) {
                db.insert(TABLA_PRODUCTOS, null, valoresDe(producto));
            }
            db.setTransactionSuccessful();
        } finally {
            db.endTransaction();
        }
    }

    private ContentValues valoresDe(Producto producto) {
        ContentValues valores = new ContentValues();
        valores.put(COL_NOMBRE, producto.getNombre());
        valores.put(COL_PRECIO, producto.getPrecio());
        valores.put(COL_STOCK, producto.getStock());
        return valores;
    }
}
```

Aspectos importantes:

- `getInstance(...)` reutiliza una única instancia; no crees un `DBHelper` en cada consulta.
- Los `Cursor` se cierran con `try-with-resources` para evitar fugas de memoria.
- Las consultas usan `?` como marcador de posición. **Nunca** concatenes datos del usuario en el SQL: así evitas inyección SQL.
- No llames a `close()` después de cada operación. La conexión se mantiene mientras viva la app; si necesitas liberarla (por ejemplo en pruebas), añade un método que cierre y ponga `instancia = null`.
- `insertar` devuelve el id nuevo o `-1` si hubo un error; `actualizar` y `eliminar` devuelven el número de filas afectadas.

---

### 3.4 Pantalla de ejemplo: formulario y listado

#### Layout `activity_base_datos.xml`

Crea `app/src/main/res/layout/activity_base_datos.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.google.android.material.appbar.MaterialToolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        app:title="@string/titulo_base_datos" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="vertical"
        android:padding="16dp">

        <com.google.android.material.textfield.TextInputLayout
            style="@style/Widget.Material3.TextInputLayout.OutlinedBox"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/hint_nombre">

            <com.google.android.material.textfield.TextInputEditText
                android:id="@+id/et_nombre"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="textCapWords"
                android:maxLines="1" />
        </com.google.android.material.textfield.TextInputLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="12dp"
            android:orientation="horizontal">

            <com.google.android.material.textfield.TextInputLayout
                style="@style/Widget.Material3.TextInputLayout.OutlinedBox"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_marginEnd="8dp"
                android:layout_weight="1"
                android:hint="@string/hint_precio">

                <com.google.android.material.textfield.TextInputEditText
                    android:id="@+id/et_precio"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:inputType="numberDecimal"
                    android:maxLines="1" />
            </com.google.android.material.textfield.TextInputLayout>

            <com.google.android.material.textfield.TextInputLayout
                style="@style/Widget.Material3.TextInputLayout.OutlinedBox"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_marginStart="8dp"
                android:layout_weight="1"
                android:hint="@string/hint_stock">

                <com.google.android.material.textfield.TextInputEditText
                    android:id="@+id/et_stock"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:inputType="number"
                    android:maxLines="1" />
            </com.google.android.material.textfield.TextInputLayout>
        </LinearLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="12dp"
            android:gravity="end"
            android:orientation="horizontal">

            <com.google.android.material.button.MaterialButton
                android:id="@+id/btn_cancelar"
                style="@style/Widget.Material3.Button.TextButton"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/btn_cancelar"
                android:visibility="gone" />

            <com.google.android.material.button.MaterialButton
                android:id="@+id/btn_guardar"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/btn_guardar" />
        </LinearLayout>

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:text="@string/titulo_productos"
            android:textAppearance="?attr/textAppearanceTitleMedium" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="4dp"
            android:text="@string/msg_selecciona_para_editar"
            android:textAppearance="?attr/textAppearanceBodySmall" />

        <ListView
            android:id="@+id/lista_productos"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_marginTop="8dp"
            android:layout_weight="1"
            android:clipToPadding="false"
            android:paddingBottom="16dp" />
    </LinearLayout>
</LinearLayout>
```

#### Activity `BaseDatosActivity.java`

Crea `app/src/main/java/com/ejemplo/guiaandroid/BaseDatosActivity.java`:

```java
package com.ejemplo.guiaandroid;

import android.os.Bundle;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.Toast;

import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;

import com.google.android.material.appbar.MaterialToolbar;
import com.google.android.material.button.MaterialButton;

import java.util.ArrayList;
import java.util.List;

public class BaseDatosActivity extends AppCompatActivity {

    private DBHelper dbHelper;
    private EditText etNombre;
    private EditText etPrecio;
    private EditText etStock;
    private MaterialButton btnGuardar;
    private MaterialButton btnCancelar;
    private ListView listaProductos;

    private ArrayAdapter<Producto> adaptador;
    private final List<Producto> productos = new ArrayList<>();
    private long idEnEdicion = -1L;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_base_datos);

        MaterialToolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(R.string.titulo_base_datos);
        }

        dbHelper = DBHelper.getInstance(this);

        etNombre = findViewById(R.id.et_nombre);
        etPrecio = findViewById(R.id.et_precio);
        etStock = findViewById(R.id.et_stock);
        btnGuardar = findViewById(R.id.btn_guardar);
        btnCancelar = findViewById(R.id.btn_cancelar);
        listaProductos = findViewById(R.id.lista_productos);

        adaptador = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                productos
        );
        listaProductos.setAdapter(adaptador);

        btnGuardar.setOnClickListener(v -> guardarProducto());
        btnCancelar.setOnClickListener(v -> limpiarFormulario());

        // Un toque carga el producto en el formulario para editarlo.
        listaProductos.setOnItemClickListener((parent, view, position, id) ->
                cargarParaEditar(productos.get(position))
        );

        // Mantener pulsado abre la confirmación de borrado.
        listaProductos.setOnItemLongClickListener((parent, view, position, id) -> {
            confirmarEliminacion(productos.get(position));
            return true;
        });

        cargarProductos();
    }

    private void guardarProducto() {
        String nombre = etNombre.getText().toString().trim();
        String precioTexto = etPrecio.getText().toString().trim().replace(',', '.');
        String stockTexto = etStock.getText().toString().trim();

        if (nombre.isEmpty() || precioTexto.isEmpty() || stockTexto.isEmpty()) {
            Toast.makeText(this, R.string.msg_campos_obligatorios, Toast.LENGTH_SHORT).show();
            return;
        }

        double precio;
        int stock;
        try {
            precio = Double.parseDouble(precioTexto);
            stock = Integer.parseInt(stockTexto);
        } catch (NumberFormatException e) {
            Toast.makeText(this, R.string.msg_numero_invalido, Toast.LENGTH_SHORT).show();
            return;
        }

        if (precio < 0 || stock < 0) {
            Toast.makeText(this, R.string.msg_numero_invalido, Toast.LENGTH_SHORT).show();
            return;
        }

        Producto producto = new Producto(nombre, precio, stock);

        if (idEnEdicion == -1L) {
            long id = dbHelper.insertar(producto);
            if (id == -1L) {
                Toast.makeText(this, R.string.msg_error_guardar, Toast.LENGTH_SHORT).show();
                return;
            }
            Toast.makeText(this, R.string.msg_guardado, Toast.LENGTH_SHORT).show();
        } else {
            producto.setId(idEnEdicion);
            int filas = dbHelper.actualizar(producto);
            if (filas == 0) {
                Toast.makeText(this, R.string.msg_error_guardar, Toast.LENGTH_SHORT).show();
                return;
            }
            Toast.makeText(this, R.string.msg_actualizado, Toast.LENGTH_SHORT).show();
        }

        limpiarFormulario();
        cargarProductos();
    }

    private void cargarProductos() {
        productos.clear();
        productos.addAll(dbHelper.obtenerTodos());
        adaptador.notifyDataSetChanged();
    }

    private void cargarParaEditar(Producto producto) {
        idEnEdicion = producto.getId();
        etNombre.setText(producto.getNombre());
        etPrecio.setText(String.valueOf(producto.getPrecio()));
        etStock.setText(String.valueOf(producto.getStock()));
        btnGuardar.setText(R.string.btn_actualizar);
        btnCancelar.setVisibility(View.VISIBLE);
        etNombre.requestFocus();
    }

    private void limpiarFormulario() {
        idEnEdicion = -1L;
        etNombre.setText("");
        etPrecio.setText("");
        etStock.setText("");
        btnGuardar.setText(R.string.btn_guardar);
        btnCancelar.setVisibility(View.GONE);
        etNombre.clearFocus();
    }

    private void confirmarEliminacion(Producto producto) {
        new AlertDialog.Builder(this)
                .setTitle(R.string.titulo_eliminar)
                .setMessage(getString(R.string.msg_eliminar, producto.getNombre()))
                .setPositiveButton(R.string.btn_eliminar, (dialog, which) -> {
                    dbHelper.eliminar(producto.getId());
                    if (producto.getId() == idEnEdicion) {
                        limpiarFormulario();
                    }
                    cargarProductos();
                    Toast.makeText(this, R.string.msg_eliminado, Toast.LENGTH_SHORT).show();
                })
                .setNegativeButton(R.string.btn_cancelar, null)
                .show();
    }

    @Override
    public boolean onSupportNavigateUp() {
        finish();
        return true;
    }
}
```

Registra la actividad en el manifiesto. Añade `android:windowSoftInputMode="adjustResize"` para que el teclado no tape el listado:

```xml
<activity
    android:name=".BaseDatosActivity"
    android:exported="false"
    android:parentActivityName=".MainActivity"
    android:windowSoftInputMode="adjustResize" />
```

### 3.5 Probar la base de datos

1. Abre la pantalla `BaseDatosActivity`.
2. Guarda un par de productos.
3. Gira el dispositivo y comprueba que los datos siguen ahí (están en disco, no en memoria).
4. Cierra y vuelve a abrir la app: los productos deben persistir.
5. Toca un producto para cargarlo, cambia el precio y pulsa **Actualizar**.
6. Mantén pulsado un producto y confirma **Eliminar**.
7. Inspecciona la base de datos en Android Studio: **View → Tool Windows → App Inspection → Database Inspector**, selecciona el proceso de la app y abre `inventario.db → productos`.

> El Database Inspector requiere que la app sea depurable y que el dispositivo/emulador tenga API 26 o superior. Si no ves la base, prueba a hacer una consulta o a reiniciar la inspección mientras la app está en primer plano.

### 3.6 Migraciones: cambiar la base de datos sin perder datos

`onUpgrade` se ejecuta solo cuando aumenta el número de `VERSION_BD`. El ejemplo de la guía borra la tabla para mantenerlo simple. En una app real, migra los datos:

```java
@Override
public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
    if (oldVersion < 2) {
        // Versión 2: nueva columna para clasificar productos.
        db.execSQL("ALTER TABLE " + TABLA_PRODUCTOS + " ADD COLUMN categoria TEXT");
    }

    // Añade aquí los siguientes pasos:
    // if (oldVersion < 3) { ... }
}
```

Pasos para una migración segura:

1. Añade la columna o tabla nueva a `onCreate` para las instalaciones nuevas.
2. Incrementa `VERSION_BD`.
3. Añade el bloque `if (oldVersion < N)` correspondiente en `onUpgrade`.
4. Actualiza el modelo `Producto` y el CRUD si la columna es nueva.
5. Prueba tanto una instalación limpia como una actualización desde la versión anterior.

### 3.7 Buenas prácticas de SQLite

- **Cierra los `Cursor`** siempre (`try-with-resources`), aunque la consulta falle.
- **Usa transacciones** para insertar o actualizar muchos registros de golpe; es mucho más rápido.
- **No hagas operaciones pesadas en el hilo principal.** Para miles de filas, ejecuta el CRUD en un `ExecutorService` y actualiza la interfaz con `runOnUiThread` o un `Handler`.
- **Crea índices** si filtras por una columna con frecuencia:

  ```sql
  CREATE INDEX idx_productos_nombre ON productos(nombre);
  ```

- **No guardes contraseñas en texto plano.** SQLite no cifra los datos por defecto; para información sensible valora SQLCipher o el almacén de claves del sistema.
- **Room** es la librería recomendada por Google para proyectos nuevos: valida las consultas en tiempo de compilación y reduce el código repetitivo. `SQLiteOpenHelper` sigue siendo útil para aprender y para proyectos pequeños.
- La base de datos es privada de la app. Si quieres compartir datos entre usuarios o dispositivos necesitas un backend (API REST), no abrir SQLite directamente.

---

## 4. Sección 3: Navigation Drawer

Un Navigation Drawer es el menú lateral que se abre desde el borde izquierdo (o derecho en idiomas RTL). En XML Views se construye con cuatro piezas:

| Pieza | Función |
|---|---|
| `DrawerLayout` | Contenedor raíz que permite arrastrar el panel lateral. |
| Contenido principal | Toolbar + `FragmentContainerView` donde se muestran las secciones. |
| `NavigationView` | Panel lateral con cabecera (`headerLayout`) y opciones (`menu`). |
| `ActionBarDrawerToggle` | Sincroniza el icono de hamburguesa con el estado del drawer. |

Las secciones se implementan como `Fragment` y se intercambian con `FragmentManager`.

### 4.1 Layout `activity_main.xml`

Sustituye `app/src/main/res/layout/activity_main.xml` por:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <!-- Contenido principal -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <com.google.android.material.appbar.MaterialToolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:title="@string/menu_inicio" />

        <androidx.fragment.app.FragmentContainerView
            android:id="@+id/contenedor_fragmentos"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1" />
    </LinearLayout>

    <!-- Panel lateral -->
    <com.google.android.material.navigation.NavigationView
        android:id="@+id/navigation_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header"
        app:menu="@menu/drawer_menu" />
</androidx.drawerlayout.widget.DrawerLayout>
```

Detalles:

- `layout_gravity="start"` abre el panel desde la izquierda en idiomas LTR y desde la derecha en RTL.
- `fitsSystemWindows="true"` evita que el panel y el contenido queden debajo de las barras del sistema en la mayoría de configuraciones.
- `FragmentContainerView` sustituye al antiguo `FrameLayout` y es el contenedor recomendado para fragments.
- El `DrawerLayout` debe ser la raíz; el contenido principal va primero y el `NavigationView` después.

### 4.2 Cabecera `nav_header.xml`

Crea `app/src/main/res/layout/nav_header.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="?attr/colorPrimaryContainer"
    android:gravity="center_vertical"
    android:orientation="vertical"
    android:paddingStart="24dp"
    android:paddingTop="32dp"
    android:paddingEnd="24dp"
    android:paddingBottom="24dp">

    <ImageView
        android:id="@+id/iv_logo"
        android:layout_width="56dp"
        android:layout_height="56dp"
        android:contentDescription="@string/app_name"
        android:src="@mipmap/ic_launcher" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:text="@string/app_name"
        android:textAppearance="?attr/textAppearanceTitleLarge"
        android:textColor="?attr/colorOnPrimaryContainer" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/nav_header_subtitulo"
        android:textAppearance="?attr/textAppearanceBodyMedium"
        android:textColor="?attr/colorOnPrimaryContainer" />
</LinearLayout>
```

Puedes cambiar el icono por un vector propio en `res/drawable`. El texto y el fondo usan colores del tema Material 3, así que se adaptan a modo claro y oscuro.

### 4.3 Menú `drawer_menu.xml`

Crea `app/src/main/res/menu/drawer_menu.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <group android:checkableBehavior="single">
        <item
            android:id="@+id/nav_inicio"
            android:title="@string/menu_inicio" />

        <item
            android:id="@+id/nav_tabla"
            android:title="@string/menu_tabla" />

        <item
            android:id="@+id/nav_base_datos"
            android:title="@string/menu_base_datos" />

        <item
            android:id="@+id/nav_acerca"
            android:title="@string/menu_acerca" />
    </group>
</menu>
```

- `checkableBehavior="single"` permite que solo haya una sección marcada a la vez.
- Sin `android:icon`, el menú funciona sin recursos adicionales.

**Añadir iconos (opcional):**

1. Clic derecho en `res/drawable` → **New → Vector Asset**.
2. Elige un icono de Material, ponle un nombre (por ejemplo `ic_inicio`, `ic_tabla`, `ic_base_datos`) y pulsa **Next → Finish**.
3. Añade el atributo a cada item:

```xml
android:icon="@drawable/ic_inicio"
```

Los iconos se tiñen automáticamente con el color del tema en `NavigationView`.

---

### 4.4 `MainActivity.java`

Sustituye `app/src/main/java/com/ejemplo/guiaandroid/MainActivity.java` por:

```java
package com.ejemplo.guiaandroid;

import android.os.Bundle;

import androidx.activity.OnBackPressedCallback;
import androidx.annotation.NonNull;
import androidx.appcompat.app.ActionBarDrawerToggle;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.view.GravityCompat;
import androidx.drawerlayout.widget.DrawerLayout;
import androidx.fragment.app.Fragment;

import com.google.android.material.appbar.MaterialToolbar;
import com.google.android.material.navigation.NavigationView;

public class MainActivity extends AppCompatActivity {

    private static final String KEY_SECCION = "seccion_seleccionada";

    private DrawerLayout drawerLayout;
    private NavigationView navigationView;
    private int seccionSeleccionada = R.id.nav_inicio;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        MaterialToolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        drawerLayout = findViewById(R.id.drawer_layout);
        navigationView = findViewById(R.id.navigation_view);

        ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(
                this,
                drawerLayout,
                toolbar,
                R.string.abrir_menu,
                R.string.cerrar_menu
        );
        drawerLayout.addDrawerListener(toggle);
        toggle.syncState();

        navigationView.setNavigationItemSelectedListener(item -> {
            mostrarSeccion(item.getItemId());
            drawerLayout.closeDrawers();
            return true;
        });

        if (savedInstanceState == null) {
            mostrarSeccion(R.id.nav_inicio);
        } else {
            // El FragmentManager restaura el fragment actual.
            // Restauramos el título y el elemento marcado del menú.
            seccionSeleccionada = savedInstanceState.getInt(KEY_SECCION, R.id.nav_inicio);
            navigationView.setCheckedItem(seccionSeleccionada);
            setTitle(tituloDe(seccionSeleccionada));
        }

        // Botón atrás: si el drawer está abierto, lo cierra; si no, sale.
        getOnBackPressedDispatcher().addCallback(this, new OnBackPressedCallback(true) {
            @Override
            public void handleOnBackPressed() {
                if (drawerLayout.isDrawerOpen(GravityCompat.START)) {
                    drawerLayout.closeDrawer(GravityCompat.START);
                } else {
                    setEnabled(false);
                    getOnBackPressedDispatcher().onBackPressed();
                }
            }
        });
    }

    private void mostrarSeccion(int menuId) {
        Fragment fragment;
        int titulo;

        if (menuId == R.id.nav_tabla) {
            fragment = PlaceholderFragment.newInstance(getString(R.string.placeholder_tabla));
            titulo = R.string.menu_tabla;
        } else if (menuId == R.id.nav_base_datos) {
            fragment = PlaceholderFragment.newInstance(getString(R.string.placeholder_base_datos));
            titulo = R.string.menu_base_datos;
        } else if (menuId == R.id.nav_acerca) {
            fragment = PlaceholderFragment.newInstance(getString(R.string.placeholder_acerca));
            titulo = R.string.menu_acerca;
        } else {
            menuId = R.id.nav_inicio;
            fragment = new InicioFragment();
            titulo = R.string.menu_inicio;
        }

        getSupportFragmentManager()
                .beginTransaction()
                .replace(R.id.contenedor_fragmentos, fragment)
                .commit();

        seccionSeleccionada = menuId;
        navigationView.setCheckedItem(menuId);
        setTitle(titulo);
    }

    private int tituloDe(int menuId) {
        if (menuId == R.id.nav_tabla) {
            return R.string.menu_tabla;
        }
        if (menuId == R.id.nav_base_datos) {
            return R.string.menu_base_datos;
        }
        if (menuId == R.id.nav_acerca) {
            return R.string.menu_acerca;
        }
        return R.string.menu_inicio;
    }

    @Override
    protected void onSaveInstanceState(@NonNull Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putInt(KEY_SECCION, seccionSeleccionada);
    }
}
```

Qué hace cada parte:

- `setSupportActionBar(toolbar)` convierte la toolbar en la ActionBar de la actividad.
- `ActionBarDrawerToggle` añade el icono de hamburguesa, abre/cierra el panel y guarda su estado al girar.
- `setNavigationItemSelectedListener` decide qué fragment mostrar y cierra el drawer.
- `mostrarSeccion` centraliza el cambio de fragment, el título y el elemento marcado.
- `onSaveInstanceState` conserva la sección seleccionada al rotar; el fragment lo restaura el `FragmentManager`.

> En este ejemplo las secciones del drawer se reemplazan sin `addToBackStack`. Así, el botón atrás cierra el drawer o sale de la app, que es el comportamiento habitual para destinos principales. Si quisieras un historial interno, añade `.addToBackStack(null)` y controla el retroceso con `getSupportFragmentManager().popBackStack()`.

### 4.5 Fragment de inicio

#### Layout `fragment_inicio.xml`

Crea `app/src/main/res/layout/fragment_inicio.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="24dp">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/inicio_titulo"
            android:textAppearance="?attr/textAppearanceHeadlineMedium" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="12dp"
            android:text="@string/inicio_texto"
            android:textAppearance="?attr/textAppearanceBodyLarge" />
    </LinearLayout>
</ScrollView>
```

#### `InicioFragment.java`

Crea `app/src/main/java/com/ejemplo/guiaandroid/InicioFragment.java`:

```java
package com.ejemplo.guiaandroid;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class InicioFragment extends Fragment {

    @Nullable
    @Override
    public View onCreateView(
            @NonNull LayoutInflater inflater,
            @Nullable ViewGroup container,
            @Nullable Bundle savedInstanceState
    ) {
        return inflater.inflate(R.layout.fragment_inicio, container, false);
    }
}
```

### 4.6 Fragment reutilizable para las demás secciones

Para que el menú funcione antes de integrar la tabla y la base de datos, usamos un fragment que muestra un texto recibido como argumento.

#### Layout `fragment_placeholder.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:orientation="vertical"
        android:padding="24dp">

        <TextView
            android:id="@+id/tv_placeholder"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:textAppearance="?attr/textAppearanceBodyLarge" />
    </LinearLayout>
</ScrollView>
```

#### `PlaceholderFragment.java`

```java
package com.ejemplo.guiaandroid;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class PlaceholderFragment extends Fragment {

    private static final String ARG_TEXTO = "texto";

    public static PlaceholderFragment newInstance(String texto) {
        PlaceholderFragment fragment = new PlaceholderFragment();
        Bundle argumentos = new Bundle();
        argumentos.putString(ARG_TEXTO, texto);
        fragment.setArguments(argumentos);
        return fragment;
    }

    @Nullable
    @Override
    public View onCreateView(
            @NonNull LayoutInflater inflater,
            @Nullable ViewGroup container,
            @Nullable Bundle savedInstanceState
    ) {
        return inflater.inflate(R.layout.fragment_placeholder, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        TextView texto = view.findViewById(R.id.tv_placeholder);
        texto.setText(requireArguments().getString(ARG_TEXTO, ""));
    }
}
```

### 4.7 Manifiesto final

El manifiesto debe declarar `MainActivity` como lanzadora y las tres pantallas de ejemplo. Este es el archivo completo de referencia:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.GuiaAndroid">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity
            android:name=".TablaActivity"
            android:exported="false"
            android:parentActivityName=".MainActivity" />

        <activity
            android:name=".TablaRecyclerActivity"
            android:exported="false"
            android:parentActivityName=".MainActivity" />

        <activity
            android:name=".BaseDatosActivity"
            android:exported="false"
            android:parentActivityName=".MainActivity"
            android:windowSoftInputMode="adjustResize" />
    </application>

</manifest>
```

### 4.8 Probar el Navigation Drawer

1. Ejecuta la app. Debe aparecer la pantalla de inicio con el icono de hamburguesa.
2. Pulsa el icono o desliza desde el borde izquierdo: debe abrirse el panel.
3. Selecciona **Tabla**, **Base de datos** y **Acerca de**. El título de la toolbar debe cambiar y el elemento debe quedar marcado.
4. Gira el dispositivo: la sección, el título y el estado del menú deben mantenerse.
5. Pulsa atrás con el drawer abierto: se cierra el panel. Pulsa atrás otra vez: se sale de la app.

Si el icono de hamburguesa no aparece:

- Revisa que el tema sea `Theme.Material3.DayNight.NoActionBar` o que no tenga ActionBar.
- Asegúrate de llamar a `setSupportActionBar(toolbar)` antes de crear el `ActionBarDrawerToggle`.
- Comprueba que has llamado a `toggle.syncState()`.

### 4.9 Alternativa con Navigation Component

La guía usa `FragmentManager` para que veas el mecanismo sin dependencias extra. Para proyectos más grandes, Google recomienda Navigation Component:

1. Añade las dependencias:

   ```kotlin
   implementation("androidx.navigation:navigation-fragment:2.8.5")
   implementation("androidx.navigation:navigation-ui:2.8.5")
   ```

2. Crea `res/navigation/nav_graph.xml` con los fragments y sus acciones.
3. En `activity_main.xml`, sustituye el `FragmentContainerView` por un `NavHostFragment`:

   ```xml
   <androidx.fragment.app.FragmentContainerView
       android:id="@+id/nav_host_fragment"
       android:name="androidx.navigation.fragment.NavHostFragment"
       android:layout_width="match_parent"
       android:layout_height="0dp"
       android:layout_weight="1"
       app:defaultNavHost="true"
       app:navGraph="@navigation/nav_graph" />
   ```

4. En `MainActivity`, conecta toolbar, menú y controlador:

   ```java
   NavController navController = Navigation.findNavController(this, R.id.nav_host_fragment);

   AppBarConfiguration config = new AppBarConfiguration.Builder(
           R.id.inicioFragment,
           R.id.tablaFragment,
           R.id.baseDatosFragment
   ).setOpenableLayout(drawerLayout).build();

   NavigationUI.setupActionBarWithNavController(this, navController, config);
   NavigationUI.setupWithNavController(navigationView, navController);
   ```

5. Sobrescribe `onSupportNavigateUp` para delegar en `navController.navigateUp()`.

### 4.10 Nota sobre edge-to-edge

Si usas `targetSdk 35` o superior, Android aplica edge-to-edge por defecto y el contenido puede quedar debajo de la barra de estado o de navegación. Las plantillas actuales de Android Studio suelen incluir el manejo de insets. Si tu toolbar aparece tapada, aplica los inset al contenedor raíz:

```java
View root = findViewById(R.id.drawer_layout);
ViewCompat.setOnApplyWindowInsetsListener(root, (v, insets) -> {
    Insets barras = insets.getInsets(WindowInsetsCompat.Type.systemBars());
    v.setPadding(barras.left, barras.top, barras.right, barras.bottom);
    return insets;
});
```

Imports necesarios:

```java
import android.view.View;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
```

---

## 5. Anexo: integrar las tres piezas

Hasta aquí tienes tres ejemplos independientes:

- `TablaActivity` y `TablaRecyclerActivity` para las tablas.
- `BaseDatosActivity` para el CRUD de SQLite.
- `MainActivity` con el drawer, que de momento muestra fragments de relleno.

En este anexo convertimos la tabla y la base de datos en fragments reales para que el drawer sea una aplicación completa.

### 5.1 Crear `fragment_tabla.xml`

Copia el contenido de `activity_tabla.xml`, elimina el `MaterialToolbar` y deja el `ScrollView` como raíz. El archivo debe quedar así (se muestran la cabecera y una fila; añade las que quieras):

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <TableLayout
        android:id="@+id/tabla"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp"
        android:stretchColumns="*">

        <TableRow android:background="?attr/colorSurfaceVariant">
            <TextView
                style="@style/Celda.Cabecera"
                android:text="@string/col_nombre" />

            <TextView
                style="@style/Celda.Cabecera"
                android:gravity="center_vertical|end"
                android:text="@string/col_stock" />

            <TextView
                style="@style/Celda.Cabecera"
                android:gravity="center_vertical|end"
                android:text="@string/col_precio" />
        </TableRow>
    </TableLayout>
</ScrollView>
```

### 5.2 Crear `TablaFragment.java`

Crea el fragment y adapta el método de la sección 2.2. Los cambios principales son:

- Inflar el layout en `onCreateView` y guardar la referencia a `TablaLayout`.
- Usar `requireContext()` en lugar de `this` al crear vistas.
- Obtener los datos de `DBHelper` en lugar del array de ejemplo.

```java
package com.ejemplo.guiaandroid;

import android.os.Bundle;
import android.view.Gravity;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TableLayout;
import android.widget.TableRow;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

import java.util.List;
import java.util.Locale;

public class TablaFragment extends Fragment {

    private TableLayout tabla;

    @Nullable
    @Override
    public View onCreateView(
            @NonNull LayoutInflater inflater,
            @Nullable ViewGroup container,
            @Nullable Bundle savedInstanceState
    ) {
        View root = inflater.inflate(R.layout.fragment_tabla, container, false);
        tabla = root.findViewById(R.id.tabla);
        cargarProductos();
        return root;
    }

    @Override
    public void onResume() {
        super.onResume();
        // Refresca si se han añadido productos en la otra sección.
        if (tabla != null) {
            cargarProductos();
        }
    }

    private void cargarProductos() {
        // Conserva la cabecera.
        while (tabla.getChildCount() > 1) {
            tabla.removeViewAt(1);
        }

        List<Producto> productos = DBHelper.getInstance(requireContext()).obtenerTodos();

        for (Producto producto : productos) {
            TableRow fila = new TableRow(requireContext());
            fila.addView(crearCelda(producto.getNombre(), 0));
            fila.addView(crearCelda(String.valueOf(producto.getStock()), 1));
            fila.addView(crearCelda(
                    String.format(Locale.getDefault(), "%.2f €", producto.getPrecio()),
                    2
            ));
            tabla.addView(fila);
        }
    }

    private TextView crearCelda(String texto, int columna) {
        TextView celda = new TextView(requireContext());

        TableRow.LayoutParams params = new TableRow.LayoutParams(
                0,
                TableRow.LayoutParams.WRAP_CONTENT,
                1f
        );
        celda.setLayoutParams(params);

        int padding = (int) (10 * getResources().getDisplayMetrics().density);
        celda.setPadding(padding, padding, padding, padding);
        celda.setText(texto);
        celda.setTextSize(14f);
        celda.setBackgroundResource(R.drawable.borde_celda);

        int gravedad = (columna == 0)
                ? Gravity.START | Gravity.CENTER_VERTICAL
                : Gravity.END | Gravity.CENTER_VERTICAL;
        celda.setGravity(gravedad);

        return celda;
    }
}
```

Si prefieres la tabla con `RecyclerView`, crea `TablaRecyclerFragment` siguiendo el mismo patrón: infla el layout sin toolbar, crea el adaptador con la lista de `Producto` y llama a `notifyDataSetChanged()` después de recargar.

### 5.3 Crear `fragment_base_datos.xml`

Copia `activity_base_datos.xml`, elimina el `MaterialToolbar` y deja el `LinearLayout` con padding como raíz. Los IDs (`et_nombre`, `et_precio`, `et_stock`, `btn_guardar`, `btn_cancelar`, `lista_productos`) deben mantenerse iguales. No olvides quitar del layout el título redundante si el título ya lo pone la toolbar de `MainActivity`.

### 5.4 Crear `BaseDatosFragment.java`

El fragment reutiliza casi todo el código de `BaseDatosActivity`. Estos son los cambios que debes aplicar:

1. En `onCreateView`, infla `fragment_base_datos`.
2. En `onViewCreated`, asigna `dbHelper = DBHelper.getInstance(requireContext())` y busca las vistas con `view.findViewById(...)`.
3. En los `Toast` y en `AlertDialog.Builder`, usa `requireContext()`.
4. Elimina `setSupportActionBar`, `setDisplayHomeAsUpEnabled` y `onSupportNavigateUp`; eso lo gestiona la actividad.
5. Mantén los métodos `guardarProducto`, `cargarProductos`, `cargarParaEditar`, `limpiarFormulario` y `confirmarEliminacion`.

Esqueleto:

```java
package com.ejemplo.guiaandroid;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.EditText;
import android.widget.ListView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

import com.google.android.material.button.MaterialButton;

import java.util.ArrayList;
import java.util.List;

public class BaseDatosFragment extends Fragment {

    private DBHelper dbHelper;
    private EditText etNombre;
    private EditText etPrecio;
    private EditText etStock;
    private MaterialButton btnGuardar;
    private MaterialButton btnCancelar;
    private ListView listaProductos;
    private ArrayAdapter<Producto> adaptador;
    private final List<Producto> productos = new ArrayList<>();
    private long idEnEdicion = -1L;

    @Nullable
    @Override
    public View onCreateView(
            @NonNull LayoutInflater inflater,
            @Nullable ViewGroup container,
            @Nullable Bundle savedInstanceState
    ) {
        return inflater.inflate(R.layout.fragment_base_datos, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        dbHelper = DBHelper.getInstance(requireContext());

        etNombre = view.findViewById(R.id.et_nombre);
        etPrecio = view.findViewById(R.id.et_precio);
        etStock = view.findViewById(R.id.et_stock);
        btnGuardar = view.findViewById(R.id.btn_guardar);
        btnCancelar = view.findViewById(R.id.btn_cancelar);
        listaProductos = view.findViewById(R.id.lista_productos);

        adaptador = new ArrayAdapter<>(
                requireContext(),
                android.R.layout.simple_list_item_1,
                productos
        );
        listaProductos.setAdapter(adaptador);

        btnGuardar.setOnClickListener(v -> guardarProducto());
        btnCancelar.setOnClickListener(v -> limpiarFormulario());

        listaProductos.setOnItemClickListener((parent, v, position, id) ->
                cargarParaEditar(productos.get(position))
        );

        listaProductos.setOnItemLongClickListener((parent, v, position, id) -> {
            confirmarEliminacion(productos.get(position));
            return true;
        });

        cargarProductos();
    }

    // Pega aquí los métodos guardarProducto(), cargarProductos(),
    // cargarParaEditar(), limpiarFormulario() y confirmarEliminacion()
    // de BaseDatosActivity, cambiando this por requireContext() donde haga falta.
}
```

### 5.5 Conectar los fragments con el drawer

En `MainActivity.mostrarSeccion`, sustituye los `PlaceholderFragment` por los nuevos fragments:

```java
if (menuId == R.id.nav_tabla) {
    fragment = new TablaFragment();
    titulo = R.string.menu_tabla;
} else if (menuId == R.id.nav_base_datos) {
    fragment = new BaseDatosFragment();
    titulo = R.string.menu_base_datos;
} else if (menuId == R.id.nav_acerca) {
    fragment = PlaceholderFragment.newInstance(getString(R.string.placeholder_acerca));
    titulo = R.string.menu_acerca;
} else {
    menuId = R.id.nav_inicio;
    fragment = new InicioFragment();
    titulo = R.string.menu_inicio;
}
```

Después de esto, las pantallas `TablaActivity`, `TablaRecyclerActivity` y `BaseDatosActivity` ya no son necesarias para navegar. Puedes conservarlas como demos independientes o eliminarlas y quitar sus entradas del manifiesto.

### 5.6 Checklist de integración

- [ ] `DBHelper` se obtiene con `getInstance(...)` y no se crea una instancia por pantalla.
- [ ] Los layouts de los fragments no incluyen toolbar ni flecha de retroceso.
- [ ] `TablaFragment` recarga los datos en `onResume` para reflejar cambios hechos en la base de datos.
- [ ] `BaseDatosFragment` usa `requireContext()` en adaptadores, diálogos y toasts.
- [ ] `MainActivity` cambia de fragment con `replace` y actualiza título y elemento marcado.
- [ ] La sección seleccionada sobrevive a la rotación (`onSaveInstanceState`).
- [ ] El botón atrás cierra el drawer antes de salir.
- [ ] El manifiesto no declara activities que ya no existen.
- [ ] Si usas `targetSdk 35+`, revisa los insets de la barra de estado.

### 5.7 Siguiente paso: arquitectura

Para una aplicación real, separa responsabilidades:

- **Modelo:** `Producto`.
- **Acceso a datos:** `DBHelper` o un `ProductoDao`.
- **Repositorio:** una clase que oculte si los datos vienen de SQLite o de una API.
- **ViewModel:** expone los datos con `LiveData` o `StateFlow` y sobrevive a cambios de configuración.
- **UI:** activities, fragments y adaptadores.

También puedes migrar SQLite a **Room** sin cambiar la interfaz: Room genera el DAO y valida las consultas en compilación.

---

## 6. Pruebas y solución de problemas

### 6.1 Checklist de pruebas

**Tabla**

- [ ] La cabecera y las filas están alineadas en modo claro y oscuro.
- [ ] Los textos largos no rompen el ancho de las columnas.
- [ ] La tabla se desplaza si tiene más filas de las que caben.
- [ ] `RecyclerView` muestra todas las filas y se desplaza con fluidez.
- [ ] Al girar la pantalla no se duplican filas.

**Base de datos**

- [ ] Insertar un producto y verlo en el listado.
- [ ] Cerrar y abrir la app: los datos persisten.
- [ ] Editar un producto y comprobar el cambio.
- [ ] Borrar un producto y comprobar que desaparece.
- [ ] Los campos vacíos o con texto no numérico muestran un aviso y no guardan.
- [ ] El Database Inspector muestra la tabla `productos`.
- [ ] Al girar la pantalla no se duplican ni se pierden datos.

**Navigation Drawer**

- [ ] El icono de hamburguesa abre y cierra el panel.
- [ ] El gesto de deslizar desde el borde también funciona.
- [ ] Cada opción cambia el fragment y el título de la toolbar.
- [ ] Solo hay una opción marcada a la vez.
- [ ] Atrás cierra el drawer antes de salir de la app.
- [ ] La sección actual se conserva al girar el dispositivo.
- [ ] El panel se adapta a RTL si cambias el idioma.

**Integración**

- [ ] Añadir un producto en la sección de base de datos y verlo en la tabla al cambiar de sección.
- [ ] La app no falla si la tabla está vacía.
- [ ] No quedan `PlaceholderFragment` en las secciones que ya tienen contenido real.
- [ ] El manifiesto no referencia activities eliminadas.

### 6.2 Problemas con tablas

| Problema | Causa probable | Solución |
|---|---|---|
| Las columnas no están alineadas | Pesos o padding distintos entre cabecera y filas | Usa los mismos `layout_weight` y el mismo padding en cabecera y celdas. |
| La tabla se corta por abajo | El `TableLayout` no está dentro de un scroll | Envuelve la tabla en un `ScrollView`. |
| Los bordes no se ven o se ven mal en oscuro | Color fijo o atributo de tema no disponible | Usa `?attr/colorOutlineVariant` o un color con contraste suficiente. |
| La tabla es más ancha que la pantalla | Demasiadas columnas | Usa `HorizontalScrollView` y anchos fijos para las celdas. |
| `RecyclerView` aparece vacío | Falta `LayoutManager` o adaptador | Llama a `setLayoutManager(...)` y `setAdapter(...)`. |
| `ClassCastException` al añadir una fila | Se añade una vista que no es `TableRow` | Añade únicamente `TableRow` al `TableLayout`. |
| Las filas se añaden con parámetros raros | Se usa `LinearLayout.LayoutParams` en una celda | Usa `TableRow.LayoutParams`. |

### 6.3 Problemas con SQLite

| Problema | Causa probable | Solución |
|---|---|---|
| `no such table: productos` | La base ya existía con otra versión/esquema y `onCreate` no se volvió a ejecutar | Sube `VERSION_BD` y migra, o desinstala la app / borra sus datos durante el desarrollo. |
| `onUpgrade` no se ejecuta | No cambiaste `VERSION_BD` | Incrementa el número de versión cada vez que cambie el esquema. |
| `CursorWindow: Window is full` o fugas | Cursores sin cerrar | Usa `try-with-resources` y recorre solo lo necesario. |
| `database is locked` | Varias escrituras concurrentes o cursor abierto | Centraliza el acceso en un único `DBHelper` y cierra los cursores. |
| Los datos desaparecen | Se desinstaló la app, se borraron datos o se recreó la base | Revisa el nombre y la versión; la base local se borra al desinstalar. |
| `Cannot resolve symbol DBHelper` | Falta el import o la clase está en otro paquete | Comprueba el `package` y añade el import correcto. |
| El precio con coma no se guarda | `Double.parseDouble` espera un punto | Convierte la coma con `replace(',', '.')` antes de parsear. |
| Operaciones lentas con muchas filas | CRUD en el hilo principal | Usa un `ExecutorService` o migra a Room con corrutinas/`LiveData`. |

Para empezar de cero con el esquema durante el desarrollo puedes borrar los datos de la app:

```bash
adb shell pm clear com.ejemplo.guiaandroid
```

> `pm clear` elimina todos los datos de la app, no solo la base de datos. Úsalo solo en desarrollo.

### 6.4 Problemas con el Navigation Drawer

| Problema | Causa probable | Solución |
|---|---|---|
| No aparece el icono de hamburguesa | Tema con ActionBar o toolbar no configurada | Usa `Theme.Material3.DayNight.NoActionBar`, llama a `setSupportActionBar(toolbar)` y a `toggle.syncState()`. |
| El drawer no se abre | `NavigationView` no tiene `layout_gravity="start"` o el `DrawerLayout` no es la raíz | Revisa `activity_main.xml`. |
| El menú no marca la opción | Falta `checkableBehavior="single"` | Añade el grupo al XML y llama a `setCheckedItem(...)` si es necesario. |
| El fragment se superpone | Se usa `add` en lugar de `replace` o el contenedor es incorrecto | Usa `replace(R.id.contenedor_fragmentos, fragment)`. |
| El título no cambia | Se modifica la toolbar directamente y no la ActionBar | Usa `setTitle(...)` después de `setSupportActionBar`. |
| La app falla al rotar | Fragment sin constructor público o estado mal gestionado | Deja un constructor público sin argumentos y guarda la sección con `onSaveInstanceState`. |
| Atrás no cierra el drawer | No se intercepta el retroceso | Usa `OnBackPressedCallback` y comprueba `isDrawerOpen`. |
| El contenido se mete bajo la barra de estado | Edge-to-edge con `targetSdk 35+` | Aplica los insets del sistema al contenedor raíz. |
| `InflateException` en `NavigationView` | Tema no compatible o dependencia de Material ausente | Usa un tema Material 3 y añade `com.google.android.material:material`. |
| `FragmentContainerView` no se infla | Falta la dependencia de fragments | Añade `androidx.fragment:fragment`. |

### 6.5 Errores generales de Android Studio

| Error | Solución rápida |
|---|---|
| `Cannot resolve symbol R` | **Build → Clean Project**, **Rebuild Project** y revisa errores de XML. |
| `resource not found` | Comprueba la carpeta (`layout`, `menu`, `drawable`), el nombre en minúsculas y la referencia `@layout/...`. |
| `Duplicate class` | Revisa dependencias repetidas y ejecuta **Sync Project with Gradle Files**. |
| `You need to use a Theme.AppCompat theme` | Cambia el tema a uno de Material 3/AppCompat y asegúrate de que el activity hereda de `AppCompatActivity`. |
| `android:exported needs to be explicitly specified` | Añade `android:exported="true"` solo a la actividad lanzadora y `false` a las demás. |
| Cambios que no se reflejan | Desinstala la app del emulador o ejecuta **Build → Clean Project**; si cambiaste el esquema, borra los datos de la app. |

### 6.6 Resumen de archivos por funcionalidad

| Funcionalidad | Archivos principales |
|---|---|
| Tabla estática | `activity_tabla.xml`, `TablaActivity.java`, `styles.xml`, `borde_celda.xml` |
| Tabla dinámica | `activity_tabla.xml`, `TablaActivity.java` |
| Tabla con RecyclerView | `activity_tabla_recycler.xml`, `item_fila_tabla.xml`, `AdaptadorTabla.java`, `TablaRecyclerActivity.java` |
| Base de datos | `Producto.java`, `DBHelper.java`, `activity_base_datos.xml`, `BaseDatosActivity.java` |
| Navigation Drawer | `activity_main.xml`, `nav_header.xml`, `drawer_menu.xml`, `MainActivity.java` |
| Integración final | `fragment_tabla.xml`, `TablaFragment.java`, `fragment_base_datos.xml`, `BaseDatosFragment.java` |

### 6.7 Recursos oficiales

- [SQLiteOpenHelper](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper)
- [Guardar datos con Room](https://developer.android.com/training/data-storage/room)
- [RecyclerView](https://developer.android.com/develop/ui/views/layout/recyclerview)
- [Navigation Drawer (Material 3)](https://m3.material.io/components/navigation-drawer/overview)
- [Navigation Component](https://developer.android.com/guide/navigation)
- [Material 3](https://m3.material.io/)
- [Database Inspector](https://developer.android.com/studio/inspect/database)

### Conclusión

Ya tienes las tres piezas y cómo unirlas:

1. **Tabla:** `TableLayout` para tablas pequeñas o `RecyclerView` para listas grandes, con cabecera y pesos coherentes.
2. **Base de datos:** `SQLiteOpenHelper` con un CRUD seguro, cursores cerrados y migraciones versionadas.
3. **Navigation Drawer:** `DrawerLayout` + `NavigationView` + fragments, con toolbar, estado y retroceso bien gestionados.

El siguiente paso recomendado es migrar el acceso a datos a **Room**, añadir un **ViewModel** y cubrir el CRUD con pruebas instrumentadas. Así la aplicación seguirá siendo mantenible cuando crezca.
