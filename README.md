# Metin2-quest

Sigue esta guía para trabajar tus quest como un profesional.
Esta guía te ayudará a:
- Organizar mejor tus quest
- Hacer más cómoda la compilación de tus quest
- Hacer el trabajo mucho más rápido

El servidor que se tomará de base es el de Marty Sama 5.4. Si tienes una versión inferior no hay problema.
En caso de que tengas otro tipo de base solo debes adaptarlo.

## 1. Crea un acceso directo a tu carpeta quest

Digita estos comandos en tu PuTTY:

```
cd /
ln -s /usr/home/svfiles/main/srv1/share/locale/germany/quest /
```

Debes poner tu ruta.
Después de ejecutar el comando, para ir a tu carpeta quest ya no es necesario hacer:
```
cd /usr/home/svfiles/main/srv1/share/locale/germany/quest
```

Tan solo debes escribir:
```
cd /quest
```

Verifica que en tu raíz se haya creado un acceso directo a la carpeta.

## 2. Deja listo tu pre_qc.py

Para compilar las quest hay dos archivos, el qc, que es el que compila las quest del lado del servidor.
Antes de ejecutar el qc, se ejecuta un pre_qc, que hace revisión de sintaxis y utilidades.
El archivo se encuentra en quest/pre_qc.py. Es un archivo de Python en versión 2.7 o 3+. En el repo dejé la versión 3.
Solo debes asegurarte de que el pre_qc.py sea el revisado por Marty Sama.
Tu máquina debe tener instalado Python 3+.
\n
La utilidad especial del pre_qc.py es la definición del token, la que encuentras en algunas quest así:
```
define GENERAL_STORE 9003

when GENERAL_STORE.click begin
...
```
Estas variables se reemplazarán por sus correspondientes números y todas las quest se almacenarán en una carpeta llamada pre_qc.

## 3. Compila las quest en pila

Al ejecutar main/admin_panel.sh existe la opción 777 para compilar quest. Debes revisar que los comandos sean:
```bash
777|quest)
	cd $v_mt2f/$v_foldername/share/locale/$v_localename/quest
	chmod u+x qc
	$v_bin pre_qc.py -ac
	cd $v_base
	cecho "quest completed"
;;
```
