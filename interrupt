#include "F28x_Project.h"
// AO ADICIONAR UM BOTÃO NA PORTA GPIO0 E UM LED NA PORTA GPIO3, EXECUTA UMA INTERRUPCAO EXTERNA 
// Defines
//
#define DELAY (CPU_RATE/1000*6*510)  //Qual period at 6 samples

//
// Globals
//
volatile Uint32 Xint1Count;
volatile Uint32 Xint2Count;
Uint32 LoopCount;

//
// Function Prototypes
//
interrupt void xint1_isr(void);
interrupt void xint2_isr(void);
int ver = 0;

//
// Main
//
void main(void)
{


//
// Step 1. Initialize System Control:
// PLL, WatchDog, enable Peripheral Clocks
// This example function is found in the F2837xD_SysCtrl.c file.
//
   InitSysCtrl();

//
// Step 2. Initialize GPIO:
// This example function is found in the F2837xD_Gpio.c file and
// illustrates how to set the GPIO to its default state.
//
// InitGpio();  // Skipped for this example

//
// Step 3. Clear all interrupts and initialize PIE vector table:
// Disable CPU interrupts
//
   DINT;

//
// Initialize PIE control registers to their default state.
// The default state is all PIE interrupts disabled and flags
// are cleared.
// This function is found in the F2837xD_PieCtrl.c file.
//
   InitPieCtrl();

//
// Disable CPU interrupts and clear all CPU interrupt flags:
//
   IER = 0x0000;
   IFR = 0x0000;

//
// Initialize the PIE vector table with pointers to the shell Interrupt
// Service Routines (ISR).
// This will populate the entire table, even if the interrupt
// is not used in this example.  This is useful for debug purposes.
// The shell ISR routines are found in F2837xD_DefaultIsr.c.
// This function is found in F2837xD_PieVect.c.
//
   InitPieVectTable();

//
// Interrupts that are used in this example are re-mapped to
// ISR functions found within this file.
//
   EALLOW;  // This is needed to write to EALLOW protected registers
   PieVectTable.XINT1_INT = &xint1_isr;
   PieVectTable.XINT2_INT = &xint2_isr;
   EDIS;    // This is needed to disable write to EALLOW protected registers

//
// Step 4. User specific code, enable interrupts:
//
// Clear the counters
//
   Xint1Count = 0; // Count XINT1 interrupts
   Xint2Count = 0; // Count XINT2 interrupts
   LoopCount = 0;  // Count times through idle loop

//
// Enable XINT1 and XINT2 in the PIE: Group 1 interrupt 4 & 5
// Enable INT1 which is connected to WAKEINT:
//
   PieCtrlRegs.PIECTRL.bit.ENPIE = 1;          // Enable the PIE block
   PieCtrlRegs.PIEIER1.bit.INTx4 = 1;          // Enable PIE Group 1 INT4
   PieCtrlRegs.PIEIER1.bit.INTx5 = 1;          // Enable PIE Group 1 INT5
   IER |= M_INT1;                              // Enable CPU INT1
   EINT;                                       // Enable Global Interrupts

//
// GPIO30 & GPIO31 are outputs, start GPIO30 high and GPIO31 low
//
   EALLOW;
   GpioDataRegs.GPASET.bit.GPIO30 = 1;         // Load the output latch
   GpioCtrlRegs.GPAMUX2.bit.GPIO30 = 0;        // GPIO
   GpioCtrlRegs.GPADIR.bit.GPIO30 = 1;         // output

   GpioDataRegs.GPACLEAR.bit.GPIO31 = 1;       // Load the output latch
   GpioCtrlRegs.GPAMUX2.bit.GPIO31 = 0;        // GPIO
   GpioCtrlRegs.GPADIR.bit.GPIO31 = 1;         // output

   GpioCtrlRegs.GPAPUD.bit.GPIO3 = 0;
   GpioCtrlRegs.GPAGMUX1.bit.GPIO3 = 0;
   GpioCtrlRegs.GPAMUX1.bit.GPIO3 = 0;
   GpioCtrlRegs.GPADIR.bit.GPIO3 = 1;
   EDIS;

//
// GPIO0 and GPIO1 are inputs
//
   EALLOW;
   GpioCtrlRegs.GPAMUX1.bit.GPIO0 = 0;         // GPIO
   GpioCtrlRegs.GPADIR.bit.GPIO0 = 0;          // input
   GpioCtrlRegs.GPAQSEL1.bit.GPIO0 = 0;        // XINT1 Synch to SYSCLKOUT only

   GpioCtrlRegs.GPAMUX1.bit.GPIO1 = 0;         // GPIO
   GpioCtrlRegs.GPADIR.bit.GPIO1 = 0;          // input
   GpioCtrlRegs.GPAQSEL1.bit.GPIO1 = 2;        // XINT2 Qual using 6 samples
   GpioCtrlRegs.GPACTRL.bit.QUALPRD0 = 0xFF;   // Each sampling window
                                               // is 510*SYSCLKOUT
   EDIS;

//
// GPIO0 is XINT1, GPIO1 is XINT2
//
   GPIO_SetupXINT1Gpio(0);
   GPIO_SetupXINT2Gpio(1);

//
// Configure XINT1
//
   XintRegs.XINT1CR.bit.POLARITY = 0;          // Falling edge interrupt
   XintRegs.XINT2CR.bit.POLARITY = 1;          // Rising edge interrupt

//
// Enable XINT1 and XINT2
//
   XintRegs.XINT1CR.bit.ENABLE = 1;            // Enable XINT1
   XintRegs.XINT2CR.bit.ENABLE = 1;            // Enable XINT2


}

//
// xint1_isr - External Interrupt 1 ISR
//
interrupt void xint1_isr(void)
{
    GpioDataRegs.GPATOGGLE.bit.GPIO3 = 1;
    ver = 1;

    //
    // Acknowledge this interrupt to get more from group 1
    //
    PieCtrlRegs.PIEACK.all = PIEACK_GROUP1;
}

//
// xint2_isr - External Interrupt 2 ISR
//
interrupt void xint2_isr(void)
{
    GpioDataRegs.GPATOGGLE.bit.GPIO3 = 1;
    ver = 1;

    //
    // Acknowledge this interrupt to get more from group 1
    //
    PieCtrlRegs.PIEACK.all = PIEACK_GROUP1;
}

//
// End of file
//
