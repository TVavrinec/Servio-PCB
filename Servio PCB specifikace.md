# Servio PCB

Cílem je mít PCB na řízení stejnosměrných motorů. Deska je založena na procesoru, který
- přijímá příkazy po komunikačním rozhraní,
- ovládá H-můstek,
- umí měřit proud motorem,
- vyčítá potenciometrový signál,
- a má možnost pro připojení magnetického enkodéru.

## Realizace a přesné požadavky
- všechny konektory (pokud není řečeno jinak) jsou JST-XH
- postaveno na procesoru STM32H503CBT6
  - procesor nepotřebuje externí oscilátor
  - procesor má vyvedeno debugovací rozhraní SWD a 1 UART jako debugovací (může být na 1 konektoru, obsahuje i 3V3)
  - (chtěli bychom STM32H503CBU, ale není skladem a brzo nebude - ideálně ale používat pouze nožičky, které mají oba)
- H-můstek je DRV8251ADDAR
  - IN1/2 piny jsou napojeny na kanály 1 a 2 časovače
  - IPROPI pin je zapojen tak, aby interní ADC procesoru mohlo číst plný rozsah proudu H-můstku
  - VREF je zapojen na DAC procesoru
  - motor je připojen přes pájecí kontakty
  - v analogových cestách jsou vyvedeny test-pointy
- komunikační rozhraní je plně-duplexní UART
  - je vyvedeno dvakrát - tak aby se serva mohla řetězit za sebe
  - 4-pinový konektor obsahuje GND, vstupní napájení, TX, RX
- potenciometr
  - znovupoužíváme potenciometr ze serva
  - připojen přes pájecí plošky 
- magnetický enkodér
  - není na desce
  - předpokládáme SPI & MT6701
  - připojen na fyzickou periferii SPI
- quadraturní enkodér
  - přijpojen na vhodný časovač
  - jen konektor, GND, A, B, VCC
- ideálně spojit konektory na mag-enc a quad-enc do jednoho konektoru, kterému se softwarově přepíná smysl (nebo má jeden pin na detekci)
- napájení
  - až 5-30V  
- signalizační LED
  - 1× červená
  - 1× zelená
  - 1× modrá
  - 1× žlutá
- formát
  - "co nejmenší, ale nehrotit" - protože
  - všechny montážní otvory jsou v rastru 5mm
  - skrze umístění komponent a konektorů, přepdokládej, že deska přijde namontovat do serva/na servo.

## Postup realizace
- prvně se nakreslí CubeMX přířazení pinů (a projde schválením)
- nakreslí se schéma (a projde schválením)
- nakreslí se PCB (a projde schválením)
- když cokoliv není jasné, tak jsme k dispozici pro dotazy.

## Bonus
Pokud si troufáš (i s velkým procesorem) vlézt se do LX-16A nebo https://www.amazon.com/DSSERVO-DS3225MG-Waterproof-Digital-Airplane/dp/B09SB8RMTT, tak do toho. Veveho 5mm montážní otvory dáme na rámeček panelu.

## Feedback iter 1.0
 - zelena/zluta ledka vyuzivaji PWM - bylo by fajn je dat na timer
    - je ale potreba at to striktne _neni_ timer co se pouziva na hmustek nebo quad encoders
 - chceme mit moznost merit _napeti_ napajeciho vstupu do serva (VIN)
    - je to prakticka informace
    - chci schopnost na "voltage cutoff" - moc nizke vstupni napeti - servo vypina motor
 - na predchozi iteraci byl jeden timer sezrany jako "vnitrni" timer pro spravne poolovani ADC
    - je mozne ze H5 bude delat to co chci i bez toho
    - nicmene bych to prelozil na "je potreba at je aspon jeden timer co jde provazat s ADC volny"

## Zmeny pro V2:
 
 0) Zmena koncepce:
    - Promyslet jestli neni vyhodnejsi se vyprdnout na "desku v desce" a mit radsi samostatnou servio desku, ktera se nasadi na debugovaci desku skrz pogo piny.

 1) Tvar desky:
    - Ocekavame pouziti 3d tistenych ramecku v servech - vezme se servio, vytiskne se ramecek a strci do serva
    => trosku zmensit tezku, idealne o 2mm na sirku/delku

 2) Procesor stejny, ale prozkoumat jestli mouser nema mensi footprint (LCSC nema)

 3) Odstranit jeden "interface connector" 
    - Misto toho udelat "skupinu" trech 4PIN konektoru na debugovaci desce ktere jsou propojene jen mezi sebou
    - To co pak muzu udelat bud pouzit jen 4PIN na serviu, nebo ho napojit do 4PINU na debug desce a od toho vest dva ruzne

 4) Pridat externi CHIP na ulozeni konfigurace - EEPROM pamet - preference na >=2kB pameti a interakci skrz I2C/SPI

 5) Prepojit hlavni "interface" UART na piny ktere se mapuji na stm32 bootloader pro flashovani, viz .pdf
    - odkaz: https://www.st.com/resource/en/application_note/an2606-stm32-microcontroller-system-memory-boot-mode-stmicroelectronics.pdf
    - v .pdf se zminuje ze je potreba nastavit spravne jeeden pin at se prepne do bootloaderu - zpristupnit jako jumper?

 6) Zvetsit "onboard" DBG connector at je dostupnejsi
    - be creative
    - zaroven je treba zpristupnit reset PIN

 7) Tvar PCB pro chtenne servo byl jemne mimo, viz. https://github.com/TVavrinec/Servio-PCB/issues/10
    - Soustredil bych se spis na to zmensit deskur skrz krok 1) nez resit tohle

 8) Zpristupnit piny s PWM vstupem do H-Mustku na logic analyzer - je to docela casta operace

 9) Neni pristupny "index reset pin" na quadrature encoder - zpristupnit
    - Details here: https://github.com/TVavrinec/Servio-PCB/issues/8
    - Teoreticky staci at na quad.enc+spi nepouzivame PB5+PB4+PB3, ale PB5,PB4,PA2

 10) Dodelat zemne kvuli sonde na scope :) at je jednoduche debuggovani
    - Staci na debugovaci desce

 11) Chceme kompatibilitu s black magic probe, takze:
    - Footprinty: https://black-magic.org/knowledge/pinouts.html
    - Zachovat STDC14 konektor, ale ocekavame ze se bude osazovat i jen jako 10pin - ARM SWD connector
       - Klidne to jde udelat jako dva konektory - STDC14 a STDC10
    - Udelat vedle STDC14 extra sadu pinu na UART - RX, TX, GND - 2,54mm pin header - at miri ven z desky
    - Naroutovat i TRACESWO - je to uzitecna featura

 12) Pridat LEDky pro RX/TX piny debugovaciho UARTu na debugovaci desce

 13) Rozdelit konektor pro "position sensors" na externi desce na separatni pro: potak, quadrature, spi (pokud jsou identicke tak staci jeden) - furt JST-XH

 14) J7 - konektor na veci na cidla na desce je _maly_ na rucni pajeni - predelat:
    - "enc-type-detect" pin muzeme oddelat uplne
    - zkusit zvetsi na vetsi pitch?
    - NENI potreba at je to jeden konektor - klidne rozbit na vice konektoru/alternativni reseni
       
