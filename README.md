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

## 3. Compila las quest en pila con pre_qc.py

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

Una de las ventajas de usar este pre_qc.py es que si obtienes un error al compilar, no pasará a la siguiente quest sino que se detendrá ahí y lo podremos visualizar para corregir.<br/>
#### Nota: una quest es compilada bien cuando se compila el último bloque when o function. Aún así no hay garantía, porque hay un bug donde si pones un caracter especial en el último caracter de un say (o algún componente gráfico quest, como select), tomará como compilado ese bloque pero no compiló la quest. La solución es, si necesariamente quieres una tilde (posible caracter), entonces adiciona un espacio.<br/>
```lua
-- on any file lua
gameforge["es"][12345] = "Texto en español. Atraerá[ENTER]a muchos latinos"
gameforge["es"][12346] = "El evento empezará"
gameforge["es"][12347] = "El evento empezará "

--quest
say(gameforge[pc.get_language()][12345]) #no error pero no compila
say(gameforge[pc.get_language()][12346]) #no error pero no compila
say(gameforge[pc.get_language()][12347]) #bien
```
En los ejemplos anteriores, el índice de la tabla 12345 y 12346, tienen tilde, uno antes de ```[ENTER]``` y el otro al final del texto. Ambos ocasionarán problemas.<br/>
La solución se encuentra en el 123457. Y recuerda que en el caracter 50 hará un salto de línea, así que ten presente para la estética, cortar tus textos hasta los 49 caracteres.<br/>
Los caracteres hasta donde tengo conocido, son todos los que lleven tilde o la cedilla (ç).

## 4. Prepara tu compilador qc

qc es el nombre del binario generado desde el source server en game/src/quest. El lenguaje C++ llama al lua y el lua crea uno extendido que se le denominó quest, poniendo así sus propias reglas.<br/>
Para compilar el qc, solo basta con ir a game/src/quest y ejecutar:
```
gmake clean
gmake all -j5
```
Puedes revisar el Makefile para verificar sus requerimientos.<br/>
De todas maneras no era necesario, ya que en el repo dejo un qc ya compilado con algo adicional.<br/>
En el qc original no puedes hacer esto:
```lua
--otro archivo lua externo
text_table = {
	"text1" = {[1] = "subtext1", [2] = "subtext2", [3] = "subtext3"},
	"text2" = {[1] = "subtext1", [2] = "subtext2", [3] = "subtext3"},
}
--quest
when any_npc.chat.text_table["text1"][1] begin --no compila aquí
	say(text_table["text1"][2])
end
```
En los when donde se ponen textos no se pueden poner tablas con más de una anidación, quiero decir, puedes llamar a ```text_table["text1"]``` (si fuera un texto) pero no a una posición más allá como ```text_table["text1"][1]```. Y si te ha sonado algo parecido, esto se usa en el multilenguaje.<br/>
<br/>
Una vez puesto el qc del repo ya funcionará como debe ser.
## 5. Si el compilador te pide funciones, haz esto
En otras versiones del compilador no matan el proceso cuando no detecta funciones declaradas. Es recomendable dejar que pida las funciones por seguridad, algunas veces el compilador me avisa de funciones que no las he creado y sí las he llamado.<br/>
Si te pide funciones, simplemente las seleccionas en la misma consola (no hagas Ctrl+C) y las pegas en quest_functions.<br/>
Cuando seleccionas cualquier texto en la consola, automáticamente este texto se copia.
## 6. Mejora la estructura de tus variables.
Para multilenguaje, si no tienes una estructura clara, vas a perderte demasiado y gastar mucho tiempo.<br/>
Lo primero que debes tener en cuenta es que los desarrolladores de Metin2 empezaron los textos con variables. Con el tiempo lo cambiaron a índices. La verdad ese cambio mejoró bastante la manera como se manejaban los textos. Así que siempre piensa en manejar índices.<br/>
Todos queremos utilizar los mismos textos del servidor oficial, sobretodo para ahorrar tiempo y que quede bien traducido, entonces hay que crear una estructura que se sincronice con los índices del oficial.<br/>
Si tienes un solo idioma, como si tienes varios, debes hacer esto:

