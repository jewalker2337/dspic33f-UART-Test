# dspic33f-UART-Test
This is being used to get the UART code up and running. Once it works it will be ported to the main Inverter files
/*
 * File:   main.c
 * Author: jewalker
 *
 * Created on July 7, 2017, 1:55 PM
 */


#include "xc.h"
#include "p33fxxxx.h"

#include <math.h>
//#include "pps.h"



#define FOSC    (80000000ULL)
#define FCY     (FOSC/2)
#define high 1
#define low 0

 /*******************************
 * Set device configuration values
********************************/
_FOSCSEL(FNOSC_FRC    & IESO_ON);
_FOSC(FCKSM_CSECME & OSCIOFNC_OFF & POSCMD_NONE & IOL1WAY_ON);
_FWDT(FWDTEN_OFF);
_FICD(JTAGEN_OFF); 
//_FPOR(FPWRT_PWR128);
/*Global Variables*/


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
     

     while(1)
     {
             //Do Something
     } 
       return(0);
 }


void InitOutPutPins(void)
{
         
       TRISA = 0;
       TRISB = 0;
       ADPCFG = 0xfff;
       
       //LATBbits.LATB9 = high;
       //LATBbits.LATB10 = low;
       
       
}

void RemapPins(void)
{
    OSCCONL = 0x46; // see page 151 of dspic33fj06gs302 manual
    OSCCONL = 0x57; // see page 151 of dspic33fj06gs302 manual
    OSCCONbits.IOLOCK = 0; // Unlock remappable I/O

    RPINR18bits.U1RXR = 1;//RP1, Pin 9 (RB1) UART Rx see page 159 dspic33fj06gs302 manual
    RPOR1bits.RP2R2 = 3;//RP2, Pin 10(RB2) UART Tx see page 150 of dspic33fj06gs302 manual
    
    OSCCONL = 0x46; // see page 151 of dspic33fj06gs302 manual
    OSCCONL = 0x57; // see page 151 of dspic33fj06gs302 manual
    OSCCONbits.IOLOCK = 1; // Lock remappable I/O

}
