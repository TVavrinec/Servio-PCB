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
