## Obiettivo
Con questa guida si vuole fornire una piccola guida per costruire un dispositivo che possa seguire autonomamente un percorso prestabilito. Si fa notare che un possibile campo di impiego sia quello industriale: un dispositivo autogestito in grado di trasportare un carico da un luogo ad un altro senza l'impiego di personale.


## Dispositivo di controllo utilizzato
Per lo sviluppo del progetto si è deciso di utilizzare una scheda [B-L475E-IOT01A2](https://www.st.com/en/evaluation-tools/b-l475e-iot01a.html)

![B-L475E-IOT01A2_IMG](https://www.st.com/bin/ecommerce/api/image.PF264366.en.feature-description-include-personalized-no-cpn-medium.jpg)

Dotata di sensori utili al progetto come quello di prossimità.
Possiede inoltre anche un modulo WIFI che, in successive implementazione, potrebbe permettere il controllo a distanza del dispositivo.


## Definizione concetti
Prima di scrivere il codice che controllerà la nostra macchina è importante chiarire alcuni concetti:<br>
<br>
-La macchina seguirà una striscia nera posizionata al di sotto della stessa, la linea sarà rilevata mediante appositi [sensori](#sensori)<br>
-Una volta capita la posizione della macchina relativamente alla linea si procede con l'azionamento degli [attuatori](#attuatori)<br>
-Nel caso in cui un ostacolo si ponga davanti alla macchina, questa si arresterà<br>


## Sensori
Sono utilizzati tre [TCRT5000.](https://www.makerslab.it/sensore-infrarossi-tcrt5000/) Di seguito si riassume il loro funzionamento:<br>
![Sensore_TCRT5000](https://www.makerslab.it/wp-content/uploads/2020/12/TCRT5000-funzionamento-768x283.jpg)
<ul>
<li>Caso A: La luce emessa dal diodo LED infrarosso non incontra alcun ostacolo.</li>
<li>Caso B: La luce emessa dal diodo LED infrarosso viene assorbita da una superficie scura, il risultato ottenuto sarà simile al <em>CASO A</em></li>
<li>Caso C: La luce emessa dal diodo LED infrarosso viene riflessa dalla superficie dell'ostacolo, polarizzando il foto-transistor proporzionalmente alla luce riflessa</li>
</ul>

<em>Fonte: "https://www.makerslab.it/sensore-infrarossi-tcrt5000/" </em>

<br>
Notiamo che questo sensore è particolarmente utile nel nostro caso in quanto è in grado di fornire una risposta digitale che fornisca informazioni su un eventuale superficie scura posta davanti al LED infrarosso.
<br>
Se posizionassimo questi sensori nella parte anteriore-sinistra, anteriore-centrale e anteriore-destra della nostra macchina, potremmo individuare una linea nera e correggere la rotta a seconda dei sensori eccitati.

## Attuatori
Si occuperanno di muovere la macchina a seconda dei segnali inviati dall'unità di controllo.
Gli attuatori utilizzati sono due [DC Motor.](https://www.wiltronics.com.au/product/10137/yellow-motor-3-12vdc-2-flats-shaft/)

![DC_MOTOR](https://www.wiltronics.com.au/wp-content/uploads/images/make-and-create/gear-motor-dc-toy-car-wheel-arduino.jpg)

Mediante l'utilizzo di questi motori, collegati a delle ruote, è possibile muovere la macchina.
<br>Le due ruote sono posizionate ai lati destro e sinistro.
<br>Per quanto riguarda il moto rettilineo, è sufficiente che le due ruote si muovano alla stessà velocità.
<br>Per effettuare una rotazione (oraria/antioraria) sarà sufficiente muovere le ruote in direzioni opposte.
<br>Poniamo che la linea nera ecciti il sensore sinistro della macchina, sarà sufficiente far ruotare questa in senso antiorario per ritrovare di nuovo la linea nera sotto il centro e proseguire dritto.
<br>A gestire questi attuatori viene utilizzato un [L298N.](https://www.lombardoandrea.com/l298n-motore-dc-arduino/)

![L298N](https://m.media-amazon.com/images/I/612gWwZKexL._AC_SL1000_.jpg)

Ovvero un H-Bridege che permette di alimentare due motori DC. La scheda va alimentata con una tensione dai 2V ai 10V. Per il nostro utilizzo si consiglia di utilizzare i 3.3V in modo tale da non rendere eccessiva la velocità di movimento e garantire una maggiore precisione e stabilità.<br>
Osserviamo gli ingressi e le uscite della scheda:

### Ingressi
<ul>
<li>'+': Tensione che alimenti la scheda</li>
<li>'-': Massa o GND</li>
<li>'IN1': Most Significant Bit per il controllo dell'attuatore 1</li>
<li>'IN2': Least Significant Bit per il controllo dell'attuatore 1</li>
<li>'IN3': Most Significant Bit per il controllo dell'attuatore 2</li>
<li>'IN4': Least Significant Bit per il controllo dell'attuatore 2</li>
</ul>

### Uscite
<ul>
<li>'Motor-A'(Pin 1): Va collegato al DC motor 1</li>
<li>'Motor-A'(Pin 2): Va collegato all'altro capo del DC motor 1</li>
<li>'Motor-A'(Pin 1): Va collegato al DC motor 2</li>
<li>'Motor-A'(Pin 2): Va collegato all'altro capo del DC motor 2</li>
</ul>

## Creazione file

### Per utilizzare il sensore di prossimità ho utilizzato il template che si trova a questa PATH

`C:\Users\{VOSTRO_USERNAME}\STM32Cube\Repository\STM32Cube_FW_L4_V1.17.2\Projects\B-L475E-IOT01A\Applications\Proximity\`

In questo modo avrete il codice del sensore di prossimità.
Di questo file le funzioni che ci serviranno saranno:

	 `static void VL53L0X_PROXIMITY_Init(void)` 
	 `static uint16_t VL53L0X_PROXIMITY_GetDistance(void)` 


## Inizializzazione PIN

Creiamo un file temporaneo e apriamo la schermata dell'IDE dove è possibile selezionare il funzionamento dei pin.
Da questa schermata impostiamo i pin necessari secondo lo schema riportato sotto:

|Nome  | PIN | GPIO	|		Funzione  							|
|:-----:|:---:|:----:|:----------------------------------------:|
|PA3   |D4   |GPIO\_PIN\_3|		MSB Ruota 1						|
|PB4   |D5   |GPIO\_PIN\_4|		LSB Ruota 1						|
|PB1   |D6   |GPIO\_PIN\_1|		MSB Ruota 2						|
|PA4   |D7   |GPIO\_PIN\_4|		LSB Ruota 2						|
|PA15  |D9 	 |GPIO\_PIN\_15|	SENSORE DI CONTROLLO CENTRALE	|
|PA2   |D10  |GPIO\_PIN\_2|		SENSORE DI CONTROLLO SX			|
|PA7   |D11  |GPIO\_PIN\_7|		SENSORE DI CONTROLLO DX			|

Una volta impostati i pin bisognerà generare il file <em>main.c</em> tramite il compilatore.
Una volta generato il file sarà necessario copiare la funzione

`static void MX_GPIO_Init(void)`

nel file del sensore di prossimità, così da inizializzare correttamente i PIN.

