# MathGenius
## Author: [m0riart3](https://twitter.com/Carlosflethesmo)
## Descripción
![Descripcion]()

## Análisis del cifrado

Al conectarnos con el comando dado vemos en la terminal que nos sale un banner y un mensaje cifrado en forma numérica,este cifrado corresponde aparentemente al output de la función que viene escrita en el archivo que el enunciado nos proporciona.
Este es el mensaje que recibimos tras el comando

```
    ____      __
   /  _/___  / /__________
   / // __ \/ __/ ___/ __ \
 _/ // / / / /_/ /  / /_/ /
/___/_/ /_/\__/_/   \____/
   _______ __                                         _     __          __
  / ____(_) /_  ___  _____________  ____ ___  _______(_)___/ /___ _____/ /
 / /   / / __ \/ _ \/ ___/ ___/ _ \/ __ `/ / / / ___/ / __  / __ `/ __  /
/ /___/ / /_/ /  __/ /  (__  )  __/ /_/ / /_/ / /  / / /_/ / /_/ / /_/ /
\____/_/_.___/\___/_/  /____/\___/\__, /\__,_/_/  /_/\__,_/\__,_/\__,_/
                                 /____/                                   
[120651644435136,107368775971668,95174353931179,172699096145932,278597911474944,60875981196690,
31102518904704,424516283881640,147697838125635,231489555056996,161030438524592,613295596332,
47227307514234,567189770919673,216743473656939,339375427543240,602978122441184,179806744085448,
321429553934220,432661416544270,60478828630860,8605679354030,345852497435955,401335208303232,
348408310879848,125956026425520,491108717977248,181444170055568,94268944594760,337307532797240,
310456796968098,591641457703578,241884451841382,348000764725264,441777741821612,20568045400128,
156555650316450,14684483740839,187867046503125,50482514599150]
```
Y este el contenido del archivo que nos da el enunciado

```python
def challenge():
    flag = open("flag.txt").read()
    arr = []
    for char in flag:
        r= random.randint(4, 5345346346435)
        r = r - 17+17+17
        arr.append(str(r * ord(char)))
    send_to_client("[{}]\n".format(",".join(arr)).encode())
```
Viendo un poco más en profundidad lo que hace la función del archivo podemos ver que usa un valor aleatorio de nombre r, en el rango de valores desde el 4 hasta el 5345346346435, y que lo  multiplica por el valor numérico del carácter de la flag, por lo que r no es constante en todo el mensaje, por tanto, la idea inicial de averiguar r queda descartada. Hacemos un script en python 3 para deshacer el cifrado partiendo de la ecuación de despejar r, implementando un for que vaya desde el valor 30 hasta el 125 y dividiendo el valor del carácter cifrado por el de la variable del bucle, hasta encontrar un valor que haga que la división sea exacta, dándonos como resultado varios posibles caracteres

Este es el script hecho en python 3
```python
import math
num = 344299729372152                                                                                             
                                                                                                                  
for x in range(33,125):                                                                                           
        rest = num / x                                                                                            
        parte_dec,parte_ent = math.modf(rest)                                                                     
        if parte_dec == 0:                                                                                        
                print("Valor encontrado:")                                                                        
                print("el valor de r es ",rest)                                                                   
                print("El valor de la letra era ",chr(x))
```      
donde habrá que cambiar el valor de num por el del carácter cifrado para cada ejecución del script. Como el  resultado son varias posibles letras, habrá que repetir el proceso varias veces
```
m0riart3@kali:~/Desktop/crypto$ python3 script.py 
Valor encontrado:
el valor de r es  9060519194004.0
El valor de la letra era  &
Valor encontrado:
el valor de r es  6040346129336.0
El valor de la letra era  9
Valor encontrado:
el valor de r es  4530259597002.0
El valor de la letra era  L
Valor encontrado:
el valor de r es  3020173064668.0
El valor de la letra era  r
```
Hasta poder completar la flag 
```
m0riart3@kali:~/Desktop/crypto$ cat flag.txt 
flag{n0_impl3m3nt3s_tU_prop14_cr1pt0!!}
```
