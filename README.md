## Obiettivo
Con questa guida si vuole fornire una piccola guida per costruire un dispositivo che possa seguire autonomamente un percorso prestabilito. Si fa notare che un possibile campo di impiego sia quello industriale: un dispositivo autogestito in grado di trasportare un carico da un luogo ad un altro senza l'impiego di personale.


## Dispositivo di controllo utilizzato
Per lo sviluppo del progetto si è deciso di utilizzare una scheda [B-L475E-IOT01A2](https://www.st.com/en/evaluation-tools/b-l475e-iot01a.html)

![B-L475E-IOT01A2_IMG](https://www.st.com/bin/ecommerce/api/image.PF264366.en.feature-description-include-personalized-no-cpn-medium.jpg)

Dotata di sensori utili al progetto come quello di prossimità.
Possiede inoltre anche un modulo WIFI che, in successive implementazioni, potrebbe permettere il controllo a distanza del dispositivo.


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
<br>Per gestire questi attuatori viene utilizzato un [L298N.](https://www.lombardoandrea.com/l298n-motore-dc-arduino/)

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

Nel main è importante non cancellare la chiamata di funzione 

`VL53L0X_PROXIMITY_Init()`

Che inizializza la comunicazione con il sensore di umidità e ne effettua la taratura


## Creazione funzioni

Per rendere il codice più leggibile e facile da sviluppare è necessario creare alcune funzioni che ci permettano di interagire con la macchina

```
void spegni_motori();
void vai_avanti();
void vai_indietro();
void frena();
int linea_nera_centrale();
int linea_nera_sx();
int linea_nera_dx();
void gira_orario();
void gira_antiorario();
```

Di seguito verrà spiegato il funzionamento e l'implementazione delle funzioni.

#### Spegnimento motori
```
void spegni_motori(){
	HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_RESET);
	HAL_GPIO_WritePin(GPIOB, GPIO_PIN_4, GPIO_PIN_RESET);

	HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET);
	HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_RESET);
}
```

Come suggerito dal nome, questa funzione ha lo scopo di spegnere i motori: nel caso in cui questi fossero in uno stato precedente di movimento, il movimento si protrarrà per inerzia fino ad esaurirsi.
Per ottenere questa modalità di funzionamento sarà sufficiente impostare i bit di controllo di entrambe le ruote a 0.

### Procedere dritto

```
void vai_avanti(){
	  //IMPOSTO I BIT DELLA PRIMA RUOTA
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_SET);	//MSB_1
	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_4, GPIO_PIN_RESET); //LSB_1

	  //IMPOSTO I BIT DELLA SECONDA RUOTA
	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);	//MSB_2
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_RESET); //LSB_2
}
```
Tramite questa funzione proseguono nella stessa direzione alla medesima velocità, il che si tradurrà, approssimativamente, in un moto rettilineo.
Questo avviene (almeno in questa configurazione) se entrambi gli MSB delle ruote sono impostati a 1 e gli LSB sono a 0.

### Procedere a ritroso

```
void vai_indietro(){
	  //CONFIGURAZIONE PRIMA RUOTA
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_RESET); //MSB_1
	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_4, GPIO_PIN_SET);   //LSB_1

	  //CONFIGURAZIONE SECONDA RUOTA
	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET);	//MSB_2
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_SET);	//LSB_2
}
```
Tramite questa funzione proseguono nella stessa direzione alla medesima velocità ruotando all'indietro.
Questo avviene (almeno in questa configurazione) se entrambi gli MSB delle ruote sono impostati a 0 e gli LSB sono a 1.


### Frenatura 

```
void frena(){
	  //CONFIGURAZIONE CON TUTTI I BIT A 1
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_SET);
	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_4, GPIO_PIN_SET);

	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_SET);
}
```

Imprime una forza bloccante alle ruote che si opporrà all'inerzia di movimento delle ruote.
Si ottiene impostando tutti i bit di entrambe le ruote a 1.

### Rileva linea al centro

```
int linea_nera_centrale(){
	//RITORNO 1 SE IL SENSORE CENTRALE HA TROVATO LA LINEA NERA
	return  (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_15) == GPIO_PIN_SET ) ? 1:0;
}
```

La funzione ritorna 1 (int) se sotto la macchina, al suo centro, vi è una linea nera.

### Rileva linea a destra

```
int linea_nera_dx(){
	//RITORNO 1 SE IL SENSORE DESTRO HA TROVATO LA LINEA NERA
	return  (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_7) == GPIO_PIN_SET ) ? 1:0;
}
```

La funzione ritorna 1 (int) se sotto la sezione destra della macchina, vi è una linea nera.

### Rileva linea a sinistra

```
int linea_nera_sx(){
	//RITORNO 1 SE IL SENSORE SINISTRO HA TROVATO LA LINEA NERA
	return  (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_SET ) ? 1:0;
}

```

La funzione ritorna 1 (int) se sotto la sezione sinistra della macchina, vi è una linea nera.


### Rotazione oraria

```
void gira_orario(){

	//La ruota dx va in antiorario
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_RESET);
	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_4, GPIO_PIN_SET);

	//La ruota sx va in orario
	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_RESET);


	  //Finché la macchina non rileva una linea al centro
	  //continua a girare
	  while(!linea_nera_centrale() && linea_nera_sx())
		  HAL_Delay(10);
	  //Trovata la linea mi fermo e spengo i motori
	  frena();
	  spegni_motori();

}
```

La funzione viene invocata nel caso in cui la linea nera dovesse essere rilevata dal sensore destro. In questo caso, come anticipato nella sezione [attuatori,](#attuatori) sarà sufficiente far ruotare l'auto in senso orario per riportare la linea al centro.
Per ottenre un movimento orario bisognerà far girare la ruota interna (destra) in senso antiorario e quella esterna (sinistra) in senso orario.
Una volta riportata la linea al centro, fermo i motori e li spengo.

### Rotazione antioraria

```
void gira_antiorario(){

	//La ruota dx va in orario
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_3, GPIO_PIN_SET);
	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_4, GPIO_PIN_RESET);

	//La ruota sx va in antiorario
	  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET);
	  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_4, GPIO_PIN_SET);

	  //Finché la macchina non rileva una linea al centro
	  //continua a girare
	  while(!linea_nera_centrale() && linea_nera_dx())
	  		  HAL_Delay(10);
	  //Trovata la linea mi fermo e spengo i motori
	  frena();
	  spegni_motori();

}

```

Il caso è molto simile a quello precedentemente analizzato ma con le direzioni invertite: in questo caso sarà la ruota sinistra a girare in senso antiorario, quella destra girerà invece in senso orario.


### Ai fini di riassumere la gestione delle ruote viene riportata di seguito una tabella

|    MSB    |	LSB   |	   OUTPUT    |
|:-----------:|:---------:|:--------------:|
|	  0     |    0    | Motore spento  |
|	  0     |    1    | Ruota antiorario    |
|	  1     |    0    | Ruota orario    |
|	  1     |    1    |    Frena     |


## Funzione main()

All'interno del main sarà necessario innanzitutto definire una variabile che contenga la distanza dell'ostacolo più vicino

```
  uint16_t distanza;
```

Dopodiché bisognerà spostarsi all'interno del ```while(true)``` e inserire il codice che gestisca la macchina durante il percorso.

```
while (1)
  {
	 //Se la distanza non è sicura mi fermo
	 distanza = VL53L0X_PROXIMITY_GetDistance();
	 if( distanza < MAX_DISTANCE ){
		spegni_motori();
		printf("DISTANCE is = %d mm \n\r", distanza);
		HAL_Delay(100);
		continue;
	}

	//Se il sensore trova linea nera, vado avanti
	if( linea_nera_centrale() ){
		vai_avanti();
	}else{
		//Se non trovo la linea nera centrale allora controllo ai lati
		if( linea_nera_sx() ){
			//Mi riposiziono per eccitare il sensore centrale girando in senso orario
			gira_orario(&linea_centrale, &linea_sx);
		}else if( linea_nera_dx() ){
			//Mi riposiziono per eccitare il sensore centrale girando in senso antiorario
			gira_antiorario(&linea_centrale, &linea_dx);
		//Se non c'è alcuna linea mi fermo
		}else{
			frena();
			spegni_motori();
		}
	}



	HAL_Delay(10);
  }

```

La prima parte che si analizza è la distanza da un ostacolo: se questa dovesse essere inferiore a un valore definito da una costante ```#MAX_DISTANCE``` allora l'esecuzione dell'iterazione viene subito annullata mediante l'utilizzo di un ```continue``` al veicolo di muoversi.

```
//Se la distanza non è sicura mi fermo
 distanza = VL53L0X_PROXIMITY_GetDistance();
 if( distanza < MAX_DISTANCE ){
	spegni_motori();
	printf("DISTANCE is = %d mm \n\r", distanza);
	HAL_Delay(100);
	continue;
}
```


Procedendo, nel caso in cui la distanza sia sicura, si controlla che la linea sia al centro, in caso positivo, si prosegue dritto

```
if( linea_nera_centrale() ){
	vai_avanti();
}
```

Nel caso in cui la linea nera non si trovi al centro, si prosegue controllando che questa sia a sinistra, in caso affermativo bisognerà ruotare la macchina tramite la funzione ```gira_orario()```

```
if( linea_nera_sx() ){
	//Mi riposiziono per eccitare il sensore centrale girando in senso orario
	gira_orario(&linea_centrale, &linea_sx);
}
```

Se la linea non dovesse trovarsi a sinistra, allora si procederà controllando a destra, nel caso l'ipotesi sia verificata il procedimento è il medesimo: 

```
else if( linea_nera_dx() ){
	//Mi riposiziono per eccitare il sensore centrale girando in senso antiorario
	gira_antiorario(&linea_centrale, &linea_dx);
}

```

Se non vi è alcuna linea, la macchina si arresta

```
else{
	frena();
	spegni_motori();
}

```
