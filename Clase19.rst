.. -*- coding: utf-8 -*-

.. _rcs_subversion:

Clase 19 - PIII 2019
====================
(Fecha: 25 de octubre)





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




Probando filtros en Proteus y en Placa
======================================


- Video sobre cómo utilizar el generador de señal (https://www.youtube.com/watch?v=qCRcNYbqBxs)



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



Ejercicio:
==========

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

:Juan:
    Frecuencia de corte para el segundo pasa bajos: 800 Hz
	
    Frecuencia de muestreo: 5 kHz

:Pablo:
    Frecuencia de corte para el segundo pasa bajos: 500 Hz
	
    Frecuencia de muestreo: 7 kHz

:Conrado:
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

	// Function for converting 1.15 radix point to IEEE floating point variable (needed for sqrt).
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

	            listo = 0;  // Indicamos que publicamos un resultado y esperamos el proximo analisis
	        }
	    }
	}


Ejercicio:
==========

- Modificar el ejemplo para utilizar la interrupción del ADC.

Ejercicio:
==========

- Modificar el ejemplo para utilizar la interrupción del ADC y no usar el timer.

Ejercicio:
==========

- En lugar de realizar el análisis cada 100Hz, realizarlo cada 10Hz.

Ejercicio:
==========

- Elejir la frecuencia de una cuerda de la guitarra y adaptar el programa para hacer un afinador de esa cuerda.

Ejercicio:
==========

- Elegir una frecuencia particular y visualizar en los puertos RB la potencia de esa frecuencia (como un vúmetro digital).


