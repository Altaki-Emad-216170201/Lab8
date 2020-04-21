# Lab8
/*
 * Copyright 2016-2020 NXP
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without modification,
 * are permitted provided that the following conditions are met:
 *
 * o Redistributions of source code must retain the above copyright notice, this list
 *   of conditions and the following disclaimer.
 *
 * o Redistributions in binary form must reproduce the above copyright notice, this
 *   list of conditions and the following disclaimer in the documentation and/or
 *   other materials provided with the distribution.
 *
 * o Neither the name of NXP Semiconductor, Inc. nor the names of its
 *   contributors may be used to endorse or promote products derived from this
 *   software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
 * ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
 * WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR
 * ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
 * (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
 * LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
 * ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
 * (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
 * SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */
 
/**
 * @file    MKL43Z256xxx4_Project_lab 8.c
 * @brief   Application entry point.
 */
#include <stdio.h>
#include "board.h"
#include "peripherals.h"
#include "pin_mux.h"
#include "clock_config.h"
#include "MKL43Z4.h"
#include "fsl_debug_console.h"
/* TODO: insert other include files here. */

/* TODO: insert other definitions and declarations here. */

/*
 * @brief   Application entry point.
 */
int main(void) {

  	/* Init board hardware. */
    BOARD_InitBootPins();
    BOARD_InitBootClocks();
    BOARD_InitBootPeripherals();
  	/* Init FSL debug console. */
    BOARD_InitDebugConsole();

    PRINTF("Hello World\n");

    SIM->SCGC5 |= ((1<<9) | (1<<12));
    PORTA->PCR[4] = 0x103; // port A pin 4
    PORTD->PCR[5] = 0x100; // port D pin 5
    PORTC->PCR[3] = 0x103; // port C pin 3
    PORTE->PCR[31] = 0x100;// port E pin 31
    PTA->PDDR &= ~(0x10);
    PTC->PDDR &= ~(0x10);
    PTD->PDDR |= (1<<5);
    PTE->PDDR |= (1<<31);

    /* Force the counter to be placed into memory. */
    volatile static int i = 0;
    volatile static int sw1 = 0;
    volatile static int sw3 = 0;

    PTD->PDOR & (1<<5); 	// read switch
    PTE->PDOR & (1<<31);	// read switch

    /* Enter an infinite loop, just incrementing a counter. */
    while(1) {
    	sw1 = PTA->PDIR & (1<<4); // read switch
    	sw3 = PTC->PDIR & (1<<3); // read switch

    	if(!sw1 && sw3){
    		if(i==0){
    			PTD->PDOR = 0; 		// LED on
    			PTE->PDOR = 0;		// LED on
    		}
    		else{
    			PTD->PDOR = (1<<5); // LED off
    			PTE->PDOR = 0;		// LED on
    		}
    	}
    	if(!sw1 && !sw3){
    		PTD->PDOR = 0;			// LED on
    		PTE->PDOR = (1<<31);	// LED off
    		i = 1;
    	}
    	if(sw1 && sw3){
    		PTD->PDOR = (1<<5); 	// LED off
    		PTE->PDOR = (1<<31);	// LED off
    		i = 0;
    	}

    	if(i==1 && (sw1 && !sw3)){
    		PTD->PDOR = (1<<5); 	// LED off
    		PTE->PDOR = 0;			// LED on
    	}
        /* 'Dummy' NOP to allow source level single stepping of
            tight while() loop */
        __asm volatile ("nop");
    }
    return 0 ;
}
