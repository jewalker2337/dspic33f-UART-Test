/*
 * File:   main.c
 * Author: jewalker
 *
 * Created on July 7, 2017, 1:55 PM
 */


#include "xc.h"
#include "p33fxxxx.h"

#define FOSC    (80000000ULL)
#define FCY     (FOSC/2)
#define high 1
#define low 0
#define Baud_Rate 265 // (Fcy/(16 * Buad Rate)) - 1 , Baud rate = 9600



int j,time,i;
int TestData[] = {'T','E','S','T'};
 
 /*******************************
 * Set device configuration values
********************************/
_FOSCSEL(FNOSC_FRC    & IESO_ON);
_FOSC(FCKSM_CSECME & OSCIOFNC_OFF & POSCMD_NONE & IOL1WAY_ON);
_FWDT(FWDTEN_OFF);
_FICD(JTAGEN_OFF); 

/*Global Variables*/



//Prototypes

void InitUART(void);
void InitOutPutPins(void);
void Delay(int);
void RemapPins(void);
void SendData(void);


int main(void) 
{
 /* **************Configure PLL for FOSC=80MHz using 7.37MHz FRC*****************/
         PLLFBD = 41; // M = 43
         CLKDIVbits.PLLPRE  = 0;  // N1 = 2
         CLKDIVbits.PLLPOST = 0;  // N2 = 2   
 /*******************************************************************************/    
  
/***************Initiate Clock Switch to Internal FRC with PLL (NOSC = 0b001)************/
         
     __builtin_write_OSCCONH( 0x01 );
     __builtin_write_OSCCONL( 0x01 );
  
      // Wait for Clock switch to occur
     while ( OSCCONbits.COSC != 0b001 );
 
     // Wait for PLL to lock
     while ( OSCCONbits.LOCK != 1 );
     
/*****************************************************************************************/
   
       

       InitOutPutPins();
       RemapPins();
       InitUART();

       
       
       
     while(1)
     {
         //Do Something
     } 
       return(0);
 }





void InitUART(void)
{
    
   /*
    * 
    * Below are basically all the registers for the UART Tx module
    * Keeping them here for now for reference
    * 
   U1MODEbits.UARTEN = 0;// disable uart before configuring    
   U1MODEbits.ABAUD = 0; //auto baud disabled
   U1MODEbits.BRGH = 0;// generates 16 clocks per bit period
   U1MODEbits.IREN = 0; //encoder/decoder disabled
   U1MODEbits.LPBACK = 0; //loopback disabled
   U1MODEbits.PDSEL = 0; // 8 bit, no parity
   U1MODEbits.RTSMD = 1; //1=RTS in simplex mode, 0 = flow control mode
   U1MODEbits.STSEL = 0; // 0 = 1 stop bit, 1  = 2 stop bits
   U1MODEbits.UEN = 0; // 0 = Tx/Rx are enabled and used,UCTS/URTS/BCLK controlled by port latches
   U1MODEbits.URXINV = 0; // 0 = UxRX idle state is 1
   U1MODEbits.USIDL = 0; // 0= UART continues in idle state
   U1MODEbits.WAKE = 0; //0 = wake up disabled
   
   U1BRG = Baud_Rate;
   
   U1STAbits.ADDEN = 0; // 0 = address detect mode disabled
   //U1STAbits.FERR  - READ ONLY
   //U1STAbits.OERR - READ/CLEAR ONLY
   //U1STAbits.PERR - READ ONLY
   //U1STAbits.RIDLE - READ ONLY
   //U1STAbits.TRMT - READ ONLY
   //U1STAbits.URXDA - READ ONLY
   //U1STAbits.URXISEL = 1; //determines when the Receive interrupt is triggered
   //U1STAbits.UTXBF - READ ONLY
   U1STAbits.UTXBRK = 0;// 0=Sync break is disabled or incomplete
   U1STAbits.UTXINV = 0; // 0=idle state is 1
   U1STAbits.UTXISEL0 = 0; //determines when Tx interrupt is triggered
     
   
   U1MODEbits.UARTEN = 1;        
   U1STAbits.UTXEN = 1; // Tx is controlled by UART module       
   
   Delay(1000);
   U1TXREG = 0x10;
    * 
    * 
    * 
 */
           
       
        U1MODEbits.UARTEN = 0; // Disable UART 
        U1MODEbits.STSEL = 0; // 1-stop bit
        U1MODEbits.PDSEL = 0; // No Parity, 8-data bits
        U1MODEbits.ABAUD = 0; // Autobaud Disabled
        U1MODEbits.BRGH = 0; // Low Speed mode
    
        U1BRG = Baud_Rate; // BAUD Rate Setting for 9600
        U1STAbits.UTXINV = 1; // bus idle state low
        U1STAbits.URXISEL = 0; // Tx interrput mode. see pg 7 of DS70000582E (UART module))
        U1STAbits.UTXISEL0 = 0; // Interrupt after one Tx character is transmitted
        U1STAbits.UTXISEL1 = 0;   
        IEC0bits.U1TXIE = 1; // Enable UART TX interrupt        
        U1MODEbits.UARTEN = 1; // Enable UART       
        U1STAbits.UTXEN = 1; // Enable UART Tx
        
        
        Delay(50000);
        U1TXREG = 'a'; // Transmit one character
        
        
 
}



void __attribute__((__interrupt__)) _U1TXInterrupt(void)
{
    IFS0bits.U1TXIF = 0; // Clear TX Interrupt flag
    
   if (i>3) // only 4 characters stored
    {
        i=0;
    }
    
        LATAbits.LATA0 = ~LATAbits.LATA0; // blink an led
        U1TXREG = TestData[i];// unload array to UART Tx reg
        i++; // increment
}



void InitOutPutPins(void)
{
         
       TRISA = 0;
       //TRISB = 0;
       ADPCFG = 0xfff; // Port configured for analog or difital. 1 = digital
       
}

void RemapPins(void)
{
    OSCCONL = 0x46; // see page 151 of dspic33fj06gs302 manual
    OSCCONL = 0x57; // see page 151 of dspic33fj06gs302 manual
    OSCCONbits.IOLOCK = 0; // Unlock remappable I/O

    //__builtin_write_OSCCONL(OSCCON & 0xDF);
    
   // RPINR18bits.U1RXR = 1;//RP1, Pin 9 (RB1) UART Rx see page 159 dspic33fj06gs302 manual
    RPOR1bits.RP3R = 0b000011;//RP2, Pin 10(RB2) UART Tx see page 150 of dspic33fj06gs302 manual
    
   // __builtin_write_OSCCONL(OSCCON | 0x40); 
     
    OSCCONL = 0x57; // see page 151 of dspic33fj06gs302 manual
    OSCCONL = 0x46; // see page 151 of dspic33fj06gs302 manual
    OSCCONbits.IOLOCK = 1; // Lock remappable I/O

}


void Delay(time)
{ j=0; 

 //j=0;
 
 while(j<time) //while(j<630) //roughly 0.254ms delay. 315=~60hz
 {
     j++;
 }
}
