# Writeups-CERTUNLP

## Descripción

![Descripcion]()

## Funcionamiento

Antes que nada ejecutamos el servidor Netcat para ver que es lo que hace el software instlado. Al ejecutarlo el servidor te proporciona un encode(linea10) y un texto para encodear(linea11). Por lo tanto vamos a empezar a escribir nuestro script

```
root@W1C3B0X:~/Descargas/ctfs/certunlp/ScriptingWriteups# nc encoderbase.ctf.cert.unlp.edu.ar 5001
1
2   ________________  ________  ___   ____    ____ 
3  / ____/ ____/ __ \/_  __/ / / / | / / /   / __ \
4 / /   / __/ / /_/ / / / / / / /  |/ / /   / /_/ /
5/ /___/ /___/ _, _/ / / / /_/ / /|  / /___/ ____/ 
6\____/_____/_/ |_| /_/  \____/_/ |_/_____/_/      
7                                                  
8
9
10Encode next string with b16encode:
11ocre
12Too late
```

## Como preparar un script

Lo primero de todo vamos a imprimir las 11 primeras lineas ya que la linea 12 no nos hace falta.
Para conectarnos al Netcat necesitamos instalar la libreria pwntools

Aclarar que cada vez que ejecutemos el netcat nos dara una base diferente, el cual son (16,32,64 y 85)

## Script1.py
```
from pwn import *
f = remote('encoderbase.ctf.cert.unlp.edu.ar', 5001) #Aqui nos conectamo al servidor
print(f.recvline())         #Con esto printeamos una linea
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
base = f.recvline()       #Guardamos la base el cual al printear base nos dara este string "Encode next string with b16encode:"
word = f.recvline()
print(base)
print(word)
```
## Output1
```
root@W1C3B0X:~/Descargas/ctfs/certunlp/ScriptingWriteups# python3 tutorial.py 
[+] Opening connection to encoderbase.ctf.cert.unlp.edu.ar on port 5001: Done
b'\n'
b'   ________________  ________  ___   ____    ____ \n'
b'  / ____/ ____/ __ \\/_  __/ / / / | / / /   / __ \\\n'
b' / /   / __/ / /_/ / / / / / / /  |/ / /   / /_/ /\n'
b'/ /___/ /___/ _, _/ / / / /_/ / /|  / /___/ ____/ \n'
b'\\____/_____/_/ |_| /_/  \\____/_/ |_/_____/_/      \n'
b'                                                  \n'
b'\n'
b'\n'
b'Encode next string with b85encode:\n'
b'ruleta\n'
```
El cual nos da esto, el cual solo nos interesa saber si es un base85,64,32 o 16, por lo tanto es hora de aplicar unos split, rstrip,....

## Script2

Añadiremos estas dos variables y printeamos la solución para comprobar si son correctas

```
baseX = base.rstrip(b':\n').split(b' ') #Quitame de la variable 'base' el string ":\n" y divideme el texto en un espacio
print(baseX[4])                         #Por lo tanto me quedaria "Encode[posicion0] next[posicion1] string[posicion2] with[posicion3] b85encode[posicion4]"
                                        #Por lo que a mi me intersa la posicion 4
wordclean = word.rstrip(b'\n')          #Y lo mismo aplicamos a este string, le quitamos el "\n" por lo que se nos quedaria como 'ruleta' el out
print(wordclean)
```

## Output2

```
root@W1C3B0X:~/Descargas/ctfs/certunlp/ScriptingWriteups# python3 tutorial.py 
[+] Opening connection to encoderbase.ctf.cert.unlp.edu.ar on port 5001: Done
b'b16encode'
b'balanza'
```
Por lo cual ya podemos saber la palabra y la base que nos piden en cada momento por lo que vamos a enviarle la informacion que necesita al servicio

## Script3
Antes que nada tenemos que importar la libreria base64 para poder cifrar estas palabras segun el cifrado que nos pidan, Añadimos ```import base64``` en la primera linea
```
import base64
if baseX[4] == b'b64encode':                    #Aqui le decimos si la baseX que en el caso de arriba es b'b16encode' == b'b64encode'
        f.sendline(base64.b64encode(wordclean))             #Si es enviamos esta linea cogiendo el string de wordclean y aplicandole un base64, pero si no es va a la               if baseX[4] == b'b32encode':                                 #siguiente linea y asi sucesivamente hasta que baseX[4] sea = al texto que pongan en los ifs
        f.sendline(base64.b32encode(wordclean))       
if baseX[4] == b'b16encode':
        f.sendline(base64.b16encode(wordclean))
if baseX[4] == b'b85encode':
        f.sendline(base64.b85encode(wordclean))
        
        #A continuacion vamos a recibir las lineas.
        #Yo se que son 5 lineas ya siempre por lo que saco las lineas y aplicamos lo mismo para sacar cual base es y la palabra.
 
print(f.recvline())
print(f.recvline())
print(f.recvline())
base2 = f.recvline()
baseX = base2.rstrip(b':\n').split(b' ')
print(baseX[4])
word = f.recvline()
word2 = word.rstrip(b'\n')
print(word2)
```
## Output3
```
root@W1C3B0X:~/Descargas/ctfs/certunlp/ScriptingWriteups# python3 tutorial.py 
[+] Opening connection to encoderbase.ctf.cert.unlp.edu.ar on port 5001: Done
b'b64encode'
b'tocar'
```
## ScriptCompleto
Por lo que ya es siempre aplicamos el mismo metodo y lo metemos en un "while true"
```
from pwn import *
import base64
f = remote('encoderbase.ctf.cert.unlp.edu.ar', 5001)
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
print(f.recvline())
base = f.recvline()
word = f.recvline()
baseX = base.rstrip(b':\n').split(b' ')
print(baseX)
wordclean = word.rstrip(b'\n')
print(wordclean)
#cipher = wordclean.decode('utf8')
#print(cipher)
#print(base64.b64encode(wordclean))
if baseX[4] == b'b64encode':
        f.sendline(base64.b64encode(wordclean))
if baseX[4] == b'b32encode':
        f.sendline(base64.b32encode(wordclean))
if baseX[4] == b'b16encode':
        f.sendline(base64.b16encode(wordclean))
if baseX[4] == b'b85encode':
        f.sendline(base64.b85encode(wordclean))
while True:
        print(f.recvline())
        print(f.recvline())
        print(f.recvline())
        base2 = f.recvline()
        baseX = base2.rstrip(b':\n').split(b' ')
        word = f.recvline()
        word2 = word.rstrip(b'\n')
        print(baseX)
        print(word2)
        if baseX[4] == b'b64encode':                                                                              
                f.sendline(base64.b64encode(word2))                                                               
        if baseX[4] == b'b32encode':                                                                              
                f.sendline(base64.b32encode(word2))                                                               
        if baseX[4] == b'b16encode':                                                                              
                f.sendline(base64.b16encode(word2))                                                               
        if baseX[4] == b'b85encode':                                                                              
                f.sendline(base64.b85encode(word2))                                                               
f.close() 
```

## Output
```
[b'Encode', b'next', b'string', b'with', b'b85encode']
b'menu'
b'Nice!\n'
b'Genius!! The flag isflag{reality_is_a_consensus}\n
```
Finalmente tras un minutito enviando y recibiendo peticiones del servidor nos tira la flag!!!

```
flag{reality_is_a_consensus}
```
