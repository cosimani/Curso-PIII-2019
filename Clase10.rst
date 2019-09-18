.. -*- coding: utf-8 -*-

.. _rcs_subversion:

Clase 10 - PIII 2018
====================
(Fecha: 7 de noviembre)



Ejercicio 6:
============

- Generar una señal de 5Hz pensado para aplicar un efecto trémolo (variación periódica del volumen) a una señal de audio que está siendo muestreada a 1kHz.


Ejercicio 7:
============

- Aplicar el trémolo de 5Hz a la señal generada de 100Hz.

.. figure:: images/clase07/captura_tremolo.png

Ejercicio 8:
============

- Muestrear una señal de audio y aplicar el trémolo anterior.

Ejercicio 9:
============

Muestrear una señal analógica (100 Hz, offset de 2 V y 4 Vpp), aplicarle un trémolo y mostrar la resultante luego de un DAC R-2R.

**Especificaciones:**

- Entrada por AN2
- Utilizar Vref+ y Vref- con valores óptimos
- Entíendase el trémolo como una señal modulante con la que se logra un índice de modulación particular (ver Variaciones por alumno)
- Frecuencia de muestreo: 1 kHz
- ADC de 12 bits
- Definir una frecuencia del trémolo particular (ver Variaciones por alumno)

**Entregar:**

- Proyecto en mikroC
- Captura de pantalla del osciloscopio con la señal resultante
- Video de algunos segundos mostrando el conexionado y la visualización en el osciloscopio

**Variaciones por alumno:**

:Agustina:
    Frecuencia del trémolo: 2 Hz
	
    Índice de modulación del 20%

:Ignacio:
    Frecuencia del trémolo: 4 Hz
	
    Índice de modulación del 30%

:Julián:
    Frecuencia del trémolo: 6 Hz
	
    Índice de modulación del 40%

:Facundo:
    Frecuencia del trémolo: 8 Hz
	
    Índice de modulación del 50%

**Ejemplo que sirve de guía:** 

- `Solución de un ejercicio parecido en Proteus <https://github.com/cosimani/Curso-PIII-2016/blob/master/resources/clase06/Ej1.rar?raw=true>`_

.. figure:: images/clase06/Ej1-Esquema.png

.. figure:: images/clase06/Ej1-Osciloscopio.png


**ADC automático para dsPIC30F4013**

.. figure:: images/clase08/adc_auto_1.png

.. figure:: images/clase08/adc_auto_2.png

.. figure:: images/clase08/adc_auto_3.png

**Ejemplo:** Realizar cálculo para muestrear la voz humana

.. figure:: images/clase08/adc_auto_ejer_1.png

.. figure:: images/clase08/adc_auto_ejer_2.png

Ejercicio 10:
============
- Programar esto y controlar con el EasydsPIC si la frecuencia de muestreo está bien.

Ejercicio 11:
============

- Adaptar el programa para el dsPIC33FJ32MC202 y controlarlo en Proteus.

**Código de ejemplo**

.. code-block:: c

	unsigned int contador = 0;

	void detectar_adc() org 0x002a  {
	    contador = contador + 1;
	    if (contador > 2000)  {  // Para que D1 cambie de estado cada 1 segundo
	        LATDbits.LATD1 = ~LATDbits.LATD1;
	        contador = 0;
	    }

	    IFS0bits.ADIF = 0;
	}

	void config_adc()  {
	    ADPCFG = 0xFFFD;  // Elegimos la entrada analógica

	    ADCON1bits.ADSIDL = 1;  // No trabaja en modo IDLE (modo bajo consumo - CPU off, Peripherals on)
	    ADCON1bits.FORM = 0b00;  // Formato de salida entero
	    ADCON1bits.SSRC = 0b111;  // Muestreo automatico
	    ADCON1bits.ASAM = 1;  // Comienza a muestrear luego de la conversion anterior

	    ADCON2bits.VCFG = 0b000;  // Referencia AVdd y AVss
	    ADCON2bits.SMPI = 0b0000;  // Lanza interrupcion luego de n muestras
	    // 0b0000 - 1 muestra / 0b0001 - 2 muestras / 0b0010 - 3 muestras

	    ADCON3bits.SAMC = 31;
	    ADCON3bits.ADCS = 55;

	    ADCHSbits.CH0SA = 0b0001;  // 0b0000 para AN0 / 0b0001 para AN1 / 0b0010 para AN2

	    ADCON1bits.ADON = 1;
	}

	void configuracionPuertos()  {
	    // Para LEDs de debug
	    TRISDbits.TRISD1 = 0;  // Debug IntADC
	}

	void main()  {
	    configuracionPuertos();

	    config_adc();

	    IEC0bits.ADIE = 1;

	    while(1)  {
	    }
	}

**Práctico sobre modulación en amplitud charlado en clase**

.. figure:: images/clase07/am1.png

.. figure:: images/clase07/am2.png


Programación de filtros
^^^^^^^^^^^^^^^^^^^^^^^	
	
**Función de transferencia**

- Relación entre la entrada y la salida al pasar por el proceso
- Se manipulan en términos de la frecuencia compleja y no del tiempo continuo para simplificar
- Las funciones de transferencia en tiempo discreto se hace en término de la variable compleja z
- Se recurre a la transformada Z que representa la frecuencia compleja para tiempo discreto

.. figure:: images/clase08/filtros_1.png

- Para el tratamiento real en término del tiempo discreto se realiza la transformada inversa de z
- La transformación inversa de la ecuación anterior queda en función del tiempo discreto queda:

.. figure:: images/clase08/filtros_2.png

- Esto es una convolución
- El número máximo que asume n es M
- M determina el orden de la función de transferencia

- FIR: Sistema sin memoria. Convolución con muestras pasadas y actuales.
- IIR: Sistema con memoria. Convolución con muestras pasadas y actuales, y también salidas pasadas y(n)

- FIR: Fácil implementación y diseño pero consumen más recursos
- IIR: Más matemática pero requieren campos de memoria más pequeños

**Convolución en C**

.. figure:: images/clase08/filtros_3.png

**El código puede ser:**

.. code-block:: c

	#define M 17
	float x[ M ];
	float h[ M ];

	float yn = 0;
	short k;
	
	for ( k = M - 1 ; k >= 1 ; k-- )
	    x[ k ] = x[ k-1 ];
		
	x[ 0 ] = x0;  // x0 es la muestra actual
	
	for ( k = 0 ; k < M ; k++ )
	    yn += h[ k ] * x[ k ];

**Función de transferencia: Filtro pasa bajos**

.. figure:: images/clase08/filtros_4.png

- Lo podemos calcular con el Excel

.. figure:: images/clase08/filtros_5.png

.. figure:: images/clase08/filtros_6.png

**Ejemplo Filtro FIR**

- Fs = 4000
- Fc = 150Hz

.. code-block:: c

	#define M 17
	float x[M];
	float h[M] = 
	    {0.037841336, 0.045332663, 0.052398494, 0.058815998, 0.064379527,
	    0.068908578, 0.072254832, 0.074307967, 0.075, 0.074307967, 0.072254832, 0.068908578,
	    0.064379527, 0.058815998, 0.052398494, 0.045332663, 0.037841336};

	float yn=0;

	unsigned int i;
	short k;
	float valorActual = 0;

	void  detectarIntADC()  org 0x002E  {
	    IFS0bits.AD1IF=0;

	    for (k=M-1 ; k>=1 ; k--)  {
	        x[k] = x[k-1];
	    }

	    //Se guarda la última muestra.
	    x[0] = ((float)ADC1BUF0-2048);

	    yn = 0;

	    for (k=0 ; k<M ; k++)  {
	        yn += h[k]*x[k];
	    }

	    valorActual = yn + 2048;

	    LATBbits.LATB2 =   ((unsigned int)valorActual & 0b0000100000000000) >> 11;
	    LATBbits.LATB3 =   ((unsigned int)valorActual & 0b0000010000000000) >> 10;
	    LATBbits.LATB4 =   ((unsigned int)valorActual & 0b0000001000000000) >> 9;
	    LATBbits.LATB5 =   ((unsigned int)valorActual & 0b0000000100000000) >> 8;
	    LATBbits.LATB6 =  ((unsigned int)valorActual &  0b0000000010000000) >> 7;
	    LATBbits.LATB7 =  ((unsigned int)valorActual &  0b0000000001000000) >> 6;
	    LATBbits.LATB8 =  ((unsigned int)valorActual &  0b0000000000100000) >> 5;
	    LATBbits.LATB9 =  ((unsigned int)valorActual &  0b0000000000010000) >> 4;
	    LATBbits.LATB10 = ((unsigned int)valorActual &  0b0000000000001000) >> 3;
	    LATBbits.LATB11 = ((unsigned int)valorActual &  0b0000000000000100) >> 2;
	    LATBbits.LATB12 = ((unsigned int)valorActual &  0b0000000000000010) >> 1;
	    LATBbits.LATB13 = ((unsigned int)valorActual &  0b0000000000000001) >> 0;
	}

	void detectarIntT2() org 0x0022  {

	    IFS0bits.T2IF=0;  //borra bandera de interrupcion de TIMER2

	    LATBbits.LATB15=~LATBbits.LATB15;

	    AD1CON1bits.SAMP=1; //pedimos muestras
	    asm nop;  //ciclo instruccion sin operacion
	    AD1CON1bits.SAMP=0;  //retener muestra e inicia conversion
	}

	void configADC()  {
	    AD1PCFGL=0b111011;  //elegimos AN2 como entrada para muestras
	    AD1CHS0 =0b0010; //usamos AN2 para recibir las muestras en el ADC
	    AD1CON1bits.SSRC=0b000; //muestreo manual
	    AD1CON1bits.ADON=0;  //apagamos ADC
	    AD1CON1bits.AD12B=1;  //12bits S&H ADC1
	    AD1CON2bits.VCFG=0b011;  //tension de referencia externa Vref+ Vref-
	    IEC0bits.AD1IE=1;  //habilitamos interrupcion del ADC
	}

	void configTIMER2()  {
	    T2CON=0x0000;   //registro de control de TIMER2 a cero
	    T2CONbits.TCKPS=0b00;// prescaler = 1
	    TMR2=0;  //desde donde va a arrancar la cuenta
	    PR2=1250;   //hasta donde cuenta segun calculo para disparo de TIMER2
	    IEC0bits.T2IE=1; //habilitamos interrupciones para TIMER2
	}

	void configPuertos()  {
	    TRISBbits.TRISB2 = 0;
	    TRISBbits.TRISB3 = 0;
	    TRISBbits.TRISB4 = 0;
	    TRISBbits.TRISB5 = 0;
	    TRISBbits.TRISB6 = 0;
	    TRISBbits.TRISB7 = 0;
	    TRISBbits.TRISB8 = 0;
	    TRISBbits.TRISB9 = 0;
	    TRISBbits.TRISB10 = 0;
	    TRISBbits.TRISB11 = 0;
	    TRISBbits.TRISB12 = 0;
	    TRISBbits.TRISB13 = 0;

	    TRISBbits.TRISB15=0;  // Debug T2
	}

	void main()  {
	    configPuertos();
	    configTIMER2();
	    configADC();

	    AD1CON1bits.ADON = 1;

	    T2CONbits.TON=1;

	    while(1)  {
	    }
	}

**Material extra de consulta sobre filtros**		

.. figure:: images/clase08/portada-material-consulta-filtros.png
	:target: images/clase08/material-consulta-filtros.pdf


Ejercicio 12:
============

- Programar esto y controlar en Proteus. 
- Analizar si la frecuencia de muestreo es la misma con el ADC encendido y apagado. Es decir, realizando el procesamiento de la señal o no.
- De ser necesario, definir una frecuencia de muestreo tal que no se vea afectado el procesamiento.
- Identificar cuál es la frecuencia máxima a la que se podría muestrear.


Ejercicio 13:
============

- Calcular esa frecuencia máxima para el ADC automático.


Primera entrega de prácticos finales
====================================


.. figure:: resources/clase07/espacio-vertical.png


**Identificador de tonos DTMF ( Agustina Alvarez - Carlos Ignacio )** 

( Clic sobre la siguiente imagen para abrir el informe en PDF )

.. figure:: resources/clase07/PrimeraEntrega-Agustina-Carlos.png
	:target: resources/clase07/PrimeraEntrega-Agustina-Carlos.pdf


.. figure:: resources/clase07/espacio-vertical.png


**Reconocimiento de voz ( Karraz Facundo - Gutierrez Julian )** 

( Clic sobre la siguiente imagen para abrir el informe en PDF )

.. figure:: resources/clase07/PrimeraEntrega-Julian-Facundo.png
	:target: resources/clase07/PrimeraEntrega-Julian-Facundo.pdf






**Problema con el Ejemplo del filtro paso bajos anterior**

- Utilizando el código que hace la sumatoria del producto de los valores de los vectores, se consume demasiado tiempo.
- Tanto tiempo que no se puede mantener la frecuencia de muestreo.
- Una opción es usar PLL.
- El siguiente código resuelve el caso con PLL x8, para el dsPIC30F4013 con cristal de 10MHz

.. code-block:: c

	#define M 17
	float x[M];
	float h[M] =  {
	    0.037841336, 0.045332663, 0.052398494, 0.058815998, 0.064379527, 
	    0.068908578, 0.072254832, 0.074307967, 0.075000000, 0.074307967, 
	    0.072254832, 0.068908578, 0.064379527, 0.058815998, 0.052398494, 
	    0.045332663, 0.037841336
	};

	float yn = 0;

	unsigned int i;
	short k;
	float valorActual = 0;

	void  detectarIntADC()  org 0x002A  {

	    IFS0bits.ADIF=0;

	    for (k=M-1 ; k>=1 ; k--)  {
	        x[k] = x[k-1];
	    }

	    //Se guarda la última muestra.
	    x[0] = ((float)ADCBUF0-2048);

	    yn = 0;

	    for (k=0 ; k<M ; k++)  {
	        yn += h[k]*x[k];
	    }

	    valorActual = yn + 2048;


	    LATCbits.LATC14 = ( (unsigned int) valorActual & 0b0000100000000000) >> 11;
	    LATBbits.LATB2 =  ( (unsigned int) valorActual & 0b0000010000000000) >> 10;
	    LATBbits.LATB3 =  ( (unsigned int) valorActual & 0b0000001000000000) >> 9;
	    LATBbits.LATB4 =  ( (unsigned int) valorActual & 0b0000000100000000) >> 8;
	    LATBbits.LATB5 =  ( (unsigned int) valorActual & 0b0000000010000000) >> 7;
	    LATBbits.LATB6 =  ( (unsigned int) valorActual & 0b0000000001000000) >> 6;
	    LATBbits.LATB8 =  ( (unsigned int) valorActual & 0b0000000000100000) >> 5;
	    LATBbits.LATB9 =  ( (unsigned int) valorActual & 0b0000000000010000) >> 4;
	    LATBbits.LATB10 = ( (unsigned int) valorActual & 0b0000000000001000) >> 3;
	    LATBbits.LATB11 = ( (unsigned int) valorActual & 0b0000000000000100) >> 2;
	    LATBbits.LATB12 = ( (unsigned int) valorActual & 0b0000000000000010) >> 1;
	    LATCbits.LATC13 = ( (unsigned int) valorActual & 0b0000000000000001) >> 0;

	    LATDbits.LATD1 = ~LATDbits.LATD1;

	}

	void detectarIntT2() org 0x0020  {

	    IFS0bits.T2IF = 0;  // borra bandera de interrupcion de T2

	    ADCON1bits.SAMP=1; // pedimos muestras
	    asm nop;  //ciclo de instruccion sin operacion
	    ADCON1bits.SAMP=0;  // retener muestra e iniciar conversion
	}

	void configADC()  {
	    ADPCFG = 0b111110;  // elegimos AN0 como entrada para muestras
	    ADCHS = 0b0000;  // usamos AN0 para recibir las muestras en el ADC
	    ADCON1bits.SSRC = 0b000; // muestreo manual
	    ADCON2bits.VCFG = 0b000;  //tension de referencia externa Vref+ Vref-

	    IEC0bits.ADIE = 1;  //habilitamos interrupcion del ADC
	}

	void configT2()  {
	    T2CONbits.TCKPS = 0b00;  // prescaler = 1
	    PR2=5000;   // PLLx8 - cristal 10MHz - Tcy=50ns - Entonces fs=4kHz

	    IEC0bits.T2IE=1; // habilitamos interrupciones para T2
	}

	void configPuertos()  {
	    TRISCbits.TRISC14 = 0;  // Bit mas significativo de la senal generada
	    TRISBbits.TRISB2 = 0;
	    TRISBbits.TRISB3 = 0;
	    TRISBbits.TRISB4 = 0;
	    TRISBbits.TRISB5 = 0;
	    TRISBbits.TRISB6 = 0;
	    TRISBbits.TRISB8 = 0;
	    TRISBbits.TRISB9 = 0;
	    TRISBbits.TRISB10 = 0;
	    TRISBbits.TRISB11 = 0;
	    TRISBbits.TRISB12 = 0;
	    TRISCbits.TRISC13 = 0;  // Bit menos significativo de la senal generada

	    TRISDbits.TRISD1=0;  // Debug
	}

	void main()  {
	    configPuertos();
	    configT2();
	    configADC();

	    ADCON1bits.ADON = 1;

	    T2CONbits.TON=1;

	    while(1)  { 
	    }
	}


**Ejercicio** 

- Intentar utilizar el código que genera el Filter Designer Tool del mikroC. 


**Probando filtros en Proteus y en Placa**

- Video sobre cómo utilizar el generador de señal (https://www.youtube.com/watch?v=qCRcNYbqBxs)

**Ejemplo para dsPIC33FJ32MC202 para Proteus**

- `Proyecto en Proteus 8.1 <https://github.com/cosimani/Curso-PIII-2016/blob/master/resources/clase08/EjemploClase8.rar?raw=true>`_

.. code-block:: c

	// Device setup:
	//     Device name: P33FJ32MC202
	//     Device clock: 010.000000 MHz
	//     Sampling Frequency: 1000 Hz
	// Filter setup:
	//     Filter kind: FIR
	//     Filter type: Lowpass filter
	//     Filter order: 30
	//     Filter window: Hamming
	//     Filter borders:
	//       Wpass:30 Hz
	const unsigned BUFFFER_SIZE  = 32;
	const unsigned FILTER_ORDER  = 30;

	const unsigned COEFF_B[FILTER_ORDER+1] = {
	    0x0022, 0x0041, 0x007B, 0x00E1, 0x0182, 0x0267,
	    0x0393, 0x0500, 0x06A1, 0x0862, 0x0A27, 0x0BD3,
	    0x0D47, 0x0E67, 0x0F1E, 0x0F5C, 0x0F1E, 0x0E67,
	    0x0D47, 0x0BD3, 0x0A27, 0x0862, 0x06A1, 0x0500,
	    0x0393, 0x0267, 0x0182, 0x00E1, 0x007B, 0x0041,
	    0x0022};

	unsigned inext;                       // Input buffer index
	ydata unsigned input[BUFFFER_SIZE];   // Input buffer, must be in Y data space

	void config_adc()  {
	    ADPCFG = 0xFFF7; // La entrada analogica es el AN3
	    // Con cero se indica entrada analogica y con 1 sigue siendo entrada digital.

	    AD1CON1bits.ADON = 0;  // ADC apagado por ahora
	    AD1CON1bits.AD12B = 0;  // ADC de 10 bits

	    // Tomar muestras en forma manual, porque lo vamos a controlar con el Timer 2
	    AD1CON1bits.SSRC = 0b000;

	    // Adquiere muestra cuando el SAMP se pone en 1. SAMP lo controlamos desde el Timer 2
	    AD1CON1bits.ASAM = 0;

	    AD1CON2bits.VCFG = 0b000;  // Referencia desde la fuente de alimentación
	    AD1CON2bits.SMPI = 0b0000;  // Lanza interrupción luego de tomar n muestras.
	    // Con SMPI=0b0000 -> 1 muestra ; Con SMPI=0b0001 -> 2 muestras ; etc.

	    // AD1CON3 no se usa ya que usamos muestreo manual

	    // Muestreo la entrada analogica AN3
	    AD1CHS0 = 0b00011;
	}

	void config_timer2()  {
	    // Prescaler 1:1   -> TCKPS = 0b00 -> Incrementa 1 en un ciclo de instruccion
	    // Prescaler 1:8   -> TCKPS = 0b01 -> Incrementa 1 en 8 ciclos de instruccion
	    // Prescaler 1:64  -> TCKPS = 0b10 -> Incrementa 1 en 64 ciclos de instruccion
	    // Prescaler 1:256 -> TCKPS = 0b11 -> Incrementa 1 en 256 ciclos de instruccion
	    T2CONbits.TCKPS = 0b00;

	    // Empieza cuenta en 0
	    TMR2=0;

	    // Cuenta hasta 5000 ciclos y dispara interrupcion
	    PR2=5000;  // 5000 * 200 nseg = 1 mseg   ->  1 / 1mseg = 1000Hz
	}

	void config_ports()  {
	    TRISBbits.TRISB1 = 1;  // Entrada para muestrear = AN3

	    TRISBbits.TRISB2 = 0;
	    TRISBbits.TRISB3 = 0;
	    TRISBbits.TRISB4 = 0;
	    TRISBbits.TRISB5 = 0;
	    TRISBbits.TRISB6 = 0;
	    TRISBbits.TRISB7 = 0;
	    TRISBbits.TRISB8 = 0;
	    TRISBbits.TRISB9 = 0;
	    TRISBbits.TRISB10 = 0;
	    TRISBbits.TRISB11 = 0;

	    TRISBbits.TRISB0 = 1;  // Para control del filtro

	    TRISBbits.TRISB13 = 0;  // Debug ADC
	    TRISBbits.TRISB14 = 0;  // Debug T2
	}

	void detect_timer2() org 0x0022  {
	    IFS0bits.T2IF=0;  // Borramos la bandera de interrupción Timer 2

	    LATBbits.LATB14 = !LATBbits.LATB14;  // Para debug de la interrupcion Timer 2

	    AD1CON1bits.DONE = 0;  // Antes de pedir una muestra ponemos en cero
	    AD1CON1bits.SAMP = 1;  // Pedimos una muestra

	    asm nop;  // Tiempo que debemos esperar para que tome una muestra

	    AD1CON1bits.SAMP = 0;  // Pedimos que retenga la muestra
	}

	void detect_adc() org 0x002e  {
	    unsigned CurrentValue;

	    IFS0bits.AD1IF = 0; // Borramos el flag de interrupciones del ADC
	    LATBbits.LATB13 = !LATBbits.LATB13;  // Para debug de la interrupcion ADC

	    if(PORTBbits.RB0 == 1)  {
	        input[inext] = ADCBUF0;                 // Fetch sample

	        CurrentValue = FIR_Radix(FILTER_ORDER+1,  // Filter order
		                             COEFF_B,         // b coefficients of the filter
		                             BUFFFER_SIZE,    // Input buffer length
		                             input,           // Input buffer
		                             inext);          // Current sample

	        inext = (inext+1) & (BUFFFER_SIZE-1);   // inext = (inext + 1) mod BUFFFER_SIZE;

	        LATBbits.LATB11 =  ((unsigned int)CurrentValue & 0b0000001000000000) >> 9;
	        LATBbits.LATB10 =  ((unsigned int)CurrentValue & 0b0000000100000000) >> 8;
	        LATBbits.LATB9 =  ((unsigned int)CurrentValue & 0b0000000010000000) >> 7;
	        LATBbits.LATB8 =  ((unsigned int)CurrentValue & 0b0000000001000000) >> 6;
	        LATBbits.LATB7 =  ((unsigned int)CurrentValue & 0b0000000000100000) >> 5;
	        LATBbits.LATB6 =  ((unsigned int)CurrentValue & 0b0000000000010000) >> 4;
	        LATBbits.LATB5 = ((unsigned int)CurrentValue & 0b0000000000001000) >> 3;
	        LATBbits.LATB4 = ((unsigned int)CurrentValue & 0b0000000000000100) >> 2;
	        LATBbits.LATB3 = ((unsigned int)CurrentValue & 0b0000000000000010) >> 1;
	        LATBbits.LATB2 = ((unsigned int)CurrentValue & 0b0000000000000001) >> 0;
	    }
	    else  {
	        LATBbits.LATB11  = ADCBUF0.B9;
	        LATBbits.LATB10  = ADCBUF0.B8;
	        LATBbits.LATB9  = ADCBUF0.B7;
	        LATBbits.LATB8  = ADCBUF0.B6;
	        LATBbits.LATB7  = ADCBUF0.B5;
	        LATBbits.LATB6  = ADCBUF0.B4;
	        LATBbits.LATB5 = ADCBUF0.B3;
	        LATBbits.LATB4 = ADCBUF0.B2;
	        LATBbits.LATB3 = ADCBUF0.B1;
	        LATBbits.LATB2 = ADCBUF0.B0;
	    }
	}

	int main()  {
	    config_ports();
	    config_timer2();
	    config_adc();

	    // Habilitamos interrupción del ADC y lo encendemos
	    IEC0bits.AD1IE = 1;
	    AD1CON1bits.ADON = 1;

	    // Habilita interrupción del Timer 2 y lo iniciamos para que comience a contar
	    IEC0bits.T2IE=1;
	    T2CONbits.TON=1;

	    while(1)  {  }

	    return 0;
	}

**Ejemplo para dsPIC30F4013 para Placa**

.. code-block:: c

	// Device setup:
	//     Device name: P30F4013
	//     Device clock: 010.000000 MHz
	//     Dev. board: EasydsPic4A
	//     Sampling Frequency: 4000 Hz
	// Filter setup:
	//     Filter kind: FIR
	//     Filter type: Lowpass filter
	//     Filter order: 30
	//     Filter window: Hamming
	//     Filter borders:
	//       Wpass:150 Hz
	const unsigned BUFFFER_SIZE  = 32;
	const unsigned FILTER_ORDER  = 30;

	const unsigned COEFF_B[FILTER_ORDER+1] = {
	    0xFFD5, 0xFFEB, 0x000F, 0x005A, 0x00E6, 0x01C9,
	    0x0312, 0x04C4, 0x06D3, 0x0926, 0x0B98, 0x0DF9,
	    0x1017, 0x11C3, 0x12D5, 0x1333, 0x12D5, 0x11C3,
	    0x1017, 0x0DF9, 0x0B98, 0x0926, 0x06D3, 0x04C4,
	    0x0312, 0x01C9, 0x00E6, 0x005A, 0x000F, 0xFFEB,
	    0xFFD5
	};

	unsigned inext;                       // Input buffer index
	ydata unsigned input[BUFFFER_SIZE];   // Input buffer, must be in Y data space

	void  detectarIntADC()  org 0x002a  {
	    unsigned CurrentValue;

	    IFS0bits.ADIF = 0; // Borramos el flag de interrupciones del ADC
	    LATFbits.LATF1 = !LATFbits.LATF1;  // Para debug de la interrupcion ADC

	    if(PORTFbits.RF4 == 0)  {
	        LATFbits.LATF5 = 1;  // Filtro no aplicado

	        input[inext] = ADCBUF0;                  // Fetch sample

	        CurrentValue = FIR_Radix(FILTER_ORDER+1, // Filter order
	                                 COEFF_B,        // b coefficients of the filter
	                                 BUFFFER_SIZE,   // Input buffer length
	                                 input,          // Input buffer
	                                 inext);         // Current sample

	        inext = (inext+1) & (BUFFFER_SIZE-1);    // inext = (inext + 1) mod BUFFFER_SIZE;

	        LATCbits.LATC14 = ((unsigned int)CurrentValue & 0b0000100000000000) >> 11;
	        LATBbits.LATB2 =  ((unsigned int)CurrentValue & 0b0000010000000000) >> 10;
	        LATBbits.LATB3 =  ((unsigned int)CurrentValue & 0b0000001000000000) >> 9;
	        LATBbits.LATB4 =  ((unsigned int)CurrentValue & 0b0000000100000000) >> 8;
	        LATBbits.LATB5 =  ((unsigned int)CurrentValue & 0b0000000010000000) >> 7;
	        LATBbits.LATB6 =  ((unsigned int)CurrentValue & 0b0000000001000000) >> 6;
	        LATBbits.LATB8 =  ((unsigned int)CurrentValue & 0b0000000000100000) >> 5;
	        LATBbits.LATB9 =  ((unsigned int)CurrentValue & 0b0000000000010000) >> 4;
	        LATBbits.LATB10 = ((unsigned int)CurrentValue & 0b0000000000001000) >> 3;
	        LATBbits.LATB11 = ((unsigned int)CurrentValue & 0b0000000000000100) >> 2;
	        LATBbits.LATB12 = ((unsigned int)CurrentValue & 0b0000000000000010) >> 1;
	        LATCbits.LATC13 = ((unsigned int)CurrentValue & 0b0000000000000001) >> 0;

	    }
	    else  {
	        LATFbits.LATF5 = 0;  // Filtro no aplicado

	        LATCbits.LATC14 = ADCBUF0.B11;
	        LATBbits.LATB2 = ADCBUF0.B10;
	        LATBbits.LATB3 = ADCBUF0.B9;
	        LATBbits.LATB4 = ADCBUF0.B8;
	        LATBbits.LATB5 = ADCBUF0.B7;
	        LATBbits.LATB6 = ADCBUF0.B6;
	        LATBbits.LATB8 = ADCBUF0.B5;
	        LATBbits.LATB9 = ADCBUF0.B4;
	        LATBbits.LATB10 = ADCBUF0.B3;
	        LATBbits.LATB11 = ADCBUF0.B2;
	        LATBbits.LATB12 = ADCBUF0.B1;
	        LATCbits.LATC13 = ADCBUF0.B0;

	    }

	    LATDbits.LATD1 = ~LATDbits.LATD1;
	}

	void detectarIntT2() org 0x0020  {
	    IFS0bits.T2IF = 0;  //borra bandera de interrupcion de T2

	    LATFbits.LATF0 = !LATFbits.LATF0;

	    ADCON1bits.SAMP = 1; // pedimos muestras
	    asm nop;  // ciclo instruccion sin operacion
	    ADCON1bits.SAMP = 0;  // etener muestra e inicia conversion
	}

	void configADC()  {
	    ADPCFG = 0b111110;  // elegimos AN0 como entrada para muestras
	    ADCHS = 0b0000; // usamos AN0 para recibir las muestras en el ADC
	    ADCON1bits.SSRC = 0b000; // muestreo manual
	    ADCON1bits.ADON = 0;  // apagamos ADC
	    ADCON2bits.VCFG = 0b000;  // tension de referencia 0 y 5
	    IEC0bits.ADIE=1;  // habilitamos interrupcion del ADC
	}

	void configT2()  {
	    PR2 = 5000;  
	    IEC0bits.T2IE = 1; // habilitamos interrupciones para T2
	}

	void configPuertos()  {

	    TRISCbits.TRISC14 = 0;  // Bit mas significativo de la senal generada
	    TRISBbits.TRISB2 = 0;
	    TRISBbits.TRISB3 = 0;
	    TRISBbits.TRISB4 = 0;
	    TRISBbits.TRISB5 = 0;
	    TRISBbits.TRISB6 = 0;
	    TRISBbits.TRISB8 = 0;
	    TRISBbits.TRISB9 = 0;
	    TRISBbits.TRISB10 = 0;
	    TRISBbits.TRISB11 = 0;
	    TRISBbits.TRISB12 = 0;
	    TRISCbits.TRISC13 = 0;  // Bit menos significativo de la senal generada

	    TRISDbits.TRISD1=0;  // Debug

	    TRISBbits.TRISB0 = 1;  // AN0

	    TRISFbits.TRISF0 = 0;  // Debug 
	    TRISFbits.TRISF1 = 0;  // Debug 

	    TRISFbits.TRISF4 = 1;  // Filtro y no filtro

	    TRISFbits.TRISF5 = 0;  // Led indicador de filtro aplicado
	}

	void main()  {
	    configPuertos();
	    configT2();
	    configADC();

	    ADCON1bits.ADON = 1;

	    T2CONbits.TON=1;

	    while(1)  {
	    }
	}



Ejercicio 14:
============

- Usar la placa con el dsPIC30F4013 y defina los parámetros que considere para lograr lo siguiente:
	- Filtro pasa bajos con frecuencia de corte 200 Hz
	- ADC Automático 
	- DAC R-2R
	- Usar el generador de señales del laboratorio
	- Elegir un pulsador para intercambiar entre:
		- Default: Señal sin procesar
		- 1- Pasa bajos con frecuencia de corte 200 Hz
		- 2- Pasa bajos con frecuencia de corte según se indica para cada alumno

- Entregar:
	- Video de aproximadamente 10 segundos mostrando cómo se atenúa la señal de entrada
	- Código fuente con comentarios en el código y organizado en funciones


**Variaciones por alumno:**

:Agustina:
    Frecuencia de corte para el segundo pasa bajos: 800 Hz
	
    Frecuencia de muestreo: 5 kHz

:Carlos:
    Frecuencia de corte para el segundo pasa bajos: 500 Hz
	
    Frecuencia de muestreo: 7 kHz

:Julián:
    Frecuencia de corte para el segundo pasa bajos: 700 Hz
	
    Frecuencia de muestreo: 4 kHz

:Facundo:
    Frecuencia de corte para el segundo pasa bajos: 600 Hz
	
    Frecuencia de muestreo: 6 kHz







**Transformada Discreta de Fourier (DFT)**

.. figure:: images/clase11/im1.png

- Esta última ecuación escrita en C quedaría:

.. code-block:: c

	float dft( float *x_n, float w, unsigned int NN )  {
	    unsigned short n;
	    float R=0.0, I=0.0;

	    // Bucle for para realizar las sumatorias.
	    for( n=0 ; n<NN ; n++ )  {
	        // Cálculo y sumatoria de los componentes
	        // reales e imaginarios.
	        R += x_n[ n ] * cos( w*n );
	        I += x_n[ n ] * sin( w*n );
	    }

	    return sqrt( R*R + I*I ); 
	}

.. figure:: images/clase11/im2.png	

**Ejemplo:** Deseamos averiguar la presencia de frecuencias de 100 Hz y 300 Hz en una señal de audio muestreada a 4kHz.

.. figure:: images/clase11/im3.png	

.. code-block:: c

	void  detectarIntADC()  org 0x002E  {
	    IFS0bits.AD1IF=0;

	    // Se corren las últimas 64 muestras en el bufer x.
	    for( i=63; i!=0; i-- )
	        x[ i ] = x[ i-1 ];

	    // Se guarda la última muestra.
	    x[0] = ( (float) ADC1BUF0 );

	    cont = cont + 1;  // Se cuentan las muestras tomadas.

	    if ( cont == 64 )  {  // Se espera a recibir 64 muestras.

	        resultado1 = dft( x, 0.1570796326, 64 );
	        resultado2 = dft( x, 0.47123889803846, 64 );

	        if( resultado1 > 500 )
	            LATBbits.LATB0=1;
	        else
	            LATBbits.LATB0=0;

	        if( resultado2 > 500 )
	            LATBbits.LATB1=1;
	        else
	            LATBbits.LATB1=0;

	        cont = 0;
	    }
	}

**Ejemplo:** El cálculo del ejemplo anterior se encuentra optimizado dentro de la biblioteca de funciones de MikroC y se utiliza de la siguiente manera:

- Video demostración: https://www.youtube.com/watch?v=n_HLYY41g1E

.. code-block:: c


	// dsPIC30F4013
	// Placa Easy dsPIC 
	// Entrada analogica AN7 - VRef es AVdd y AVss -
	// Detecta las frecuencias 100 Hz, 200 Hz, 300 Hz, ..., 6300 Hz
	// Publica el resultado en binario en los puertos RB0-RB5 (valores desde 1 al 63)

	const unsigned long CANT_MUESTRAS = 128;  // 128 pares de valores [Re, Im]
	const unsigned long FREC_MUESTREO  = 12800;  // Frecuencia de muestreo.

	unsigned Samples[ CANT_MUESTRAS * 2 ];  // Capacidad para 256. Porque son 128 pares

	// La funcion FFT requiere que las muestras se almacenen en el bloque de datos Y.
	// Este bloque de memoria es una caracteristica de los dsPIC que permite realizar
	// operaciones en una sola instruccion, lo que aumenta la velocidad de calculo.
	ydata unsigned InputSamples[ CANT_MUESTRAS * 2 ];

	unsigned freq = 0;

	// Es un indice para llevar la cuenta de cuantas muestras vamos guardando en Samples.
	unsigned globali = 0;

	// Bandera para saber si ya se encuentra listo el procesamiento FFT para mostrar el resultado.
	char listo = 0;

	void configuracionADC()  {
	    ADPCFG = 0b01111111;  // elegimos AN7 como entrada para muestras
	    ADCHS = 0b0111; // usamos AN7 para recibir las muestras en el ADC
	    ADCON1bits.SSRC = 0b111; //  Internal counter ends sampling and starts conversion (auto convert)
	    ADCON1bits.FORM = 0b11;  // Signed Fractional (DOUT = sddd dddd dd00 0000)
	    ADCON2bits.VCFG = 0b000;  // tension de referencia Avdd y Avss
	}

	// Auxiliary function for converting 1.15 radix point to IEEE floating point variable (needed for sqrt).
	float Fract2Float( int input )  {
	    if ( input < 0 )
	        input = - input;
	    return ( input / 32768. );
	}

	// Analiza los componentes de la FFT para luego publicar el resultado en los puertos RB0-RB5
	// Las muestras "Samples" contiene la secuencia Re, Im, Re, Im...
	void obtenerResultado() {
	    unsigned Re, Im, k, max;
	    unsigned i = 0;  // Solo como indice para ir avanzando sobre InputSamples
	    float    ReFloat, ImFloat, amplitud;

	    // La k corresponde al componente, k=0 para la continua, k=1 para 100 Hz,
	    // k=2 para 200 Hz, etc. hasta k=63 para 6300 Hz
	    k = 0;
	    max = 0;  // Almacena el valor maximo de la amplitud de la muestra DFT
	    freq = 0;  // Reset current max. frequency for new reading

	    // 63 ciclos porque no podria muestrear mas de 63 * 100 Hz = 6300 Hz
	    // (que es la mitad de la frecuencia de muestreo)
	    while ( k < ( CANT_MUESTRAS / 2 ) )  {
	        Re = InputSamples[ i++ ];  // Parte Real de la muestra DFT
	        Im = InputSamples[ i++ ];  // Parte Imaginaria de la muestra DFT

	        ReFloat = Fract2Float( Re );  // Conversion a float
	        ImFloat = Fract2Float( Im );  // Conversion a float

	        // Amplitud de la actual muestra DFT
	        amplitud = sqrt( ReFloat * ReFloat + ImFloat * ImFloat );

	        // DFT esta en escala 1/amplitud, por eso lo volvemos a escala
	        amplitud  = amplitud * CANT_MUESTRAS;

	        if ( k == 0 )
	            amplitud = 0;  // Elimina la continua

	        if ( amplitud > max ) {
	            max = amplitud;  // Almacenamos el valor maximo hasta ahora
	            freq = k;  // Almacenamos el componente con mayor potencia
	        }

	        // Avanzamos de a un componente.
	        // En este caso, nos desplzamos 100 Hz cada vez que incrementamos k
	        k++;
	    }

	    // Con esta linea freq tomaria los valores en Hz de la frecuencia con mas potencia.
	    // freq *= (FREC_MUESTREO / CANT_MUESTRAS);

	    // Desplegamos el valor en los puertos RB0-RB5
	    LATBbits.LATB5 = ( freq & 0b0000000000100000 ) >> 5;
	    LATBbits.LATB4 = ( freq & 0b0000000000010000 ) >> 4;
	    LATBbits.LATB3 = ( freq & 0b0000000000001000 ) >> 3;
	    LATBbits.LATB2 = ( freq & 0b0000000000000100 ) >> 2;
	    LATBbits.LATB1 = ( freq & 0b0000000000000010 ) >> 1;
	    LATBbits.LATB0 = ( freq & 0b0000000000000001 ) >> 0;

	    LATBbits.LATB11 = !LATBbits.LATB11;  // Cada vez que se publica el resultado
	}

	unsigned leerAdc()  {
	    ADCON1bits.SAMP = 1;  // Pedimos una muestra
	    asm nop;  // Tiempo que debemos esperar para que tome una muestra
	    ADCON1bits.SAMP = 0;  // Pedimos que retenga la muestra

	    return ADCBUF0;  // Devolvemos el valor muestreado por el ADC
	}

	// Llena Samples con las muestras en Re y Im se pone en 0. Luego copia en el bloque de memoria Y
	void SampleInput()  {
	    Samples[ globali++ ] = leerAdc();   // Re
	    Samples[ globali++ ] = 0;           // Im

	    LATFbits.LATF1 = !LATFbits.LATF1;  // En este puerto se puede ver la frecuencia de muestreo

	    // Entra a este if cuando ya tiene 128 pares.
	    if ( globali >= ( CANT_MUESTRAS * 2 ) )  {
	        globali = 0;
	        if ( ! listo )  {  // Todavia no tenemos suficientes muestras

	            // Copiamos las muestras del ADC hacia el bloque de memoria Y
	            memcpy( InputSamples, Samples, CANT_MUESTRAS * 2 );

	            // Ya estamos listos para aplicar FFT.
	            // Esto habilita el uso de la funcion FFT en la funcion main()
	            listo = 1;
	        }
	    }
	}

	void  configuracionPuertos()  {
	    TRISFbits.TRISF1 = 0;  // Debug frec de muestreo
	    TRISBbits.TRISB11 = 0;  // Debug cada vez que se publica el resultado

	    // Lo siguientes puertos para mostrar la frecuencia con mayor potencia
	    TRISBbits.TRISB0 = 0;
	    TRISBbits.TRISB1 = 0;
	    TRISBbits.TRISB2 = 0;
	    TRISBbits.TRISB3 = 0;
	    TRISBbits.TRISB4 = 0;
	    TRISBbits.TRISB5 = 0;

	    TRISBbits.TRISB7 = 1;  // AN7 para entrada analogica

	}

	void detectarT2() org 0x0020  {
	    SampleInput();  // Se encarga de tomar las muestras
	    IFS0bits.T2IF = 0;  // Bandera Timer 2
	}

	void configuracionT2()  {
	    PR2 = ( unsigned long )( Get_Fosc_kHz() ) * 1000 / ( 4 * FREC_MUESTREO );
	    IEC0bits.T2IE = 1;  // Habilitamos interrucion del Timer 2
	}

	void main()  {

	    memset( InputSamples, 0, CANT_MUESTRAS * 2 );  // Ponemos en cero el buffer para las muestras

	    configuracionPuertos();

	    configuracionT2();
	    T2CONbits.TON = 1;  // Encendemos Timer 2

	    configuracionADC();
	    ADCON1bits.ADON = 1;  // Encendemos el ADC

	    while ( 1 )  {
	        if ( listo ) {
	            // Calcula FFT en 7 etapas, 128 pares de muestras almacenados en InputSamples.
	            FFT( 7, TwiddleCoeff_128, InputSamples );

	            // Método de inversión de bits, necesario para aplicar el algoritmo de FFT.
	            BitReverseComplex( 7, InputSamples );

	            // Analiza la amplitud de las muestras DFT y publica resultados en RB0-RB5
	            obtenerResultado();  

	            listo = 0;  // Indicamos que publicamos un resultado y ahora esperamos el proximo analisis
	        }
	    }
	}


Ejercicio 15:
============

- Modificar el ejemplo para utilizar la interrupción del ADC.

Ejercicio 16:
============

- Modificar el ejemplo para utilizar la interrupción del ADC y no usar el timer.

Ejercicio 17:
============

- En lugar de realizar el análisis cada 100Hz, realizarlo cada 10Hz.

Ejercicio 18:
============

- Elejir la frecuencia de una cuerda de la guitarra y adaptar el programa para hacer un afinador de esa cuerda.

Ejercicio 19:
============

- Elegir una frecuencia particular y visualizar en los puertos RB la potencia de esa frecuencia (como un vúmetro digital).













**Ejemplo: FFT en entrada en AN8 y envío de datos a través de UART**

- `Descargar desde aquí <https://github.com/cosimani/Curso-PIII-2018/blob/master/resources/clase10/FFTyUART.rar?raw=true>`_

**Ejemplo: FFT en entrada en AN7 y envío de datos a través de UART a una aplicación C++**

- `Descargar desde aquí la aplicación portable <http://www.vayra.com.ar/piii2017/portable.rar>`_

- `Descargar desde aquí el código fuente C++ <http://www.vayra.com.ar/piii2017/fuente.rar>`_

- `Descargar desde aquí el código fuente mikroC <http://www.vayra.com.ar/piii2017/mikroc.zip>`_


