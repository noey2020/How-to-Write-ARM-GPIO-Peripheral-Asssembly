# How-to-Write-ARM-GPIO-Peripheral-Asssembly

Write ARM GPIO Assembly with no CMSIS!		June 29, 2020

I appreciate comments. Shoot me an email at noel_s_cruz@yahoo.com!

Hire Me!

Write constants patterned after stm32l1xx.h and utilize data structures defined to
increase readability & portability.

Study this skeleton assembly and the pattern for defines and the load, modify,
and store operations on the STM32 core register file. LDR, MOV, ORR, BIC, and STR are
the favorite instructions to use.

Indexed addressing resembles from this code snippet or actually C pointers take their 
cue from indexed addressing, e.g.,

	LDR r7, =RCC_BASE	  ; Load address of reset & clock control (RCC)
  
	LDR r1, [r7, #RCC_AHBENR]
	
As an aside, CMSIS provides lots of starter code which you don't have to rewrite from
scratch. The CMSIS core header files give you minimal set of functions & macros to 
access the key Cortex-M processor registers. Startup code, system code, and device
header files give you more pre-written code to play around.

What is good about CMSIS is, it is open source but is subject to GNU General Public
License v3 license copyright to use. Just include the copyright notice and you should 
be ok. CMSIS is just a suggestion to adopt. But it does save you a lot of effort and
time.
	
Skeleton assembly program:

;		INCLUDE stm32l1xx_constants.s       ; Load Constant Definitions

;		INCLUDE stm32l1xx_tim_constants.s   ; TIM Constants

		AREA myData, DATA, READWRITE
    
; Or INCLUDE stm32l1xx_constants.s       ; Load Constant Definitions 

; Originally derived from #defines in stm32l1xx.h. From chip manufacturer

; 		Peripheral Memory Map

PERIPH_BASE           EQU   (0x40000000) ; Peripheral base address in the alias region 

AHBPERIPH_BASE        EQU   (PERIPH_BASE + 0x20000)

GPIOB_BASE            EQU   (AHBPERIPH_BASE + 0x0400)

RCC_BASE              EQU   (AHBPERIPH_BASE + 0x3800)

;************ General Purpose IO ************

; Byte offset of each variable in GPIO_TypeDef structure

GPIO_MODER		EQU   0x00;

GPIO_OTYPER		EQU   0x04; uint16_t

GPIO_RESERVED0  EQU   0x06; uint16_t

GPIO_OSPEEDR	EQU   0x08;

GPIO_PUPDR		EQU   0x0C;

GPIO_IDR		EQU   0x10; uint16_t

GPIO_RESERVED1  EQU   0x12; uint16_t

GPIO_ODR		EQU   0x14; uint16_t

GPIO_RESERVED2  EQU   0x16; uint16_t

GPIO_BSRRL		EQU   0x18; uint16_t BSRR register is split to 2 * 16-bit fields BSRRL 

GPIO_BSRRH		EQU   0x1A; uint16_t BSRR register is split to 2 * 16-bit fields BSRRH 

GPIO_LCKR		EQU   0x1C;

GPIO_AFR0		EQU   0x20;  AFR[0]

GPIO_AFR1		EQU   0x24;  AFR[1]

GPIO_AFRL       EQU   0x20

GPIO_AFRH       EQU   0x24

;************ Reset and Clock Control ************

; Byte offset of variable AHBENR in RCC_TypeDef structure

RCC_AHBENR      EQU   0x1C

		AREA myCode, CODE, READONLY
    
		EXPORT __main		; __main and not _main as defined 
    
		ALIGN			; startup_stm32l1xx_md.s. lines 192,
    
		ENTRY			; 217, and 36
    
__main	PROC

		; Enable clock to GPIO port B
    
		LDR r7, =RCC_BASE			; Load address of reset & clock control (RCC)
    
		LDR r1, [r7, #RCC_AHBENR]	; r1 = RCC->AHBENR
    
		ORR r1, r1, #0x00000002		; Set bit 2 of AHBENR
    
		STR r1, [r7, #RCC_AHBENR]	; GPIO port B clock enable	
    
		
		; Set pin 6 IO mode as general-purpose output
    
		LDR r7, =GPIOB_BASE			; Load GPIO port B base address
    
		LDR r1, [r7, #GPIO_MODER]	; r1 = GPIOB->MODER
    
		BIC r1, r1, #(0x03 << 12)	; Direction mask pin 6. clear bits 12 & 13
    
		ORR r1, r1, #(0x1 << 12)	; Set mode as digital output (0b01)
    
		STR r1, [r7, #GPIO_MODER]	; Save 
		
		; Set pin 6 push-pull mode as output type
		LDR r1, [r7, #GPIO_OTYPER]	; r1 = GPIO->OTYPER
		BIC r1, r1, #(1 << 6)		; Push-pull (0, default), open-drain(1)
		STR r1, [r7, #GPIO_OTYPER]	; Save

		; Set IO output clock freq value, 2MHz
    
		LDR r1, [r7, #GPIO_OSPEEDR]	; r1 = GPIO->OSPEEDR
    
		BIC r1, r1, #(0x03 << 12)	; clock freq mask for pin 6
    
		ORR r1, r1, #(0x03 << 12)	; 400KHx(00), 2MHz(01), 10MHz(10), 40MHz(11)
    
		STR r1, [r7, #GPIO_OSPEEDR]	; Save 

		; Set IO as no PUPD
    
		LDR r1, [r7, #GPIO_PUPDR]	; r1 = GPIO->PUPDR
    
		BIC r1, r1, #(0x03 << 12)	; PUPD mask for pin 6
    
		ORR r1, r1, #(0x00 << 12)	; No PUPD (00, reset), PU (01)
    
		STR r1, [r7, #GPIO_PUPDR]	; Save 

		; Turn on LED
    
		LDR r7, =GPIOB_BASE			; Load GPIO port B base address
    
		LDR r1, [r7, #GPIO_ODR]		; r1 = GPIO->ODR
    
		ORR r1, r1, #(1 << 6)		; Set output pin 6 high
    
		STR r1, [r7, #GPIO_ODR]		; Write to output data register 
    
stop 	b stop

		ENDP
    
		END
		
I appreciate comments. Shoot me an email at noel_s_cruz@yahoo.com!

Hire Me!

Happy coding!
