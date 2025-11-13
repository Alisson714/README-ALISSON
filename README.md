# Ciclo de CI/CD: Explicación Detallada con Ejemplo Práctico

## Introducción

El ciclo de CI/CD (Integración Continua / Despliegue Continuo) es una metodología de desarrollo de software que automatiza el proceso de integración de cambios en el código fuente, ejecución de pruebas y despliegue de aplicaciones. Esto permite entregar software de manera más rápida, confiable y frecuente, reduciendo errores humanos y mejorando la calidad del producto.

El enfoque principal es integrar cambios pequeños y frecuentes en lugar de grandes actualizaciones, lo que facilita la detección temprana de problemas.

## Etapas del Ciclo CI/CD

El ciclo CI/CD se divide en varias etapas secuenciales. A continuación, se describen las principales, enfocándonos hasta la construcción del paquete (Build), como se solicita:

1. **Código Fuente (Source)**: Los desarrolladores escriben y modifican el código en un repositorio de control de versiones como Git. Cada cambio se sube (push) al repositorio central.

2. **Integración Continua (CI)**: Automatiza la fusión de cambios de múltiples desarrolladores. Incluye la compilación del código y la ejecución de pruebas automáticas para verificar que los cambios no rompan la funcionalidad existente.

3. **Construcción del Paquete (Build)**: En esta etapa, se compila el código fuente en un artefacto o paquete ejecutable (por ejemplo, un archivo JAR en Java, un bundle en JavaScript o un contenedor Docker). Este paquete es lo que se despliega posteriormente.

4. **Pruebas**: Se ejecutan diferentes tipos de pruebas para asegurar la calidad:
   - **Pruebas Unitarias**: Verifican funciones individuales.
   - **Pruebas de Integración**: Verifican la interacción entre componentes.
   - **Pruebas de Rendimiento**: Evalúan el comportamiento bajo carga.

5. **Despliegue Continuo (CD)**: Automatiza el despliegue del paquete a entornos de staging o producción. Incluye rollback automático en caso de fallos.

En este documento, nos enfocamos en las etapas hasta la construcción del paquete, incluyendo ejemplos de pruebas.

## Ejemplo Práctico: Aplicación Node.js con CI/CD usando GitHub Actions

Vamos a crear una aplicación simple en Node.js que realiza una suma, incluye pruebas unitarias y un proceso de construcción. Usaremos GitHub Actions para automatizar el CI/CD hasta el build.

### Paso 1: Configuración del Proyecto

Crea un directorio para tu proyecto (por ejemplo, `mi-app-ci-cd`).

Dentro del directorio, crea los siguientes archivos:

**package.json** (define dependencias y scripts):

```json
{
  "name": "mi-app-ci-cd",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "build": "webpack --mode production",
    "test": "jest"
  },
  "devDependencies": {
    "jest": "^29.7.0",
    "webpack": "^5.89.0",
    "webpack-cli": "^5.1.4"
  }
}
```

**index.js** (código principal de la aplicación):

```javascript
function suma(a, b) {
  return a + b;
}

console.log('Resultado de 5 + 3:', suma(5, 3));

module.exports = { suma };
```

**webpack.config.js** (configuración para el build):

```javascript
const path = require('path');

module.exports = {
  entry: './index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

### Paso 2: Ejemplos de Pruebas

Crea el archivo **suma.test.js** para pruebas unitarias usando Jest:

```javascript
const { suma } = require('./index');

// Prueba unitaria básica
test('suma de 1 + 2 debe ser 3', () => {
  expect(suma(1, 2)).toBe(3);
});

// Prueba con números negativos
test('suma de -1 + 1 debe ser 0', () => {
  expect(suma(-1, 1)).toBe(0);
});

// Prueba de integración simple (simulando uso en otro módulo)
test('suma integrada en un cálculo mayor', () => {
  const resultado = suma(suma(2, 3), 4); // (2+3) + 4 = 9
  expect(resultado).toBe(9);
});
```

Para ejecutar las pruebas localmente:
- Instala dependencias: `npm install`
- Ejecuta pruebas: `npm test`

Esto debería pasar todas las pruebas y mostrar algo como:
```
PASS  ./suma.test.js
  ✓ suma de 1 + 2 debe ser 3 (2 ms)
  ✓ suma de -1 + 1 debe ser 0
  ✓ suma integrada en un cálculo mayor

Test Suites: 1 passed, 1 total
Tests: 3 passed, 3 total
```

### Paso 3: Construcción del Paquete (Build)

El build compila el código en un paquete optimizado.

Ejecuta: `npm run build`

Esto usa Webpack para crear un archivo `bundle.js` en la carpeta `dist/`, que es el paquete listo para despliegue.

### Paso 4: Automatización con GitHub Actions (CI/CD Pipeline)

Para automatizar el proceso, crea la carpeta `.github/workflows/` y dentro un archivo `ci-cd.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout código
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Build package
      run: npm run build

    - name: Upload build artifacts
      uses: actions/upload-artifact@v4
      with:
        name: dist-files
        path: dist/
```

Este pipeline:
- Se activa en push o PR a la rama `main`.
- Instala Node.js y dependencias.
- Ejecuta pruebas.
- Construye el paquete (hasta el build).
- Sube los artefactos (el paquete) para posible despliegue posterior.

### Paso 5: Creación del Repositorio, Subida y Compartir Enlace

Para completar el ejemplo práctico:

1. Inicializa un repositorio Git local: `git init`
2. Agrega todos los archivos: `git add .`
3. Haz un commit: `git commit -m "Initial commit: Agrega aplicación Node.js con CI/CD"`
4. Crea un nuevo repositorio en GitHub (o tu plataforma Git preferida).
5. Agrega el remote: `git remote add origin https://github.com/tuusuario/tu-repo.git` (reemplaza con tu URL).
6. Sube el código: `git push -u origin main`
7. Comparte el enlace del repositorio: Por ejemplo, `https://github.com/tuusuario/tu-repo`

Ahora, cada push activará el pipeline de CI/CD, ejecutando pruebas y construyendo el paquete automáticamente.

Este ejemplo demuestra el ciclo completo hasta la construcción del paquete, incluyendo pruebas, de manera práctica y automatizada.