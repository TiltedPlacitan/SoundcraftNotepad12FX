// ============================================================================
// 
// Copyright 2024 by Jack Bates.  Distributed under the MIT License.
//
// Permission is hereby granted, free of charge, to any person
// obtaining a copy of this software and associated documentation
// files (the “Software”), to deal in the Software without restriction,
// including without limitation the rights to use, copy, modify, merge,
// publish, distribute, sublicense, and/or sell copies of the Software,
// and to permit persons to whom the Software is furnished to do so,
// subject to the following conditions:
// 
// The above copyright notice and this permission notice shall be
// included in all copies or substantial portions of the Software.
//
// THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND,
// EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
// MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
// NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS
// BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN
// ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
// CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
// SOFTWARE.
//
// ============================================================================

// ============================================================================
//
// THIS CODE IS FOR THE SOUNDCRAFT NOTEPAD 12FX
//
// The USB interface will send MIC 1/2 as channels 1 and 2.
//
// Channels 3 and 4 are selectable, and can be one of:
//
//      MIC 3/4
//      STEREO 5/6
//      STEREO 7/8
//      MAIN L/R
//
// This little command-line program, written in C against Linux libusb allows
// you to easily switch sources.  It may be possible to use this on other
// SOUNDCRAFT NOTEPAD mixers, but I have not tested it.
//
// TO USE: from the command line
//         the first invocation will set source to MIC 3/4
//
//      sudo notebook-12fx-source
//
//          -or-
//
//      sudo notebook-12fx-source 34
//
//          -or-
//
//      sudo notebook-12fx-source 56
//
//          -or-
//
//      sudo notebook-12fx-source 78
//
//          -or-
//
//      sudo notebook-12fx-source LR
//
// The default permissions settings for USB devices requires sudo, in order to
// open the device.
//
// TO COMPILE: NOTE: requires build-essential and libusb 1.0.
//
//      sudo apt install build-essential libusb-1.0-0-dev
//
//      gcc -I /usr/include/libusb-1.0 -o notepad-12fx-source notepad-12fx-source.c -lusb-1.0
//
// ============================================================================

#include <assert.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <libusb.h>
#include <err.h>

#define VID 0x05fc  // VID: SOUNDCRAFT NOTEPAD 12FX
#define PID 0x0032  // PID: SOUNDCRAFT NOTEPAD 12FX

// number of possible sources settings on this mixer
#define NOTEPAD_12FX_NUM_SOURCES 4

// ============================================================================

int main(int argc, char * argv[])
{
    // what we expect to see on the command line
    const char * sources[NOTEPAD_12FX_NUM_SOURCES] = { "34", "56", "78", "LR" };

    unsigned char source = 0xFF;
    if (argc > 1)
    {
        assert(argc == 2);
        for (int i = 0; i < NOTEPAD_12FX_NUM_SOURCES; i++)
        {
            if (!strcmp(argv[1], sources[i]))
            {
                source = i;
                break;
            }
        }
    } else
    {
        source = 0;
    }
    if (source == 0xFF)
    {
        fprintf(stderr, "INVALID SOURCE - specify '34' '56' '78' or 'LR'\n");
        exit(1);
    }

    if (libusb_init(NULL) != 0)
    {
        fprintf(stderr, "libusb: iniitialization failed\n");
        exit(1);
    }

    struct libusb_device_handle * handle = libusb_open_device_with_vid_pid(NULL, VID, PID);
    if (!handle)
    {
        fprintf(stderr, "libusb: open device failed\n");
        exit(1);
    }

    uint8_t       bmReqType = 0x40;
    uint8_t            bReq = 16;
    uint16_t           wVal = 0;
    uint16_t         wIndex = 0;
    unsigned char   data[8] = { 0x00 ,0x00 ,0x04 ,0x00 ,source ,0x00 ,0x00 ,0x00 };
    uint16_t           wLen = 8;
    unsigned int         to = 0;

    if (libusb_control_transfer(handle,bmReqType,bReq,wVal,wIndex,data,wLen,to) != 0)
    {
        fprintf(stderr, "libusb: control transfer failed\n");
        exit(1);
    }

    libusb_exit(NULL);
}

// ============================================================================
