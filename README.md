# Metin2-quest

Sigue esta guía para trabajar tus quest como un profesional.<br/>
Esta guía te ayudará a:
- Organizar mejor tus quest
- Hacer más cómoda la compilación de tus quest
- Hacer el trabajo mucho más rápido

El servidor que se tomará de base es el de Marty Sama 5.4. Si tienes una versión inferior no hay problema.<br/>
En caso de que tengas otro tipo de base solo debes adaptarlo.

## 1. Crea un acceso directo a tu carpeta quest

Digita estos comandos en tu PuTTY:

```
cd /
ln -s /usr/home/svfiles/main/srv1/share/locale/germany/quest /
```

Debes poner tu ruta.<br/>
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

Para compilar las quest hay dos archivos, el qc, que es el que compila las quest del lado del servidor.<br/>
Antes de ejecutar el qc, se ejecuta un pre_qc, que hace revisión de sintaxis y utilidades.<br/>
El archivo se encuentra en quest/pre_qc.py. Es un archivo de Python en versión 2.7 o 3+. En el repo dejé la versión 3.<br/>
Solo debes asegurarte de que el pre_qc.py sea el revisado por Marty Sama.<br/>
Tu máquina debe tener instalado Python 3+.<br/>
Una utilidad son los comentarios, puedes omitir la compilación de una quest anteponiendo #.<br/>
La utilidad especial del pre_qc.py es la definición del token, la que encuentras en algunas quest así:
```
define GENERAL_STORE 9003

when GENERAL_STORE.click begin
...
```
Estas variables se reemplazarán por sus correspondientes números y todas las quest se almacenarán en una carpeta llamada pre_qc.

## 3. Compila las quest en pila

Al ejecutar main/admin_panel.sh existe la opción 777 para compilar quest. Debes revisar que el bloque 777 sea así:
```bash
777|quest)
	cd $v_mt2f/$v_foldername/share/locale/$v_localename/quest
	chmod u+x qc
	$v_bin pre_qc.py -ac
	cd $v_base
	cecho "quest completed"
;;
```
En caso de que no utilices Marty Sama, puedes hacer estos comandos:
```bash
	cd /quest
	chmod u+x qc #da permisos de ejecución al qc en caso de que nos los tenga
	python3 pre_qc.py -ac #ejecuta el pre_qc.py con los parámetros a(all) y c(compile).
	cd / #si la ruta anterior está guardada en una variable, mucho mejor
	cecho "quest completed"
```

En el código del pre_qc.py se encuentra el archivo que almacena las quest, que es el questlist.
```py
QUESTLIST_PATH_DFT="./quest_list"
WORKINGPATH_DFT="./pre_qc"
```

Una de las ventajas de usar este pre_qc.py es que si obtienes un error al compilar, no pasará a la siguiente quest sino que se detendrá ahí y lo podremos visualizar para corregir.
