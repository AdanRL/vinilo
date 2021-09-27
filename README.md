![Git](https://img.shields.io/badge/git%20-%23F05033.svg?&style=for-the-badge&logo=git&logoColor=white)![GitHub](https://img.shields.io/badge/github%20-%23121011.svg?&style=for-the-badge&logo=github&logoColor=white)![HTML5](https://img.shields.io/badge/html5-%23E34F26.svg?style=for-the-badge&logo=html5&logoColor=white)![CSS3](https://img.shields.io/badge/css3-%231572B6.svg?style=for-the-badge&logo=css3&logoColor=white)[![Visual Studio](https://badgen.net/badge/icon/visualstudio?icon=visualstudio&label)](https://visualstudio.microsoft.com)
# Apuntes: Disco de Vinilo
## Autor: :rocket:Adán de la Rosa Lugo:rocket:
## Fecha: 25/09/2021 --Semana 2--

### 1. Vite y scaffolding del proyecto.
Para este proyecto, haremos uso de Vite, un empaquetador que nos facilitará el trabajo automatizando muchos procesos de desarrollo como creaccion de la estructura de directorios del proyecto o pasar codigo postcss a css nativo.La forma simple de utilizar vite seria mediante el comando vite nombreProyecto , sin embargo, esto nos crearia una estructura de directorios que no nos interesa. Para ello, modificaremos mediante un script la forma en la que vitelo genera.
```bash
#!/bin/bash
# Autor: ManzDev
VERSION=0.1
NAME=$1
JQ=$(which jq)
WGET=$(which wget)
NPM=$(npm --version)
NPM=${NPM::1}

if [ -z "$JQ" ]; then
  echo "jq not detected. Install with sudo apt-get install jq"
  exit
fi

if [ -z "$WGET" ]; then
  echo "wget not detected. Install with sudo apt-get install wget"
  exit
fi

if [[ "$NPM" == "6" ]]; then
  echo "npm7 is required. Install with npm install -g npm"
  exit
fi

if [ $# -eq 0 ]; then
  echo "Sintaxis:"
  echo -e "  $(basename $0) <nombre-carpeta>\n"
  echo "Recuerda que necesitas tener instalado jq, wget y npm7+."
  exit
fi

npm init @vitejs/app $NAME -y -- --template vanilla >/dev/null
cd "$NAME"

# Inicializamos git
git init

cat >apackage.json << EOF
{
  "scripts": {
    "build": "rm -rf dist && vite build",
    "deploy": "gh-pages -d dist"
  },
  "keywords": [],
  "license": "ISC"
}
EOF

jq -s '.[0] * .[1]' package.json apackage.json >package2.json
rm apackage.json package.json index.html
mv package2.json package.json
rm main.js style.css favicon.svg

mkdir public
mkdir -p src/{components,assets}
touch src/index.{css,js}

cat >.gitignore << EOF
node_modules/
dist/
EOF

cat >postcss.config.js << EOF
module.exports = {
  plugins: {
    "postcss-nesting": true
  }
};
EOF

cat >src/index.html << EOF
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <link rel="stylesheet" href="./index.css">
  <script type="module" src="./index.js"></script>
</head>
<body>

</body>
</html>
EOF

cat >vite.config.js << EOF
const path = require("path");
const mode = process.env.NODE_ENV === "production" ? "production" : "development";
const base = mode === "production" ? "/" + path.basename(process.cwd()) + "/" : "/";

module.exports = {
  root: "src",
  base,
  mode,
  publicDir: "../public",
  build: {
    outDir: "../dist",
    assetsDir: "./"
  }
};
EOF

cat >.eslintrc.js << EOF
module.exports = {
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": [
    "standard"
  ],
  "parserOptions": {
    "ecmaVersion": 12,
    "sourceType": "module"
  },
  "rules": {
    "quotes": ["error", "double"],
    "semi": ["error", "always"],
    "comma-dangle": ["error", "only-multiline"],
    "space-before-function-paren": ["error", "never"]
  }
};
EOF

cat >.stylelintrc << EOF
{
  "extends": "stylelint-config-standard",
  "rules": {
    "declaration-colon-newline-after": "always-multi-line",
    "selector-type-no-unknown": null,
    "property-no-unknown": [true, {
        "ignoreProperties": ["content-visibility"]
    }],
    "selector-nested-pattern": "^&",
    "no-descending-specificity": null,
    "no-eol-whitespace": null,
    "declaration-empty-line-before": null
  }
}
EOF

npm install --save --package-lock-only --no-package-lock -D \
    stylelint stylelint-config-standard \
    eslint eslint-config-standard eslint-plugin-import eslint-plugin-node eslint-plugin-promise \
    postcss-nesting \
    gh-pages

cat << EOF

¡Listo! Para empezar, escribe:

cd $NAME
git remote add origin <repo>
npm install
npm run dev

EOF
```
#### Instalando dependencias de desarrollo y configuración de proyecto.
Una vez instalado el script, lo ejecutaremos como ```mkweb nombreProyecto```, agregamos el enlace remoto a nuestro repo en github ```git remote add origin git@github.com:AdanRL/vinilo.git``` y hacemos un ```npm install``` para instalar todas las dependencias. 

En el fichero vite.config.js cargamos la configuración de arranque de vite.
```js
const path = require("path");
const mode = process.env.NODE_ENV === "production" ? "production" : "development";
const base = mode === "production" ? "/" + path.basename(process.cwd()) + "/" : "/";

module.exports = {
  root: "src",
  base,
  mode,
  publicDir: "../public",
  build: {
    outDir: "../dist",
    assetsDir: "./"
  }
};
```
##### Postcss
Tambien, tendremos que instalar el preprocesador de css, en este caso postcss(recordar instalar con la opcion -D como opcion de desarrollo) y los plugins que se que se necesiten.
```npm install -D postcss-cli postcss-nesting```.
La configuración se cargará en el fichero postcss.config.js donde se encontrarán todos los plugins que se usarán.
```js
module.exports = {
  plugins: {
    "postcss-nesting": true
  }
};
```
NOTA: el fichero de configuración tambien se podria crear como un json con el nombre: .postcssrc
```json
{
  "plugins": {
    "postcss-nesting": true,
    "potcss-easy-import": {
        "prefix": "_",
        "extensions": [".pcss", ".css"]
    }
  }
}
```

##### Otros plugins postcss:
* postcss-easy-import: permite hacer @imports de varios ficheros a la vez 
* postcss-mixins: Permite crear funciones reutilizables en CSS.
* postcss-preset-env: Conjunto de plugins, permite utilizar caracteristicas CSS oficiales que no estan implementadas aun en los navegadores.

#### Configuración de Linterns
Los linterns son necesarios para desarrollar un código limpio y con estilo que sea entendible para otras personas. En este caso, haremos uso de stylelint si se instalase manualmente: ```npm install -D stylelint stylelint-config-standard``` El fichero de configuracion se encuentra en .stylelintrc donde pondremos todas las reglas que queramos utilizar.
```json
{
  "extends": "stylelint-config-standard",
  "rules": {
    "declaration-colon-newline-after": "always-multi-line",
    "selector-type-no-unknown": null,
    "property-no-unknown": [true, {
        "ignoreProperties": ["content-visibility"]
    }],
    "selector-nested-pattern": "^&",
    "no-descending-specificity": null,
    "no-eol-whitespace": null,
    "declaration-empty-line-before": null
  }
}
```
#### Scripts NPM.
Tambien tenemos configurado en el package.json los scripts necesarios para el desarrollo que ejecutaremos con ```npm run xxx```

```json
  "scripts": {
    "dev": "vite",
    "build": "rm -rf dist && vite build",
    "serve": "vite preview",
    "deploy": "gh-pages -d dist"
  }
```

* dev: Nos permite ejecutar el servidor que se quedará escuchando los cambios que realicemos en nuestro codigo.
* build: Crea la carpeta dist con los ficheros finales procesados listos para subir a produccion.
* serve:
* deploy: Permite publicar en github en nuestro repo remoto creando la rama gh-pages y lo publica


### 2. Git y creación de ramas.
Crearemos dos ramas para el proyecto, una de develop donde desarrollaremos el codigo y otra doc, donde realizaremos la documentación. Recordar crear el .gitignore añadiendo los elementos que se quieran ignorar a la hora de subir a github.

### 3. Publicar en github-pages.
Primero instalamos la dependencia ```npm install -D gh-pages```, y agregamos el script al package.json (Ver script deploy en el apartado de scripts NPM).
Hay que hacer un build y luego un deploy para subirlo a github-pages.





