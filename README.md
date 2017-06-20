# Graphics-controller-ILI9341

The graphics library is intended to provide capabilities to drive the TFT color display, it implements the low level driver SPI communication, data and command transmission to the LCD, and presents the middleware graphics layer. The final result is a library to perform drawing of basic graphics, with colours and shapes like circles, lines, rectangles, text, etc.

# Register driver for SPI - IP on ZYNQ device
```
TFT_SPI_DISPLAY_240x320.h
```
# SPI driver for ILI9341

The SPI drivers for the ILI9341 controller is the lowest abstraction level of the graphics library, this implements the SPI setup, for sending data and the graphics commands. The code is implemented in tft_spi_driver.c.

```C
/*
 * tft_spi_driver.c
 *
 *  Created on: 2 de dic. de 2016
 *      Author: Yarib Nevárez
 */
#include "xparameters.h"
#include "TFT_SPI_DISPLAY_240x320.h"
#include "tft_spi_driver.h"

void tft_spi_initialize(void)
{
    SET_TFT_SPI_BAUD_RATE_DIVIDER(0xFF);
    SET_TFT_SPI_SETTLE_TIME(3);

    SET_TFT_DATA_COMMAND(1);

    SET_TFT_SPI_CLOCK_POLARITY(1);
    SET_TFT_SPI_CLOCK_PHASE(1);

    SET_TFT_CS_FORCE(0);
    SET_TFT_SPI_DATA_LENGTH(0);
}

void tft_spi_baud_rate(uint8_t baud_rate)
{
    SET_TFT_SPI_BAUD_RATE_DIVIDER(baud_rate);
    SET_TFT_SPI_SETTLE_TIME(2);
}

void tft_hardware_initialize(void)
{
    int nop = 10000;
    SET_TFT_DATA_COMMAND(1);
    SET_TFT_RESET(0);
    while(nop--);
    SET_TFT_RESET(1);
}

void tft_spi_write_data(uint8_t data)
{
    while(!GET_TFT_TRANSMISSION_DONE);
    SET_TFT_SPI_DATA_LENGTH(0);

    TFT_SPI_DATA = data;
}

void tft_spi_write_data16(uint16_t word)
{
    while(!GET_TFT_TRANSMISSION_DONE);
    SET_TFT_SPI_DATA_LENGTH(1);

    TFT_SPI_DATA = word;
}

void tft_spi_write_command(uint8_t cmd)
{
    SET_TFT_DATA_COMMAND(0);
    tft_spi_write_data(cmd);
    SET_TFT_DATA_COMMAND(1);
}
```

In the previous code it can be seen that the SPI configuration: CPOL = 1, CPHA = 1, 8 and 16 bits.


#Graphics routines library

The graphics library provides the middle ware software layer for graphics. The routines to draw pixels, lines, rectangles, circles, text, etc, are implemented in this layer. This layer is implemented in a C++ style using OOP design, the class is defined in tft_graphics.h.
The next code exhibit the class definition.

```C
/*
 * tft_graphics.h
 *
 *  Created on: 2 de dic. de 2016
 *      Author: Yarib Nevárez    yarib_007@hotmail.com
 */

#ifndef SRC_TFT_DISPLAY_TFT_GRAPHICS_H_
#define SRC_TFT_DISPLAY_TFT_GRAPHICS_H_

#include "xil_types.h"
#include "ili9341.h"

typedef enum {
    ROT0 = 0,   // Portrait
    ROT90 = 1,  // Landscape
    ROT180 = 2, // Flipped portrait
    ROT270 = 3  // Flipped landscape
} TFTGraphics_Rotation;

typedef struct
{
    void (*initialize)     (void);
    void (*speed)          (uint8_t rate);
    void (*setAddress)     (uint16_t x1,uint16_t y1,uint16_t x2,uint16_t y2);
    void (*drawRectFilled) (uint16_t x, uint16_t y, uint16_t w, uint16_t h, uint16_t colour);
    void (*drawPixel)      (uint16_t x, uint16_t y, uint16_t colour);
    void (*drawPixel_2)    (uint16_t x, uint16_t y, uint8_t size, uint16_t colour);
    void (*drawLine)       (uint16_t x0, uint16_t y0, uint16_t x1, uint16_t y1, uint16_t colour);
    void (*drawRect)       (uint16_t x,uint16_t y,uint16_t w,uint16_t h,uint16_t colour);
    void (*drawClear)      (uint16_t colour);
    void (*setRotation)    (TFTGraphics_Rotation rotation);
    void (*drawCircle)     (uint16_t poX, uint16_t poY, uint16_t radius, uint16_t colour);
    void (*drawChar)       (uint16_t x, uint16_t y, char c, uint8_t size, uint16_t colour, uint16_t bg);
    void (*drawString)     (uint16_t x, uint16_t y, const char *string, uint8_t size, uint16_t colour, uint16_t bg);
    void (*setupScrollArea)(uint16_t TFA, uint16_t BFA);
    void (*scrollAddress)  (uint16_t VSP);
    int  (*scrollLine)     (void);
} TFTGraphics;

TFTGraphics * TFTGraphics_instance(void);

#endif /* SRC_TFT_DISPLAY_TFT_GRAPHICS_H_ */
```

The following lines show the initialization procedure, this code initializes the TFT LCD and draws a red line in diagonal of the screen.

```C
static uint8_t Poxi_initGraphics(void)
{
    Poxi_graphics = TFTGraphics_instance();
    uint8_t rc = Poxi_graphics != NULL;

    if (rc)
    {
        Poxi_graphics->initialize();
        Poxi_graphics->speed(0xF0); // Regular SPI speed
        Poxi_graphics->drawLine(00, 00, ILI9341_WIDTH, ILI9341_HEIGHT, RED); // TEST!
        Poxi_graphics->speed(0xFE); // Speed up !!!
    }

    return rc;
}
```
For more detailed information it can be referred to the actual code.

Best regards,

-Yarib Nevárez
