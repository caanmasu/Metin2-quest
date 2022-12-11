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
Una utilidad son los comentarios, puedes omitir la compilación de una quest anteponiendo # en el ```quest_list```.<br/>
La utilidad especial del ```pre_qc.py``` es la definición del token, la que encuentras en algunas quest así:
```
define GENERAL_STORE 9003

when GENERAL_STORE.click begin
...
```
Estas variables se reemplazarán por sus correspondientes números y todas las quest se almacenarán en una carpeta llamada pre_qc.

## 3. Compila las quest en pila con pre_qc.py

Al ejecutar ```main/admin_panel.sh``` existe la opción 777 para compilar quest. Debes revisar que el bloque 777 sea así:
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

En el código del pre_qc.py se encuentra el archivo que almacena las quest, que es el ```quest_list```.
```py
QUESTLIST_PATH_DFT="./quest_list"
WORKINGPATH_DFT="./pre_qc"
```

Una de las ventajas de usar este ```pre_qc.py``` es que si obtienes un error al compilar, no pasará a la siguiente quest sino que se detendrá ahí y lo podremos visualizar para corregir.<br/>
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
Los caracteres hasta donde tengo conocido, son todos los que lleven tilde o la cedilla (ç).<br/>
<br/>
Al compilar las quest, las carpetas ```object``` y ```pre_qc``` se eliminarán y se generarán unas nuevas.
## 4. Prepara tu compilador qc

```qc``` es el nombre del binario generado desde el source server en ```game/src/quest```. El lenguaje C++ llama al lua y el lua crea uno extendido que se le denominó quest, poniendo así su propia estructura y sintaxis.<br/>
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
Si tienes un solo idioma, como si tienes varios, debes entender esto:
Al iniciar el servidor, se cargará todas las quest, incluidos los archivos de extensión ```lua``` del locale (germany).<br/>
El archivo que conecta con todos los textos es ```translate.lua```. Para adaptar este archivo a uno o varios idiomas deber borrar todo su contenido y agregar esto:
```lua
exportTestForCharset = "üöäÜÖÄß "
gameforge = {}

languages = {"es"} --tu lista de idiomas
for i in languages do
	gameforge[languages[i]] = {}
	dofile(string.format("%s/translate/gameforge/translate_%s.lua", get_locale_base_path(), languages[i]))
	dofile(string.format("%s/translate/new_texts/translate_%s.lua", get_locale_base_path(), languages[i]))
end
```
En esta estructura creo dinámicamente un nuevo índice en la tabla ```gameforge``` por cada idioma. Cada índice es una nueva tabla. Todavía el contenido está vacío.<br/>
```gameforge``` es la tabla que almacena todos los textos.<br/>
<br/>
También se carga dinámicamente cada archivo de translate, es decir, los que están en la estructura del repo.<br/>
Una carpeta llamada ```gameforge``` para todas las variables de texto del juego, las originales. Y luego otra carpeta con nuestros propios textos llamada ```new_texts```.<br/>
Ahora haremos uso del convertidor de variables de cliente del oficial a servidor, autoría de Owsap.<br/>
https://metin2.dev/topic/22862-localestring-localequest-builder/ <br/>
Se debe ejecutar el programa para el ```locale_quest.txt``` (en el repo está el más actualizado hasta el momento) y obtener ```locale_xx.lua``` de los idiomas que necesites.<br/>
Estos archivos ```lua``` debes ponerlos en la carpeta ```gameforge``` y esto es importante:
#### borra la primera línea (```gameforge[your_lang] = {}```) de cada archivo.<br/>
<br/>
Ahora debes crear los mismos archivos en la carpeta ```new_texts``` y poner tus textos. Si no lo entiendes, no importa; en el repo está clara la estructura.<br/>
Van a aparecer varios errores al iniciar el servidor si pusiste ciertos idiomas. Esto es porque hay caracteres inválidos en los translate, solo se debe corregir uno por uno.
## 7. Prepara tu questcategory.txt

Si tienes el sistema de questcategory o questrenewal debes entender que el archivo ```questcategory.txt``` solo funciona con saltos de línea lf (o generado en Unix). Para comprobarlo, debes abrirlo con Notepad++ y en la parte inferior derecha, si lo tienes en ```Windows(CR LF)``` debes cambiarlo a ```Unix (LF)```.

## 8. Deja el nombre de la quest igual que el de los archivos

Esto no es obligatorio pero es mejor manejarlo así. Si tu quest empieza por ```quest test begin```, debería llamarse ```test.quest```.<br/>
Lo normal es que si el archivo tiene la estructura ```quest ... begin```, tenga la extensión ```quest```.<br/>
En cambio, si no tiene estructura quest y solo usas el archivo para librería o almacenar variables, deja la extensión ```lua```.

## 9. Pon la codificación correcta

Si en el juego te aparecen caracteres especiales en tus textos, tal como en vez de mostrar tildes muestra otros caracteres, entonces la codificación no es la correcta.<br/>
Debes abrir el archivo con Notepad++, ir a la barra de herramientas, opción ```Codificación``` y debe estar elegido ```Codificar en ANSI```.<br/>
Si tu codificación está seleccionada en ```Juego de caracteres/Coreano/EUC-KR``` no se puede convertir, fallará la conversión. En ese caso debes copiar todo el contenido del archivo, crear un nuevo archivo en formato ANSI y pegar el contenido.

## 10. Separa questlib.lua de tu propia lib

El archivo ```questlib.lua``` es el principal archivo de librerías de quest. Allí se almacenan variables y funciones globales que son útiles para usar luego en otras partes.<br/>
Recomiendo crear otro archivo, puede ser llamado ```my_questlib.lua``` donde almacenes tus nuevas variables y funciones. Es mejor tener un archivo externo de lib donde lo puedas portar luego a otras creaciones, incluso de esta manera es más organizado.

Para obtener el texto en multilenguaje o no, recuerda la parte del translate, debe ser la siguiente:

```lua
--my_questlib.lua
ENABLE_MULTILANGUAGE_SYSTEM = true

function locale_quest(idx)
	local text = ""
	if ENABLE_MULTILANGUAGE_SYSTEM:
		text = gameforge[pc.get_language()][idx]
	else
		text = gameforge["es"][idx]
	end
	if text == nil or text == "" then
		text = string.format("Text error: %s", idx)
	end
	return text
end
```
En ```"es"``` debes poner tu idioma si no tienes multilenguaje.

## 11. Utiliza locale_quest

Esta función ya existe en el sistema de multilenguaje de Owsap más moderno. Maneja índices de textos. Aunque así tengas o no multilenguaje, usarás esta función por conveniencia, que ya la creamos anteriormente.

Este es un ejemplo de quest donde utilizo locale_quest. Una vez puesta esta quest, debería funcionar. Si no se instaló bien la estructura de los textos, debería aparecer ```Text error: index```. <br/>

Debes compilar con 777.

```lua
define ITEM_CHANGE_NAME 71055

quest change_name begin
	state start begin
		when ITEM_CHANGE_NAME.use begin
			if pc.is_married() then
				syschat(locale_quest(20500))
				return
			end

			if pc.is_polymorphed() then
				syschat(locale_quest(20501))
				return
			end

			if pc.has_guild() then
				syschat(locale_quest(20502))
				return
			end

			if party.is_party() then
				syschat(locale_quest(20503))
				return
			end
			say_title(item_name(item.vnum))
			say()
			say(locale_quest(20504)) 

			local new_name = input()
			
			if string.len(new_name) > 13 or string.len(new_name) < 3 then
				syschat(locale_quest(20505))
				return
			end
			
			if string.find(new_name, " ") then
				syschat(locale_quest(20506))
				return
			end
			
			local k = pc.change_name(new_name)
			if k == 0 then
				syschat(locale_quest(20507))
			elseif k == 1 then
				syschat(locale_quest(20508))
			elseif k == 2 then
				syschat(locale_quest(20509))
			elseif k == 3 then
				syschat(locale_quest(20510))
			elseif k == 4 then
				if pc.count_item(item.vnum) > 0 then
					syschat(locale_quest(20511))
					pc.remove_item(item.vnum)
					pc.setqf("next_time", get_time() + 60*60*24*3)
				end
			elseif k == 5 then
				syschat(locale_quest(20512))
			elseif k == 6 then
				syschat(locale_quest(20513))
			else
				syschat(locale_quest(20514))
			end
		end
	end
end
```

## 12. Crea una carpeta aparte para tus quest

Por lo general guardan las quest en la misma carpeta ```quest``` junto con ```object```, ```pre_qc``` y varios archivos. Recomiendo crear una carpeta aparte. A veces la llaman ```source``` pero recomendaré el nombre ```my_quests```.<br/>
En ```my_quests``` debes crear más carpetas según las categorías que tus quests. Esto lo irás haciendo mientras las vas agregando. Generalmente se crean las categorías ```item```, ```npc```, ```basic```, ```system```, ```dungeon```, etc.
