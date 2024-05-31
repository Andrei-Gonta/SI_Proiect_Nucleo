// FUNCTIA MAIN

int main(void)
{
	U8 status = 0;

	PeriphInit();
	
  keypad_init();
	LCD_init();
	RCC->AHB1ENR |=  2;             /* enable GPIOB clock */
  GPIOB->MODER &= ~0x0000ff00;    /* clear pin mode */
  GPIOB->MODER |=  0x00005500;    /* set pins to output mode */
	
		
	
		LCD_data('P');
		LCD_data('A');
		LCD_data('S');
		LCD_data('S');
		LCD_data('W');
		LCD_data('O');
		LCD_data('R');
		LCD_data('D');
		LCD_data(':');
		newLine(9);
		char triesChar1[100];	
	sprintf(triesChar1, "TRIES: %d", tries);
		writeStringLCD(triesChar1);
		while(1) 
			{
		
		delayMs(1000);
				if(tries>1)
				{
					citire();
					
					if(count==4)
			
					{
						 /* clear LCD display */
						LCD_command(1);
						delayMs(1000);
					
						//writeLEDs(0x0); 
						LCD_data('U');
						LCD_data('N');
						LCD_data('L');
						LCD_data('O');
						LCD_data('C');
						LCD_data('K');
						LCD_data('E');
						LCD_data('D');
						LCD_data('!');
						k=0;
					  count=0;
						writeLEDs(0x0);
						
						//while(1);
						break;
					}
					else
					{
						 /* clear LCD display */
						LCD_command(1);
						delayMs(1000);
					
						tries--;
						LCD_data('W');
						LCD_data('R');
						LCD_data('O');
						LCD_data('N');
						LCD_data('G');
						newLine(7);
						writeStringLCD(". Tries = ");
						//writeIntLCD(t,tries);
						char triesChar[100];	
						sprintf(triesChar, "%d", tries);
						
						writeStringLCD(triesChar);
						//writeLEDs(0xF);

						k=0;
					  count=0;
						delayMs(1000);
						LCD_command(1);
						
						LCD_data('P');
						LCD_data('A');
						LCD_data('S');
						LCD_data('S');
						LCD_data('W');
						LCD_data('O');
						LCD_data('R');
						LCD_data('D');
						LCD_data(':');
						
						writeLEDsOptional(0xF, tries);

						citire();
					}
			}
		else
				{
				LCD_command(1);
				LCD_data('L');
        LCD_data('O');
				LCD_data('C');
				LCD_data('K');
				LCD_data('E');
				LCD_data('D');
				LCD_data('!');
				//delayMs(1000);
				//LCD_command(1);
				writeLEDs(0xF);
				break;
				}
	}
		
}




// FUNCTII UTILIZATE

/* configure SPI1 and the associated GPIO pins */
void LCD_init(void) {
    RCC->AHB1ENR |= 1;              /* enable GPIOA clock */
    RCC->AHB1ENR |= 4;              /* enable GPIOC clock */
    RCC->APB2ENR |= 0x1000;         /* enable SPI1 clock */

    /* PORTA 5, 7 for SPI1 MOSI and SCLK */
    GPIOA->MODER &= ~0x0000CC00;    /* clear pin mode */
    GPIOA->MODER |=  0x00008800;    /* set pin alternate mode */
    GPIOA->AFR[0] &= ~0xF0F00000;   /* clear alt mode */
    GPIOA->AFR[0] |=  0x50500000;   /* set alt mode SPI1 */

    /* PA12 as GPIO output for SPI slave select */
    GPIOA->MODER &= ~0x03000000;    /* clear pin mode */
    GPIOA->MODER |=  0x01000000;    /* set pin output mode */

    /* initialize SPI1 module */
    SPI1->CR1 = 0x31F;
    SPI1->CR2 = 0;
    SPI1->CR1 |= 0x40;              /* enable SPI1 module */

    /* LCD controller reset sequence */
    delayMs(20);
    LCD_nibble_write(0x30, 0);
    delayMs(5);
    LCD_nibble_write(0x30, 0);
    delayMs(1);
    LCD_nibble_write(0x30, 0);
    delayMs(1);
    LCD_nibble_write(0x20, 0);  /* use 4-bit data mode */
    delayMs(1);
    LCD_command(0x28);          /* set 4-bit data, 2-line, 5x7 font */
    LCD_command(0x06);          /* move cursor right */
    LCD_command(0x01);          /* clear screen, move cursor to home */
    LCD_command(0x0F);          /* turn on display, cursor blinking */
}

void LCD_nibble_write(char data, unsigned char control) {
    data &= 0xF0;       /* clear lower nibble for control */
    control &= 0x0F;    /* clear upper nibble for data */
    SPI1_write (data | control);           /* RS = 0, R/W = 0 */
    SPI1_write (data | control | EN);      /* pulse E */
    delayMs(0);
    SPI1_write (data);
}

// FUNCTIE PENTRU CLEAR LCD
void LCD_command(unsigned char command) {
    LCD_nibble_write(command & 0xF0, 0);    /* upper nibble first */
    LCD_nibble_write(command << 4, 0);      /* then lower nibble */

    if (command < 4)
        delayMs(2);         /* command 1 and 2 needs up to 1.64ms */
    else
        delayMs(1);         /* all others 40 us */
}

// FUNCTIE PENTRU AFISARE PE LCD
void LCD_data(char data) {
    LCD_nibble_write(data & 0xF0, RS);      /* upper nibble first */
    LCD_nibble_write(data << 4, RS);        /* then lower nibble */

    delayMs(1);
}


void delayMs(int n) {
    int i;
    for (; n > 0; n--)
        for (i = 0; i < 3195; i++) ;
}



void newLine(unsigned int size){
    for(unsigned int i=0; i < 40-size; i++)
    {
        LCD_data(' ');
    }
}

// FUNCTIE PENTRU UTILIZAREA TASTATURII 4X4
char keypad_getkey(void)  
{
    int row, col;

    /* check to see any key is pressed first */
    outputEnableCols(0xF);      /* enable all columns */
    writeCols(0xF);             /* and drive them high */
    delay();                    /* wait for signal to settle */
    row = readRows();           /* read all rows */
    writeCols(0x0);             /* discharge all columns */
    outputEnableCols(0x0);      /* disable all columns */
    if (row == 0) return 0;     /* if no key pressed, return a zero */

    /* If a key is pressed, it gets here to find out which key.
     * It activates one column at a time and read the rows to see
     * which is active.
     */
    for (col = 0; col < 4; col++) {
        outputEnableCols(1 << col); /* enable one column */
        writeCols(1 << col);        /* turn the active row high */
        delay();                    /* wait for signal to settle */
        row = readRows();           /* read all rows */
        writeCols(0x0);             /* discharge all columns */
        if (row != 0) break;        /* if one of the row is low, some key is pressed. */
    }

    outputEnableCols(0x0);          /* disable all columns */
    if (col == 4)
        return 0;                   /* if we get here, no key is pressed */

    /* gets here when one of the rows has key pressed.
     * generate a unique key code and return it.
     */
    if (row == 0x01) {return 0 + col;}    // key in row 0
    if (row == 0x02) {return 4 + col;  }  // key in row 1
    if (row == 0x04) {return 8 + col;   } // key in row 2
    if (row == 0x08) {return 12 + col;   }// key in row 3

    return 0;   /* just to be safe */
}

//FUNCTIE PENTRU APRINDERE 4 LEDURI
void writeLEDs(char n) {
    GPIOB->BSRR = 0x00F00000;   // turn off all LEDs
    GPIOB->BSRR = n << 4;       // turn on LEDs
}

FUNCTIE PENTRU APRINDERE 'nedLED' numar de leduri
void writeLEDsOptional(char n, int nedLed)
{
		GPIOB->BSRR = 0x00F00000;   // turn off all LEDs
    GPIOB->BSRR = n << nedLed;       // turn on LEDs
}

// FUNCTIE CARE CITESTE SI VERIFICA PIN
void citire()
{
 while(k<4)
    {
    while((key = keypad_getkey()) == 0);
    
        if(key != corect[k])
        {
             
        }
        else
        {
            ++count;
        }
        ++k;
        while(keypad_getkey() != 0);
    }
}
