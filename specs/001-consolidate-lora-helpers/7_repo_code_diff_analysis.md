# Complete 7-Repository Code-Level Diff Analysis

This document details EVERY SINGLE CODE DIFFERENCE across all 7 device repositories at git HEAD prior to consolidation.

## 1. Original File Locations in Git HEAD

| Repository | Ra01S.h Location | Ra01S.cpp Location | LoRaHelper.h Location | LoraMsg.h Location | LoraMsg.cpp Location |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Gateway** | lib/Arduino-Ra01S/Ra01S.h | lib/Arduino-Ra01S/Ra01S.cpp | src/LoRaHelper.h | src/LoraMsg.h | src/LoraMsg.cpp |
| **Button** | src/Ra01S.h | src/Ra01S.cpp | src/LoRaHelper.h | src/LoraMsg.h | src/LoraMsg.cpp |
| **DualGateController** | src/Ra01S.h | src/Ra01S.cpp | src/LoRaHelper.h | src/LoraMsg.h | src/LoraMsg.cpp |
| **DualPIR** | include/Ra01S.h | src/Ra01S.cpp | include/LoRaHelper.h | include/LoraMsg.h | src/LoraMsg.cpp |
| **Repeater** | src/Ra01S.h | src/Ra01S.cpp | src/LoRaHelper.h | src/LoraMsg.h | src/LoraMsg.cpp |
| **Victron** | include/Ra01S.h | src/Ra01S.cpp | include/LoRaHelper.h | include/LoraMsg.h | src/LoraMsg.cpp |
| **WaterTankLevel** | include/Ra01S.h | src/Ra01S.cpp | include/LoRaHelper.h | include/LoraMsg.h | src/LoraMsg.cpp |

---

## Code Comparison: Ra01S.h

### Diff: Gateway vs Button

`diff
--- Gateway
+++ Button
@@ -1,5 +1,8 @@
 #ifndef _RA01S_H
 #define _RA01S_H
+
+#define SPI_Speed 500000
+#define ON_AIR_TIMEOUT 1000 //ms to send packet
 
 //return values
 #define ERR_NONE                        0
@@ -25,6 +28,7 @@
 #define XTAL_FREQ                       ( double )32000000
 #define FREQ_DIV                        ( double )pow( 2.0, 25.0 )
 #define FREQ_STEP                       ( double )( XTAL_FREQ / FREQ_DIV )
+#define BUSY_WAIT                       5000
 
 // SX126X Model
 #define SX1261_TRANCEIVER                             0x01
@@ -361,7 +365,6 @@
 #define SX126x_TXMODE_SYNC                            0x02
 #define SX126x_TXMODE_BACK2RX                         0x04
 
-
 // common low-level SPI interface
 class SX126x {
   public:
@@ -417,7 +420,7 @@
     uint8_t  GetRssiInst();
     void     GetRxBufferStatus(uint8_t *payloadLength, uint8_t *rxStartBufferPointer);
     void     Wakeup(void);
-    void     WaitForIdle(unsigned long timeout = 5000);
+    void     WaitForIdle(unsigned long timeout, const char *text, bool stop);
     uint8_t  ReadBuffer(uint8_t *rxData, uint8_t maxLen);
     void     WriteBuffer(uint8_t *txData, uint8_t txDataLen);
     void     WriteRegister(uint16_t reg, uint8_t* data, uint8_t numBytes, bool waitForBusy = true);
`

### Diff: Gateway vs DualGateController

`diff
--- Gateway
+++ DualGateController
@@ -1,5 +1,8 @@
 #ifndef _RA01S_H
 #define _RA01S_H
+
+#define SPI_Speed 500000
+#define ON_AIR_TIMEOUT 1000 //ms to send packet
 
 //return values
 #define ERR_NONE                        0
@@ -25,6 +28,7 @@
 #define XTAL_FREQ                       ( double )32000000
 #define FREQ_DIV                        ( double )pow( 2.0, 25.0 )
 #define FREQ_STEP                       ( double )( XTAL_FREQ / FREQ_DIV )
+#define BUSY_WAIT                       5000
 
 // SX126X Model
 #define SX1261_TRANCEIVER                             0x01
@@ -361,7 +365,6 @@
 #define SX126x_TXMODE_SYNC                            0x02
 #define SX126x_TXMODE_BACK2RX                         0x04
 
-
 // common low-level SPI interface
 class SX126x {
   public:
@@ -417,7 +420,7 @@
     uint8_t  GetRssiInst();
     void     GetRxBufferStatus(uint8_t *payloadLength, uint8_t *rxStartBufferPointer);
     void     Wakeup(void);
-    void     WaitForIdle(unsigned long timeout = 5000);
+    void     WaitForIdle(unsigned long timeout, const char *text, bool stop);
     uint8_t  ReadBuffer(uint8_t *rxData, uint8_t maxLen);
     void     WriteBuffer(uint8_t *txData, uint8_t txDataLen);
     void     WriteRegister(uint16_t reg, uint8_t* data, uint8_t numBytes, bool waitForBusy = true);
`

### Diff: Gateway vs DualPIR

`diff
--- Gateway
+++ DualPIR
@@ -1,5 +1,8 @@
 #ifndef _RA01S_H
 #define _RA01S_H
+
+#define SPI_Speed 500000
+#define ON_AIR_TIMEOUT 1000 //ms to send packet
 
 //return values
 #define ERR_NONE                        0
@@ -24,7 +27,9 @@
 // SX126X physical layer properties
 #define XTAL_FREQ                       ( double )32000000
 #define FREQ_DIV                        ( double )pow( 2.0, 25.0 )
-#define FREQ_STEP                       ( double )( XTAL_FREQ / FREQ_DIV )
+#define FREQ_DIV_2_25                   FREQ_DIV
+#define FREQ_STEP                       ( double )( XTAL_FREQ / FREQ_DIV_2_25 )
+#define BUSY_WAIT                       5000
 
 // SX126X Model
 #define SX1261_TRANCEIVER                             0x01
@@ -361,7 +366,6 @@
 #define SX126x_TXMODE_SYNC                            0x02
 #define SX126x_TXMODE_BACK2RX                         0x04
 
-
 // common low-level SPI interface
 class SX126x {
   public:
@@ -417,7 +421,7 @@
     uint8_t  GetRssiInst();
     void     GetRxBufferStatus(uint8_t *payloadLength, uint8_t *rxStartBufferPointer);
     void     Wakeup(void);
-    void     WaitForIdle(unsigned long timeout = 5000);
+    void     WaitForIdle(unsigned long timeout, const char *text, bool stop);
     uint8_t  ReadBuffer(uint8_t *rxData, uint8_t maxLen);
     void     WriteBuffer(uint8_t *txData, uint8_t txDataLen);
     void     WriteRegister(uint16_t reg, uint8_t* data, uint8_t numBytes, bool waitForBusy = true);
`

### Diff: Gateway vs Repeater

`diff
--- Gateway
+++ Repeater
@@ -1,5 +1,8 @@
 #ifndef _RA01S_H
 #define _RA01S_H
+
+#define SPI_Speed 500000
+#define ON_AIR_TIMEOUT 1000 //ms to send packet
 
 //return values
 #define ERR_NONE                        0
@@ -25,6 +28,7 @@
 #define XTAL_FREQ                       ( double )32000000
 #define FREQ_DIV                        ( double )pow( 2.0, 25.0 )
 #define FREQ_STEP                       ( double )( XTAL_FREQ / FREQ_DIV )
+#define BUSY_WAIT                       5000
 
 // SX126X Model
 #define SX1261_TRANCEIVER                             0x01
@@ -361,7 +365,6 @@
 #define SX126x_TXMODE_SYNC                            0x02
 #define SX126x_TXMODE_BACK2RX                         0x04
 
-
 // common low-level SPI interface
 class SX126x {
   public:
@@ -417,7 +420,7 @@
     uint8_t  GetRssiInst();
     void     GetRxBufferStatus(uint8_t *payloadLength, uint8_t *rxStartBufferPointer);
     void     Wakeup(void);
-    void     WaitForIdle(unsigned long timeout = 5000);
+    void     WaitForIdle(unsigned long timeout, const char *text, bool stop);
     uint8_t  ReadBuffer(uint8_t *rxData, uint8_t maxLen);
     void     WriteBuffer(uint8_t *txData, uint8_t txDataLen);
     void     WriteRegister(uint16_t reg, uint8_t* data, uint8_t numBytes, bool waitForBusy = true);
`

### Diff: Gateway vs Victron

`diff
--- Gateway
+++ Victron
@@ -1,5 +1,8 @@
 #ifndef _RA01S_H
 #define _RA01S_H
+
+#define SPI_Speed 500000
+#define ON_AIR_TIMEOUT 1000 //ms to send packet
 
 //return values
 #define ERR_NONE                        0
@@ -25,6 +28,7 @@
 #define XTAL_FREQ                       ( double )32000000
 #define FREQ_DIV                        ( double )pow( 2.0, 25.0 )
 #define FREQ_STEP                       ( double )( XTAL_FREQ / FREQ_DIV )
+#define BUSY_WAIT                       5000
 
 // SX126X Model
 #define SX1261_TRANCEIVER                             0x01
@@ -361,7 +365,6 @@
 #define SX126x_TXMODE_SYNC                            0x02
 #define SX126x_TXMODE_BACK2RX                         0x04
 
-
 // common low-level SPI interface
 class SX126x {
   public:
@@ -417,7 +420,7 @@
     uint8_t  GetRssiInst();
     void     GetRxBufferStatus(uint8_t *payloadLength, uint8_t *rxStartBufferPointer);
     void     Wakeup(void);
-    void     WaitForIdle(unsigned long timeout = 5000);
+    void     WaitForIdle(unsigned long timeout, const char *text, bool stop);
     uint8_t  ReadBuffer(uint8_t *rxData, uint8_t maxLen);
     void     WriteBuffer(uint8_t *txData, uint8_t txDataLen);
     void     WriteRegister(uint16_t reg, uint8_t* data, uint8_t numBytes, bool waitForBusy = true);
`

### Diff: Gateway vs WaterTankLevel

`diff
--- Gateway
+++ WaterTankLevel
@@ -1,5 +1,8 @@
 #ifndef _RA01S_H
 #define _RA01S_H
+
+#define SPI_Speed 500000
+#define ON_AIR_TIMEOUT 1000 //ms to send packet
 
 //return values
 #define ERR_NONE                        0
@@ -25,6 +28,7 @@
 #define XTAL_FREQ                       ( double )32000000
 #define FREQ_DIV                        ( double )pow( 2.0, 25.0 )
 #define FREQ_STEP                       ( double )( XTAL_FREQ / FREQ_DIV )
+#define BUSY_WAIT                       5000
 
 // SX126X Model
 #define SX1261_TRANCEIVER                             0x01
@@ -361,7 +365,6 @@
 #define SX126x_TXMODE_SYNC                            0x02
 #define SX126x_TXMODE_BACK2RX                         0x04
 
-
 // common low-level SPI interface
 class SX126x {
   public:
@@ -417,7 +420,7 @@
     uint8_t  GetRssiInst();
     void     GetRxBufferStatus(uint8_t *payloadLength, uint8_t *rxStartBufferPointer);
     void     Wakeup(void);
-    void     WaitForIdle(unsigned long timeout = 5000);
+    void     WaitForIdle(unsigned long timeout, const char *text, bool stop);
     uint8_t  ReadBuffer(uint8_t *rxData, uint8_t maxLen);
     void     WriteBuffer(uint8_t *txData, uint8_t txDataLen);
     void     WriteRegister(uint16_t reg, uint8_t* data, uint8_t numBytes, bool waitForBusy = true);
`

## Code Comparison: Ra01S.cpp

### Diff: Gateway vs Button

`diff
--- Gateway
+++ Button
@@ -1,7 +1,6 @@
 #include "Arduino.h"
 #include <SPI.h>
 #include "Ra01S.h"
-
 
 SX126x::SX126x(int spiSelect, int reset, int busy, int txen, int rxen)
 {
@@ -29,16 +28,16 @@
   //Serial.println("begin");
   //Serial.print("debugPrint=");
   //Serial.println(debugPrint);
-  //Serial.print("SX126x_SPI_SELECT=");
-  //Serial.println(SX126x_SPI_SELECT);
-  //Serial.print("SX126x_RESET=");
-  //Serial.println(SX126x_RESET);
-  //Serial.print("SX126x_BUSY=");
-  //Serial.println(SX126x_BUSY);
-  //Serial.print("SX126x_TXEN=");
-  //Serial.println(SX126x_TXEN);
-  //Serial.print("SX126x_RXEN=");
-  //Serial.println(SX126x_RXEN);
+  Serial.print("SX126x_SPI_SELECT=");
+  Serial.println(SX126x_SPI_SELECT);
+  Serial.print("SX126x_RESET=");
+  Serial.println(SX126x_RESET);
+  Serial.print("SX126x_BUSY=");
+  Serial.println(SX126x_BUSY);
+  Serial.print("SX126x_TXEN=");
+  Serial.println(SX126x_TXEN);
+  Serial.print("SX126x_RXEN=");
+  Serial.println(SX126x_RXEN);
   
   if ( txPowerInDbm > 22 )
     txPowerInDbm = 22;
@@ -46,24 +45,24 @@
     txPowerInDbm = -3;
   
   Reset();
-  //Serial.println("Reset");
+  Serial.println("Reset");
   
   uint8_t wk[2];
   ReadRegister(SX126X_REG_LORA_SYNC_WORD_MSB, wk, 2); // 0x0740
   uint16_t syncWord = (wk[0] << 8) + wk[1];
-  //Serial.print("syncWord=0x");
-  //Serial.println(syncWord, HEX);
+  Serial.print("syncWord=0x");
+  Serial.println(syncWord, HEX);
   if (syncWord != SX126X_SYNC_WORD_PUBLIC && syncWord != SX126X_SYNC_WORD_PRIVATE) {
-    //Serial.println("SX126x error, maybe no SPI connection");
+    Serial.println("SX126x error, maybe no SPI connection");
     return ERR_INVALID_MODE;
   }
 
-  //Serial.println("SX126x installed");
+  Serial.println("SX126x installed");
   SetStandby(SX126X_STANDBY_RC);
 
   SetDio2AsRfSwitchCtrl(true);
-  //Serial.print("tcxoVoltage=");
-  //Serial.println(tcxoVoltage);
+  Serial.print("tcxoVoltage=");
+  Serial.println(tcxoVoltage);
   // set TCXO control, if requested
   if(tcxoVoltage > 0.0) {
     SetDio3AsTcxoCtrl(tcxoVoltage, RADIO_TCXO_SETUP_TIME); // Configure the radio to use a TCXO controlled by DIO3
@@ -78,8 +77,8 @@
                   | SX126X_CALIBRATE_RC64K_ON
                   );
 
-  //Serial.print("useRegulatorLDO=");
-  //Serial.println(useRegulatorLDO);
+  Serial.print("useRegulatorLDO=");
+  Serial.println(useRegulatorLDO);
   if (useRegulatorLDO) {
     SetRegulatorMode(SX126X_REGULATOR_LDO); // set regulator mode: LDO
   } else {
@@ -212,7 +211,6 @@
   if ( txActive == false )
   {
     txActive = true;
-    SetStandby(SX126X_STANDBY_RC); // Ensure chip is in Standby before config
     PacketParams[2] = 0x00; //Variable length packet (explicit header)
     PacketParams[3] = len;
     WriteCommand(SX126X_CMD_SET_PACKET_PARAMS, PacketParams, 6); // 0x8C
@@ -221,7 +219,8 @@
     ClearIrqStatus(SX126X_IRQ_ALL);
 
     WriteBuffer(pData, len);
-    SetTx(0xFFFF); // ~1 second timeout
+    //SetTx(500);
+    SetTx(ON_AIR_TIMEOUT);
 
     if ( mode & SX126x_TXMODE_SYNC )
     {
@@ -231,7 +230,6 @@
         delay(1);
         irqStatus = GetIrqStatus();
       }
-
       if (debugPrint) {
         Serial.print("irqStatus=");
         Serial.println(irqStatus, HEX);
@@ -310,13 +308,14 @@
   digitalWrite(SX126x_RESET,1);
   delay(10);
   // ensure BUSY is low (state meachine ready)
-  WaitForIdle();
+  WaitForIdle(BUSY_WAIT, "Reset", true);
 }
 
 
 void SX126x::Wakeup(void)
 {
-  GetStatus();
+  //GetStatus();
+  digitalWrite(SX126x_SPI_SELECT, LOW); delay(1); digitalWrite(SX126x_SPI_SELECT, HIGH);
... (196 lines omitted)
`

### Diff: Gateway vs DualGateController

`diff
--- Gateway
+++ DualGateController
@@ -2,18 +2,16 @@
 #include <SPI.h>
 #include "Ra01S.h"
 
-
-SX126x::SX126x(int spiSelect, int reset, int busy, int txen, int rxen)
-{
+SX126x::SX126x(int spiSelect, int reset, int busy, int txen, int rxen) {
   SX126x_SPI_SELECT = spiSelect;
-  SX126x_RESET      = reset;
-  SX126x_BUSY       = busy;
-  SX126x_TXEN       = txen;
-  SX126x_RXEN       = rxen;
-  
-  txActive          = false;
-  debugPrint        = false;
-  
+  SX126x_RESET = reset;
+  SX126x_BUSY = busy;
+  SX126x_TXEN = txen;
+  SX126x_RXEN = rxen;
+
+  txActive = false;
+  debugPrint = false;
+
   pinMode(SX126x_SPI_SELECT, OUTPUT);
   pinMode(SX126x_RESET, OUTPUT);
   pinMode(SX126x_BUSY, INPUT);
@@ -24,8 +22,7 @@
 }
 
 
-int16_t SX126x::begin(uint32_t frequencyInHz, int8_t txPowerInDbm, float tcxoVoltage, bool useRegulatorLDO) 
-{
+int16_t SX126x::begin(uint32_t frequencyInHz, int8_t txPowerInDbm, float tcxoVoltage, bool useRegulatorLDO) {
   //Serial.println("begin");
   //Serial.print("debugPrint=");
   //Serial.println(debugPrint);
@@ -39,51 +36,50 @@
   //Serial.println(SX126x_TXEN);
   //Serial.print("SX126x_RXEN=");
   //Serial.println(SX126x_RXEN);
-  
-  if ( txPowerInDbm > 22 )
+
+  if (txPowerInDbm > 22)
     txPowerInDbm = 22;
-  if ( txPowerInDbm < -3 )
+  if (txPowerInDbm < -3)
     txPowerInDbm = -3;
-  
+
   Reset();
   //Serial.println("Reset");
-  
+
   uint8_t wk[2];
-  ReadRegister(SX126X_REG_LORA_SYNC_WORD_MSB, wk, 2); // 0x0740
+  ReadRegister(SX126X_REG_LORA_SYNC_WORD_MSB, wk, 2);  // 0x0740
   uint16_t syncWord = (wk[0] << 8) + wk[1];
   //Serial.print("syncWord=0x");
   //Serial.println(syncWord, HEX);
   if (syncWord != SX126X_SYNC_WORD_PUBLIC && syncWord != SX126X_SYNC_WORD_PRIVATE) {
-    //Serial.println("SX126x error, maybe no SPI connection");
+    Serial.println("SX126x error, maybe no SPI connection");
     return ERR_INVALID_MODE;
   }
 
   //Serial.println("SX126x installed");
-  SetStandby(SX126X_STANDBY_RC);
+  //SetStandby(SX126X_STANDBY_RC);
 
   SetDio2AsRfSwitchCtrl(true);
   //Serial.print("tcxoVoltage=");
   //Serial.println(tcxoVoltage);
   // set TCXO control, if requested
-  if(tcxoVoltage > 0.0) {
-    SetDio3AsTcxoCtrl(tcxoVoltage, RADIO_TCXO_SETUP_TIME); // Configure the radio to use a TCXO controlled by DIO3
-  }
-
-  Calibrate(  SX126X_CALIBRATE_IMAGE_ON
-                  | SX126X_CALIBRATE_ADC_BULK_P_ON
-                  | SX126X_CALIBRATE_ADC_BULK_N_ON
-                  | SX126X_CALIBRATE_ADC_PULSE_ON
-                  | SX126X_CALIBRATE_PLL_ON
-                  | SX126X_CALIBRATE_RC13M_ON
-                  | SX126X_CALIBRATE_RC64K_ON
-                  );
+  if (tcxoVoltage > 0.0) {
+    SetDio3AsTcxoCtrl(tcxoVoltage, RADIO_TCXO_SETUP_TIME);  // Configure the radio to use a TCXO controlled by DIO3
+  }
+
+  Calibrate(SX126X_CALIBRATE_IMAGE_ON
+            | SX126X_CALIBRATE_ADC_BULK_P_ON
+            | SX126X_CALIBRATE_ADC_BULK_N_ON
+            | SX126X_CALIBRATE_ADC_PULSE_ON
+            | SX126X_CALIBRATE_PLL_ON
+            | SX126X_CALIBRATE_RC13M_ON
+            | SX126X_CALIBRATE_RC64K_ON);
 
   //Serial.print("useRegulatorLDO=");
   //Serial.println(useRegulatorLDO);
   if (useRegulatorLDO) {
-    SetRegulatorMode(SX126X_REGULATOR_LDO); // set regulator mode: LDO
+    SetRegulatorMode(SX126X_REGULATOR_LDO);  // set regulator mode: LDO
   } else {
-    SetRegulatorMode(SX126X_REGULATOR_DC_DC); // set regulator mode: DC-DC
+    SetRegulatorMode(SX126X_REGULATOR_DC_DC);  // set regulator mode: DC-DC
   }
 
   SetBufferBaseAddress(0, 0);
@@ -95,15 +91,14 @@
   // SX1268_TRANCEIVER
   SetPaConfig(0x04, 0x07, 0x00, 0x01); // PA Optimal Settings +22 dBm
 #endif
-  SetPaConfig(0x04, 0x07, 0x00, 0x01); // PA Optimal Settings +22 dBm
-  SetOvercurrentProtection(60.0);  // current max 60mA for the whole device
-  SetPowerConfig(txPowerInDbm, SX126X_PA_RAMP_200U); //0 fuer Empfaenger
... (1105 lines omitted)
`

### Diff: Gateway vs DualPIR

`diff
--- Gateway
+++ DualPIR
@@ -1,7 +1,6 @@
 #include "Arduino.h"
 #include <SPI.h>
 #include "Ra01S.h"
-
 
 SX126x::SX126x(int spiSelect, int reset, int busy, int txen, int rxen)
 {
@@ -29,16 +28,16 @@
   //Serial.println("begin");
   //Serial.print("debugPrint=");
   //Serial.println(debugPrint);
-  //Serial.print("SX126x_SPI_SELECT=");
-  //Serial.println(SX126x_SPI_SELECT);
-  //Serial.print("SX126x_RESET=");
-  //Serial.println(SX126x_RESET);
-  //Serial.print("SX126x_BUSY=");
-  //Serial.println(SX126x_BUSY);
-  //Serial.print("SX126x_TXEN=");
-  //Serial.println(SX126x_TXEN);
-  //Serial.print("SX126x_RXEN=");
-  //Serial.println(SX126x_RXEN);
+  Serial.print("SX126x_SPI_SELECT=");
+  Serial.println(SX126x_SPI_SELECT);
+  Serial.print("SX126x_RESET=");
+  Serial.println(SX126x_RESET);
+  Serial.print("SX126x_BUSY=");
+  Serial.println(SX126x_BUSY);
+  Serial.print("SX126x_TXEN=");
+  Serial.println(SX126x_TXEN);
+  Serial.print("SX126x_RXEN=");
+  Serial.println(SX126x_RXEN);
   
   if ( txPowerInDbm > 22 )
     txPowerInDbm = 22;
@@ -46,24 +45,24 @@
     txPowerInDbm = -3;
   
   Reset();
-  //Serial.println("Reset");
+  Serial.println("Reset");
   
   uint8_t wk[2];
   ReadRegister(SX126X_REG_LORA_SYNC_WORD_MSB, wk, 2); // 0x0740
   uint16_t syncWord = (wk[0] << 8) + wk[1];
-  //Serial.print("syncWord=0x");
-  //Serial.println(syncWord, HEX);
+  Serial.print("syncWord=0x");
+  Serial.println(syncWord, HEX);
   if (syncWord != SX126X_SYNC_WORD_PUBLIC && syncWord != SX126X_SYNC_WORD_PRIVATE) {
-    //Serial.println("SX126x error, maybe no SPI connection");
+    Serial.println("SX126x error, maybe no SPI connection");
     return ERR_INVALID_MODE;
   }
 
-  //Serial.println("SX126x installed");
+  Serial.println("SX126x installed");
   SetStandby(SX126X_STANDBY_RC);
 
   SetDio2AsRfSwitchCtrl(true);
-  //Serial.print("tcxoVoltage=");
-  //Serial.println(tcxoVoltage);
+  Serial.print("tcxoVoltage=");
+  Serial.println(tcxoVoltage);
   // set TCXO control, if requested
   if(tcxoVoltage > 0.0) {
     SetDio3AsTcxoCtrl(tcxoVoltage, RADIO_TCXO_SETUP_TIME); // Configure the radio to use a TCXO controlled by DIO3
@@ -78,8 +77,8 @@
                   | SX126X_CALIBRATE_RC64K_ON
                   );
 
-  //Serial.print("useRegulatorLDO=");
-  //Serial.println(useRegulatorLDO);
+  Serial.print("useRegulatorLDO=");
+  Serial.println(useRegulatorLDO);
   if (useRegulatorLDO) {
     SetRegulatorMode(SX126X_REGULATOR_LDO); // set regulator mode: LDO
   } else {
@@ -108,8 +107,8 @@
   // see SX1262/SX1268 datasheet, chapter 15 Known Limitations, section 15.4 for details
   // When exchanging LoRa packets with inverted IQ polarity, some packet losses may be observed for longer packets.
   // Workaround: Bit 2 at address 0x0736 must be set to:
-  // ��0�� when using inverted IQ polarity (see the SetPacketParam(...) command)
-  // ��1�� when using standard IQ polarity
+  // 0 when using inverted IQ polarity (see the SetPacketParam(...) command)
+  // 1 when using standard IQ polarity
 
   // read current IQ configuration
   uint8_t iqConfigCurrent = 0;
@@ -212,7 +211,6 @@
   if ( txActive == false )
   {
     txActive = true;
-    SetStandby(SX126X_STANDBY_RC); // Ensure chip is in Standby before config
     PacketParams[2] = 0x00; //Variable length packet (explicit header)
     PacketParams[3] = len;
     WriteCommand(SX126X_CMD_SET_PACKET_PARAMS, PacketParams, 6); // 0x8C
@@ -221,7 +219,8 @@
     ClearIrqStatus(SX126X_IRQ_ALL);
 
     WriteBuffer(pData, len);
-    SetTx(0xFFFF); // ~1 second timeout
+    //SetTx(500);
+    SetTx(ON_AIR_TIMEOUT);
 
     if ( mode & SX126x_TXMODE_SYNC )
     {
@@ -231,7 +230,6 @@
         delay(1);
         irqStatus = GetIrqStatus();
       }
-
       if (debugPrint) {
         Serial.print("irqStatus=");
         Serial.println(irqStatus, HEX);
@@ -310,13 +308,14 @@
   digitalWrite(SX126x_RESET,1);
   delay(10);
... (225 lines omitted)
`

### Diff: Gateway vs Repeater

`diff
--- Gateway
+++ Repeater
@@ -1,7 +1,6 @@
 #include "Arduino.h"
 #include <SPI.h>
 #include "Ra01S.h"
-
 
 SX126x::SX126x(int spiSelect, int reset, int busy, int txen, int rxen)
 {
@@ -29,16 +28,16 @@
   //Serial.println("begin");
   //Serial.print("debugPrint=");
   //Serial.println(debugPrint);
-  //Serial.print("SX126x_SPI_SELECT=");
-  //Serial.println(SX126x_SPI_SELECT);
-  //Serial.print("SX126x_RESET=");
-  //Serial.println(SX126x_RESET);
-  //Serial.print("SX126x_BUSY=");
-  //Serial.println(SX126x_BUSY);
-  //Serial.print("SX126x_TXEN=");
-  //Serial.println(SX126x_TXEN);
-  //Serial.print("SX126x_RXEN=");
-  //Serial.println(SX126x_RXEN);
+  Serial.print("SX126x_SPI_SELECT=");
+  Serial.println(SX126x_SPI_SELECT);
+  Serial.print("SX126x_RESET=");
+  Serial.println(SX126x_RESET);
+  Serial.print("SX126x_BUSY=");
+  Serial.println(SX126x_BUSY);
+  Serial.print("SX126x_TXEN=");
+  Serial.println(SX126x_TXEN);
+  Serial.print("SX126x_RXEN=");
+  Serial.println(SX126x_RXEN);
   
   if ( txPowerInDbm > 22 )
     txPowerInDbm = 22;
@@ -46,24 +45,24 @@
     txPowerInDbm = -3;
   
   Reset();
-  //Serial.println("Reset");
+  Serial.println("Reset");
   
   uint8_t wk[2];
   ReadRegister(SX126X_REG_LORA_SYNC_WORD_MSB, wk, 2); // 0x0740
   uint16_t syncWord = (wk[0] << 8) + wk[1];
-  //Serial.print("syncWord=0x");
-  //Serial.println(syncWord, HEX);
+  Serial.print("syncWord=0x");
+  Serial.println(syncWord, HEX);
   if (syncWord != SX126X_SYNC_WORD_PUBLIC && syncWord != SX126X_SYNC_WORD_PRIVATE) {
-    //Serial.println("SX126x error, maybe no SPI connection");
+    Serial.println("SX126x error, maybe no SPI connection");
     return ERR_INVALID_MODE;
   }
 
-  //Serial.println("SX126x installed");
+  Serial.println("SX126x installed");
   SetStandby(SX126X_STANDBY_RC);
 
   SetDio2AsRfSwitchCtrl(true);
-  //Serial.print("tcxoVoltage=");
-  //Serial.println(tcxoVoltage);
+  Serial.print("tcxoVoltage=");
+  Serial.println(tcxoVoltage);
   // set TCXO control, if requested
   if(tcxoVoltage > 0.0) {
     SetDio3AsTcxoCtrl(tcxoVoltage, RADIO_TCXO_SETUP_TIME); // Configure the radio to use a TCXO controlled by DIO3
@@ -78,8 +77,8 @@
                   | SX126X_CALIBRATE_RC64K_ON
                   );
 
-  //Serial.print("useRegulatorLDO=");
-  //Serial.println(useRegulatorLDO);
+  Serial.print("useRegulatorLDO=");
+  Serial.println(useRegulatorLDO);
   if (useRegulatorLDO) {
     SetRegulatorMode(SX126X_REGULATOR_LDO); // set regulator mode: LDO
   } else {
@@ -212,7 +211,6 @@
   if ( txActive == false )
   {
     txActive = true;
-    SetStandby(SX126X_STANDBY_RC); // Ensure chip is in Standby before config
     PacketParams[2] = 0x00; //Variable length packet (explicit header)
     PacketParams[3] = len;
     WriteCommand(SX126X_CMD_SET_PACKET_PARAMS, PacketParams, 6); // 0x8C
@@ -221,7 +219,8 @@
     ClearIrqStatus(SX126X_IRQ_ALL);
 
     WriteBuffer(pData, len);
-    SetTx(0xFFFF); // ~1 second timeout
+    //SetTx(500);
+    SetTx(ON_AIR_TIMEOUT);
 
     if ( mode & SX126x_TXMODE_SYNC )
     {
@@ -231,7 +230,6 @@
         delay(1);
         irqStatus = GetIrqStatus();
       }
-
       if (debugPrint) {
         Serial.print("irqStatus=");
         Serial.println(irqStatus, HEX);
@@ -310,13 +308,14 @@
   digitalWrite(SX126x_RESET,1);
   delay(10);
   // ensure BUSY is low (state meachine ready)
-  WaitForIdle();
+  WaitForIdle(BUSY_WAIT, "Reset", true);
 }
 
 
 void SX126x::Wakeup(void)
 {
-  GetStatus();
+  //GetStatus();
+  digitalWrite(SX126x_SPI_SELECT, LOW); delay(1); digitalWrite(SX126x_SPI_SELECT, HIGH);
... (196 lines omitted)
`

### Diff: Gateway vs Victron

`diff
--- Gateway
+++ Victron
@@ -1,7 +1,6 @@
 #include "Arduino.h"
 #include <SPI.h>
 #include "Ra01S.h"
-
 
 SX126x::SX126x(int spiSelect, int reset, int busy, int txen, int rxen)
 {
@@ -29,16 +28,16 @@
   //Serial.println("begin");
   //Serial.print("debugPrint=");
   //Serial.println(debugPrint);
-  //Serial.print("SX126x_SPI_SELECT=");
-  //Serial.println(SX126x_SPI_SELECT);
-  //Serial.print("SX126x_RESET=");
-  //Serial.println(SX126x_RESET);
-  //Serial.print("SX126x_BUSY=");
-  //Serial.println(SX126x_BUSY);
-  //Serial.print("SX126x_TXEN=");
-  //Serial.println(SX126x_TXEN);
-  //Serial.print("SX126x_RXEN=");
-  //Serial.println(SX126x_RXEN);
+  Serial.print("SX126x_SPI_SELECT=");
+  Serial.println(SX126x_SPI_SELECT);
+  Serial.print("SX126x_RESET=");
+  Serial.println(SX126x_RESET);
+  Serial.print("SX126x_BUSY=");
+  Serial.println(SX126x_BUSY);
+  Serial.print("SX126x_TXEN=");
+  Serial.println(SX126x_TXEN);
+  Serial.print("SX126x_RXEN=");
+  Serial.println(SX126x_RXEN);
   
   if ( txPowerInDbm > 22 )
     txPowerInDbm = 22;
@@ -46,24 +45,24 @@
     txPowerInDbm = -3;
   
   Reset();
-  //Serial.println("Reset");
+  Serial.println("Reset");
   
   uint8_t wk[2];
   ReadRegister(SX126X_REG_LORA_SYNC_WORD_MSB, wk, 2); // 0x0740
   uint16_t syncWord = (wk[0] << 8) + wk[1];
-  //Serial.print("syncWord=0x");
-  //Serial.println(syncWord, HEX);
+  Serial.print("syncWord=0x");
+  Serial.println(syncWord, HEX);
   if (syncWord != SX126X_SYNC_WORD_PUBLIC && syncWord != SX126X_SYNC_WORD_PRIVATE) {
-    //Serial.println("SX126x error, maybe no SPI connection");
+    Serial.println("SX126x error, maybe no SPI connection");
     return ERR_INVALID_MODE;
   }
 
-  //Serial.println("SX126x installed");
+  Serial.println("SX126x installed");
   SetStandby(SX126X_STANDBY_RC);
 
   SetDio2AsRfSwitchCtrl(true);
-  //Serial.print("tcxoVoltage=");
-  //Serial.println(tcxoVoltage);
+  Serial.print("tcxoVoltage=");
+  Serial.println(tcxoVoltage);
   // set TCXO control, if requested
   if(tcxoVoltage > 0.0) {
     SetDio3AsTcxoCtrl(tcxoVoltage, RADIO_TCXO_SETUP_TIME); // Configure the radio to use a TCXO controlled by DIO3
@@ -78,8 +77,8 @@
                   | SX126X_CALIBRATE_RC64K_ON
                   );
 
-  //Serial.print("useRegulatorLDO=");
-  //Serial.println(useRegulatorLDO);
+  Serial.print("useRegulatorLDO=");
+  Serial.println(useRegulatorLDO);
   if (useRegulatorLDO) {
     SetRegulatorMode(SX126X_REGULATOR_LDO); // set regulator mode: LDO
   } else {
@@ -212,7 +211,6 @@
   if ( txActive == false )
   {
     txActive = true;
-    SetStandby(SX126X_STANDBY_RC); // Ensure chip is in Standby before config
     PacketParams[2] = 0x00; //Variable length packet (explicit header)
     PacketParams[3] = len;
     WriteCommand(SX126X_CMD_SET_PACKET_PARAMS, PacketParams, 6); // 0x8C
@@ -221,7 +219,8 @@
     ClearIrqStatus(SX126X_IRQ_ALL);
 
     WriteBuffer(pData, len);
-    SetTx(0xFFFF); // ~1 second timeout
+    //SetTx(500);
+    SetTx(ON_AIR_TIMEOUT);
 
     if ( mode & SX126x_TXMODE_SYNC )
     {
@@ -231,7 +230,6 @@
         delay(1);
         irqStatus = GetIrqStatus();
       }
-
       if (debugPrint) {
         Serial.print("irqStatus=");
         Serial.println(irqStatus, HEX);
@@ -310,13 +308,14 @@
   digitalWrite(SX126x_RESET,1);
   delay(10);
   // ensure BUSY is low (state meachine ready)
-  WaitForIdle();
+  WaitForIdle(BUSY_WAIT, "Reset", true);
 }
 
 
 void SX126x::Wakeup(void)
 {
-  GetStatus();
+  //GetStatus();
+  digitalWrite(SX126x_SPI_SELECT, LOW); delay(1); digitalWrite(SX126x_SPI_SELECT, HIGH);
... (204 lines omitted)
`

### Diff: Gateway vs WaterTankLevel

`diff
--- Gateway
+++ WaterTankLevel
@@ -1,7 +1,6 @@
 #include "Arduino.h"
 #include <SPI.h>
 #include "Ra01S.h"
-
 
 SX126x::SX126x(int spiSelect, int reset, int busy, int txen, int rxen)
 {
@@ -29,16 +28,16 @@
   //Serial.println("begin");
   //Serial.print("debugPrint=");
   //Serial.println(debugPrint);
-  //Serial.print("SX126x_SPI_SELECT=");
-  //Serial.println(SX126x_SPI_SELECT);
-  //Serial.print("SX126x_RESET=");
-  //Serial.println(SX126x_RESET);
-  //Serial.print("SX126x_BUSY=");
-  //Serial.println(SX126x_BUSY);
-  //Serial.print("SX126x_TXEN=");
-  //Serial.println(SX126x_TXEN);
-  //Serial.print("SX126x_RXEN=");
-  //Serial.println(SX126x_RXEN);
+  Serial.print("SX126x_SPI_SELECT=");
+  Serial.println(SX126x_SPI_SELECT);
+  Serial.print("SX126x_RESET=");
+  Serial.println(SX126x_RESET);
+  Serial.print("SX126x_BUSY=");
+  Serial.println(SX126x_BUSY);
+  Serial.print("SX126x_TXEN=");
+  Serial.println(SX126x_TXEN);
+  Serial.print("SX126x_RXEN=");
+  Serial.println(SX126x_RXEN);
   
   if ( txPowerInDbm > 22 )
     txPowerInDbm = 22;
@@ -46,24 +45,24 @@
     txPowerInDbm = -3;
   
   Reset();
-  //Serial.println("Reset");
+  Serial.println("Reset");
   
   uint8_t wk[2];
   ReadRegister(SX126X_REG_LORA_SYNC_WORD_MSB, wk, 2); // 0x0740
   uint16_t syncWord = (wk[0] << 8) + wk[1];
-  //Serial.print("syncWord=0x");
-  //Serial.println(syncWord, HEX);
+  Serial.print("syncWord=0x");
+  Serial.println(syncWord, HEX);
   if (syncWord != SX126X_SYNC_WORD_PUBLIC && syncWord != SX126X_SYNC_WORD_PRIVATE) {
-    //Serial.println("SX126x error, maybe no SPI connection");
+    Serial.println("SX126x error, maybe no SPI connection");
     return ERR_INVALID_MODE;
   }
 
-  //Serial.println("SX126x installed");
+  Serial.println("SX126x installed");
   SetStandby(SX126X_STANDBY_RC);
 
   SetDio2AsRfSwitchCtrl(true);
-  //Serial.print("tcxoVoltage=");
-  //Serial.println(tcxoVoltage);
+  Serial.print("tcxoVoltage=");
+  Serial.println(tcxoVoltage);
   // set TCXO control, if requested
   if(tcxoVoltage > 0.0) {
     SetDio3AsTcxoCtrl(tcxoVoltage, RADIO_TCXO_SETUP_TIME); // Configure the radio to use a TCXO controlled by DIO3
@@ -78,8 +77,8 @@
                   | SX126X_CALIBRATE_RC64K_ON
                   );
 
-  //Serial.print("useRegulatorLDO=");
-  //Serial.println(useRegulatorLDO);
+  Serial.print("useRegulatorLDO=");
+  Serial.println(useRegulatorLDO);
   if (useRegulatorLDO) {
     SetRegulatorMode(SX126X_REGULATOR_LDO); // set regulator mode: LDO
   } else {
@@ -212,7 +211,6 @@
   if ( txActive == false )
   {
     txActive = true;
-    SetStandby(SX126X_STANDBY_RC); // Ensure chip is in Standby before config
     PacketParams[2] = 0x00; //Variable length packet (explicit header)
     PacketParams[3] = len;
     WriteCommand(SX126X_CMD_SET_PACKET_PARAMS, PacketParams, 6); // 0x8C
@@ -221,7 +219,8 @@
     ClearIrqStatus(SX126X_IRQ_ALL);
 
     WriteBuffer(pData, len);
-    SetTx(0xFFFF); // ~1 second timeout
+    //SetTx(500);
+    SetTx(ON_AIR_TIMEOUT);
 
     if ( mode & SX126x_TXMODE_SYNC )
     {
@@ -231,7 +230,6 @@
         delay(1);
         irqStatus = GetIrqStatus();
       }
-
       if (debugPrint) {
         Serial.print("irqStatus=");
         Serial.println(irqStatus, HEX);
@@ -310,13 +308,14 @@
   digitalWrite(SX126x_RESET,1);
   delay(10);
   // ensure BUSY is low (state meachine ready)
-  WaitForIdle();
+  WaitForIdle(BUSY_WAIT, "Reset", true);
 }
 
 
 void SX126x::Wakeup(void)
 {
-  GetStatus();
+  //GetStatus();
+  digitalWrite(SX126x_SPI_SELECT, LOW); delay(1); digitalWrite(SX126x_SPI_SELECT, HIGH);
... (204 lines omitted)
`

## Code Comparison: LoRaHelper.h

### Diff: Gateway vs Button

`diff
--- Gateway
+++ Button
@@ -1,31 +1,22 @@
-#ifndef LoRaHelper_H
-#define LoRaHelper_H
 #include <Ra01S.h>
+#include "Pinout.h"
 
 #define RF_FREQUENCY 915000000    // Hz  center frequency
 #define TX_OUTPUT_POWER 22        // dBm tx output power
 #define LORA_BANDWIDTH 6          // bandwidth \
                                   // 2: 31.25Khz \
                                   // 3: 62.5Khz \
-                                  // 4: 125Khz \
-                                  // 5: 250KHZ \
-                                  // 6: 500Khz
-#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF12]
+                                  // 4: 125Khz   SF9 \
+                                  // 5: 250KHZ   SF10 \
+                                  // 6: 500Khz   SF11
+#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF11]
 #define LORA_CODINGRATE 1         // [1: 4/5, \
                                   //  2: 4/6, \
                                   //  3: 4/7, \
                                   //  4: 4/8]
 
 #define LORA_PREAMBLE_LENGTH 8  // Same for Tx and Rx
+#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header) \
+                                // 1..255  Fixed length packet (implicit header)
+  
 
-#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header) \
-
-                                // 1..255  Fixed length packet (implicit header)
-
-
-
-extern SX126x lora;
-
-
-
-#endif
`

### Diff: Gateway vs DualGateController

`diff
--- Gateway
+++ DualGateController
@@ -1,31 +1,26 @@
-#ifndef LoRaHelper_H
-#define LoRaHelper_H
-#include <Ra01S.h>
+#ifndef LORAHELPER_H
+#define LORAHELPER_H
+
+#include "Ra01S.h"
+#include "MyPins.h"
 
 #define RF_FREQUENCY 915000000    // Hz  center frequency
 #define TX_OUTPUT_POWER 22        // dBm tx output power
-#define LORA_BANDWIDTH 6          // bandwidth \
-                                  // 2: 31.25Khz \
-                                  // 3: 62.5Khz \
-                                  // 4: 125Khz \
-                                  // 5: 250KHZ \
-                                  // 6: 500Khz
-#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF12]
-#define LORA_CODINGRATE 1         // [1: 4/5, \
-                                  //  2: 4/6, \
-                                  //  3: 4/7, \
+#define LORA_BANDWIDTH 6          // bandwidth
+                                  // 2: 31.25Khz
+                                  // 3: 62.5Khz
+                                  // 4: 125Khz   SF9
+                                  // 5: 250KHZ   SF10
+                                  // 6: 500Khz   SF11
+#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF11]
+#define LORA_CODINGRATE 1         // [1: 4/5,
+                                  //  2: 4/6,
+                                  //  3: 4/7,
                                   //  4: 4/8]
 
 #define LORA_PREAMBLE_LENGTH 8  // Same for Tx and Rx
+#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header)
+                                // 1..255  Fixed length packet (implicit header)
+  
+#endif
 
-#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header) \
-
-                                // 1..255  Fixed length packet (implicit header)
-
-
-
-extern SX126x lora;
-
-
-
-#endif
`

### Diff: Gateway vs DualPIR

`diff
--- Gateway
+++ DualPIR
@@ -1,31 +1,20 @@
-#ifndef LoRaHelper_H
-#define LoRaHelper_H
 #include <Ra01S.h>
+#include "Pinout.h"
 
 #define RF_FREQUENCY 915000000    // Hz  center frequency
 #define TX_OUTPUT_POWER 22        // dBm tx output power
-#define LORA_BANDWIDTH 6          // bandwidth \
-                                  // 2: 31.25Khz \
-                                  // 3: 62.5Khz \
-                                  // 4: 125Khz \
-                                  // 5: 250KHZ \
-                                  // 6: 500Khz
-#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF12]
-#define LORA_CODINGRATE 1         // [1: 4/5, \
-                                  //  2: 4/6, \
-                                  //  3: 4/7, \
+#define LORA_BANDWIDTH 6          // bandwidth
+                                  // 2: 31.25Khz
+                                  // 3: 62.5Khz
+                                  // 4: 125Khz   SF9
+                                  // 5: 250KHZ   SF10
+                                  // 6: 500Khz   SF11
+#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF11]
+#define LORA_CODINGRATE 1         // [1: 4/5,
+                                  //  2: 4/6,
+                                  //  3: 4/7,
                                   //  4: 4/8]
 
 #define LORA_PREAMBLE_LENGTH 8  // Same for Tx and Rx
-
-#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header) \
-
+#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header)
                                 // 1..255  Fixed length packet (implicit header)
-
-
-
-extern SX126x lora;
-
-
-
-#endif
`

### Diff: Gateway vs Repeater

`diff
--- Gateway
+++ Repeater
@@ -1,31 +1,22 @@
-#ifndef LoRaHelper_H
-#define LoRaHelper_H
-#include <Ra01S.h>
+#include "Ra01S.h"
+#include "Pinout.h"
 
 #define RF_FREQUENCY 915000000    // Hz  center frequency
 #define TX_OUTPUT_POWER 22        // dBm tx output power
-#define LORA_BANDWIDTH 6          // bandwidth \
-                                  // 2: 31.25Khz \
-                                  // 3: 62.5Khz \
-                                  // 4: 125Khz \
-                                  // 5: 250KHZ \
-                                  // 6: 500Khz
-#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF12]
-#define LORA_CODINGRATE 1         // [1: 4/5, \
-                                  //  2: 4/6, \
-                                  //  3: 4/7, \
+#define LORA_BANDWIDTH 6          // bandwidth
+                                  // 2: 31.25Khz
+                                  // 3: 62.5Khz
+                                  // 4: 125Khz   SF9
+                                  // 5: 250KHZ   SF10
+                                  // 6: 500Khz   SF11
+#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF11]
+#define LORA_CODINGRATE 1         // [1: 4/5,
+                                  //  2: 4/6,
+                                  //  3: 4/7,
                                   //  4: 4/8]
 
 #define LORA_PREAMBLE_LENGTH 8  // Same for Tx and Rx
+#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header)
+                                // 1..255  Fixed length packet (implicit header)
+  
 
-#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header) \
-
-                                // 1..255  Fixed length packet (implicit header)
-
-
-
-extern SX126x lora;
-
-
-
-#endif
`

### Diff: Gateway vs Victron

`diff
--- Gateway
+++ Victron
@@ -1,31 +1,22 @@
-#ifndef LoRaHelper_H
-#define LoRaHelper_H
 #include <Ra01S.h>
+#include "Pinout.h"
 
 #define RF_FREQUENCY 915000000    // Hz  center frequency
 #define TX_OUTPUT_POWER 22        // dBm tx output power
-#define LORA_BANDWIDTH 6          // bandwidth \
-                                  // 2: 31.25Khz \
-                                  // 3: 62.5Khz \
-                                  // 4: 125Khz \
-                                  // 5: 250KHZ \
-                                  // 6: 500Khz
-#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF12]
-#define LORA_CODINGRATE 1         // [1: 4/5, \
-                                  //  2: 4/6, \
-                                  //  3: 4/7, \
+#define LORA_BANDWIDTH 6          // bandwidth 
+                                  // 2: 31.25Khz 
+                                  // 3: 62.5Khz 
+                                  // 4: 125Khz   SF9 
+                                  // 5: 250KHZ   SF10 
+                                  // 6: 500Khz   SF11
+#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF11]
+#define LORA_CODINGRATE 1         // [1: 4/5, 
+                                  //  2: 4/6, 
+                                  //  3: 4/7, 
                                   //  4: 4/8]
 
 #define LORA_PREAMBLE_LENGTH 8  // Same for Tx and Rx
+#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header) 
+                                // 1..255  Fixed length packet (implicit header)
+  
 
-#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header) \
-
-                                // 1..255  Fixed length packet (implicit header)
-
-
-
-extern SX126x lora;
-
-
-
-#endif
`

### Diff: Gateway vs WaterTankLevel

`diff
--- Gateway
+++ WaterTankLevel
@@ -1,31 +1,20 @@
-#ifndef LoRaHelper_H
-#define LoRaHelper_H
 #include <Ra01S.h>
+#include "Pinout.h"
 
 #define RF_FREQUENCY 915000000    // Hz  center frequency
 #define TX_OUTPUT_POWER 22        // dBm tx output power
-#define LORA_BANDWIDTH 6          // bandwidth \
-                                  // 2: 31.25Khz \
-                                  // 3: 62.5Khz \
-                                  // 4: 125Khz \
-                                  // 5: 250KHZ \
-                                  // 6: 500Khz
-#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF12]
-#define LORA_CODINGRATE 1         // [1: 4/5, \
-                                  //  2: 4/6, \
-                                  //  3: 4/7, \
+#define LORA_BANDWIDTH 6          // bandwidth
+                                  // 2: 31.25Khz
+                                  // 3: 62.5Khz
+                                  // 4: 125Khz   SF9
+                                  // 5: 250KHZ   SF10
+                                  // 6: 500Khz   SF11
+#define LORA_SPREADING_FACTOR 11  // spreading factor [SF5..SF11]
+#define LORA_CODINGRATE 1         // [1: 4/5,
+                                  //  2: 4/6,
+                                  //  3: 4/7,
                                   //  4: 4/8]
 
 #define LORA_PREAMBLE_LENGTH 8  // Same for Tx and Rx
-
-#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header) \
-
+#define LORA_PAYLOADLENGTH 0    // 0: Variable length packet (explicit header)
                                 // 1..255  Fixed length packet (implicit header)
-
-
-
-extern SX126x lora;
-
-
-
-#endif
`

## Code Comparison: LoraMsg.h

### Diff: Gateway vs Button

`diff
--- Gateway
+++ Button
@@ -24,16 +24,15 @@
   PortValue getPortValue(int index);
   uint8_t numberOfPortValues();
   void printMessage();
-  uint8_t getFromByte(const uint8_t byteNumber);
   void encryptMessage();
   void decryptMessage();
   bool isForMe(const uint8_t* address);
+  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 
 private:
   void addAddress(const uint8_t* address);
   //void addEndMarker();
   uint8_t toAddress[6];     // Store toAddress for encryption/decryption
-  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 };
 
 #endif
`

### Diff: Gateway vs DualGateController

`diff
--- Gateway
+++ DualGateController
@@ -9,7 +9,7 @@
 
 class LoraMsg {
 private:
-  static const int MAX_MSG_SIZE = 128;
+  static const int MAX_MSG_SIZE = 256;
   //static const uint8_t END_MARKER[4];
   uint8_t message[MAX_MSG_SIZE] = { 0 };
   int currentIndex;
@@ -24,16 +24,16 @@
   PortValue getPortValue(int index);
   uint8_t numberOfPortValues();
   void printMessage();
-  uint8_t getFromByte(const uint8_t byteNumber);
   void encryptMessage();
   void decryptMessage();
   bool isForMe(const uint8_t* address);
+  void getFromAddress(uint8_t* address);
+  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 
 private:
   void addAddress(const uint8_t* address);
   //void addEndMarker();
   uint8_t toAddress[6];     // Store toAddress for encryption/decryption
-  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 };
 
 #endif
`

### Diff: Gateway vs DualPIR

`diff
--- Gateway
+++ DualPIR
@@ -24,16 +24,15 @@
   PortValue getPortValue(int index);
   uint8_t numberOfPortValues();
   void printMessage();
-  uint8_t getFromByte(const uint8_t byteNumber);
   void encryptMessage();
   void decryptMessage();
   bool isForMe(const uint8_t* address);
+  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 
 private:
   void addAddress(const uint8_t* address);
   //void addEndMarker();
   uint8_t toAddress[6];     // Store toAddress for encryption/decryption
-  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 };
 
 #endif
`

### Diff: Gateway vs Repeater

`diff
--- Gateway
+++ Repeater
@@ -9,7 +9,7 @@
 
 class LoraMsg {
 private:
-  static const int MAX_MSG_SIZE = 128;
+  static const int MAX_MSG_SIZE = 256;
   //static const uint8_t END_MARKER[4];
   uint8_t message[MAX_MSG_SIZE] = { 0 };
   int currentIndex;
@@ -19,21 +19,21 @@
   LoraMsg(const uint8_t* toAddress, const uint8_t* fromAddress);
   LoraMsg(const uint8_t* encryptedMessage, byte sizeOfMsg);
   bool addPortValue(const PortValue& portValue);
+  bool setPortValue(const char type[2], uint16_t newValue);
   uint8_t* getMessage();
   uint8_t getMessageLength();
   PortValue getPortValue(int index);
   uint8_t numberOfPortValues();
   void printMessage();
-  uint8_t getFromByte(const uint8_t byteNumber);
   void encryptMessage();
   void decryptMessage();
   bool isForMe(const uint8_t* address);
+  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 
 private:
   void addAddress(const uint8_t* address);
   //void addEndMarker();
   uint8_t toAddress[6];     // Store toAddress for encryption/decryption
-  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 };
 
 #endif
`

### Diff: Gateway vs Victron

`diff
--- Gateway
+++ Victron
@@ -24,16 +24,15 @@
   PortValue getPortValue(int index);
   uint8_t numberOfPortValues();
   void printMessage();
-  uint8_t getFromByte(const uint8_t byteNumber);
   void encryptMessage();
   void decryptMessage();
   bool isForMe(const uint8_t* address);
+  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 
 private:
   void addAddress(const uint8_t* address);
   //void addEndMarker();
   uint8_t toAddress[6];     // Store toAddress for encryption/decryption
-  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 };
 
 #endif
`

### Diff: Gateway vs WaterTankLevel

`diff
--- Gateway
+++ WaterTankLevel
@@ -24,16 +24,15 @@
   PortValue getPortValue(int index);
   uint8_t numberOfPortValues();
   void printMessage();
-  uint8_t getFromByte(const uint8_t byteNumber);
   void encryptMessage();
   void decryptMessage();
   bool isForMe(const uint8_t* address);
+  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 
 private:
   void addAddress(const uint8_t* address);
   //void addEndMarker();
   uint8_t toAddress[6];     // Store toAddress for encryption/decryption
-  uint16_t getMessageID();  // Retrieve the message ID for encryption/decryption
 };
 
 #endif
`

## Code Comparison: LoraMsg.cpp

### Diff: Gateway vs Button

`diff
--- Gateway
+++ Button
@@ -20,26 +20,6 @@
   }
   currentIndex = sizeOfMsg;
   memcpy(this->toAddress, encryptedMessage, 6);  // Extract toAddress for decryption
-
-  /* Debug output to check currentIndex and message content
-  Serial.print("Encrypted message received. Current index set to: ");
-  Serial.println(currentIndex);
-  Serial.print("Message content: ");
-  for (int i = 0; i < currentIndex; i++) {
-    if (message[i] < 0x10) Serial.print("0");
-    Serial.print(message[i], HEX);
-    Serial.print(" ");
-  }
-  Serial.println();
-  */
-}
-
-uint8_t LoraMsg::getFromByte(const uint8_t byteNumber) {
-  if (byteNumber < 6) {
-    return message[6 + byteNumber];
-  } else {
-    return 0;
-  }
 }
 
 // Add PortValue to the message
@@ -63,7 +43,6 @@
 PortValue LoraMsg::getPortValue(int index) {
   PortValue portValue;
   int startIndex = 12 + index * 4;      // Start after addresses (12 bytes total)
-  //Serial.printf("\nGetPortValue index:%d location:%d currentIndex:%d\n",index,startIndex,currentIndex);
   if (startIndex + 3 < currentIndex) {  // Exclude the end marker #####removed -4 as initially there is no marker
     portValue.type[0] = message[startIndex];
     portValue.type[1] = message[startIndex + 1];
@@ -96,6 +75,10 @@
 
 // Encrypt the message starting from the fromAddress (6th byte)
 void LoraMsg::encryptMessage() {
+  Serial.print("\nCurIndex:");
+  Serial.println(currentIndex);
+  delay(100);
+  
   uint16_t messageID = getMessageID();
   // Encrypt fromAddress and beyond
   for (int i = 6; i < 12; i++) {
@@ -134,6 +117,15 @@
 
 // Print the message in a detailed format
 void LoraMsg::printMessage() {
+
+  //Serial.print("\nPV Method content: ");
+  //for (int i = 0; i < 128; i++) {
+  //  if (message[i] < 0x10) Serial.print("0");
+  //  Serial.print(message[i], HEX);
+  //  if (i == 11 || (i > 11 && (i - 11) % 4 == 0)) Serial.print(" ");
+  //}
+  //Serial.printf("\n currentIndex:%d\n", currentIndex);
+
   Serial.print("To: ");
   for (int i = 0; i < 6; i++) {
     if (message[i] < 0x10) Serial.print("0");  // Ensure two digits for each byte
`

### Diff: Gateway vs DualGateController

`diff
--- Gateway
+++ DualGateController
@@ -20,26 +20,6 @@
   }
   currentIndex = sizeOfMsg;
   memcpy(this->toAddress, encryptedMessage, 6);  // Extract toAddress for decryption
-
-  /* Debug output to check currentIndex and message content
-  Serial.print("Encrypted message received. Current index set to: ");
-  Serial.println(currentIndex);
-  Serial.print("Message content: ");
-  for (int i = 0; i < currentIndex; i++) {
-    if (message[i] < 0x10) Serial.print("0");
-    Serial.print(message[i], HEX);
-    Serial.print(" ");
-  }
-  Serial.println();
-  */
-}
-
-uint8_t LoraMsg::getFromByte(const uint8_t byteNumber) {
-  if (byteNumber < 6) {
-    return message[6 + byteNumber];
-  } else {
-    return 0;
-  }
 }
 
 // Add PortValue to the message
@@ -63,7 +43,6 @@
 PortValue LoraMsg::getPortValue(int index) {
   PortValue portValue;
   int startIndex = 12 + index * 4;      // Start after addresses (12 bytes total)
-  //Serial.printf("\nGetPortValue index:%d location:%d currentIndex:%d\n",index,startIndex,currentIndex);
   if (startIndex + 3 < currentIndex) {  // Exclude the end marker #####removed -4 as initially there is no marker
     portValue.type[0] = message[startIndex];
     portValue.type[1] = message[startIndex + 1];
@@ -96,6 +75,10 @@
 
 // Encrypt the message starting from the fromAddress (6th byte)
 void LoraMsg::encryptMessage() {
+  //Serial.print("\nCurIndex:");
+  //Serial.println(currentIndex);
+  //delay(100);
+  
   uint16_t messageID = getMessageID();
   // Encrypt fromAddress and beyond
   for (int i = 6; i < 12; i++) {
@@ -128,12 +111,27 @@
   return true;
 }
 
+void LoraMsg::getFromAddress(uint8_t* address) {
+  for (int i = 0; i < 6; i++) {
+    address[i] = message[i + 6];
+  }
+}
+
 uint8_t LoraMsg::getMessageLength() {
   return currentIndex;
 }
 
 // Print the message in a detailed format
 void LoraMsg::printMessage() {
+
+  //Serial.print("\nPV Method content: ");
+  //for (int i = 0; i < 128; i++) {
+  //  if (message[i] < 0x10) Serial.print("0");
+  //  Serial.print(message[i], HEX);
+  //  if (i == 11 || (i > 11 && (i - 11) % 4 == 0)) Serial.print(" ");
+  //}
+  //Serial.printf("\n currentIndex:%d\n", currentIndex);
+
   Serial.print("To: ");
   for (int i = 0; i < 6; i++) {
     if (message[i] < 0x10) Serial.print("0");  // Ensure two digits for each byte
`

### Diff: Gateway vs DualPIR

`diff
--- Gateway
+++ DualPIR
@@ -1,3 +1,4 @@
+#include <Arduino.h>
 #include "LoraMsg.h"
 
 uint16_t LoraMsg::messageCounter = 0;
@@ -20,26 +21,6 @@
   }
   currentIndex = sizeOfMsg;
   memcpy(this->toAddress, encryptedMessage, 6);  // Extract toAddress for decryption
-
-  /* Debug output to check currentIndex and message content
-  Serial.print("Encrypted message received. Current index set to: ");
-  Serial.println(currentIndex);
-  Serial.print("Message content: ");
-  for (int i = 0; i < currentIndex; i++) {
-    if (message[i] < 0x10) Serial.print("0");
-    Serial.print(message[i], HEX);
-    Serial.print(" ");
-  }
-  Serial.println();
-  */
-}
-
-uint8_t LoraMsg::getFromByte(const uint8_t byteNumber) {
-  if (byteNumber < 6) {
-    return message[6 + byteNumber];
-  } else {
-    return 0;
-  }
 }
 
 // Add PortValue to the message
@@ -63,7 +44,6 @@
 PortValue LoraMsg::getPortValue(int index) {
   PortValue portValue;
   int startIndex = 12 + index * 4;      // Start after addresses (12 bytes total)
-  //Serial.printf("\nGetPortValue index:%d location:%d currentIndex:%d\n",index,startIndex,currentIndex);
   if (startIndex + 3 < currentIndex) {  // Exclude the end marker #####removed -4 as initially there is no marker
     portValue.type[0] = message[startIndex];
     portValue.type[1] = message[startIndex + 1];
@@ -96,6 +76,10 @@
 
 // Encrypt the message starting from the fromAddress (6th byte)
 void LoraMsg::encryptMessage() {
+  Serial.print("\nCurIndex:");
+  Serial.println(currentIndex);
+  //delay(100);
+  
   uint16_t messageID = getMessageID();
   // Encrypt fromAddress and beyond
   for (int i = 6; i < 12; i++) {
@@ -134,6 +118,15 @@
 
 // Print the message in a detailed format
 void LoraMsg::printMessage() {
+
+  //Serial.print("\nPV Method content: ");
+  //for (int i = 0; i < 128; i++) {
+  //  if (message[i] < 0x10) Serial.print("0");
+  //  Serial.print(message[i], HEX);
+  //  if (i == 11 || (i > 11 && (i - 11) % 4 == 0)) Serial.print(" ");
+  //}
+  //Serial.printf("\n currentIndex:%d\n", currentIndex);
+
   Serial.print("To: ");
   for (int i = 0; i < 6; i++) {
     if (message[i] < 0x10) Serial.print("0");  // Ensure two digits for each byte
`

### Diff: Gateway vs Repeater

`diff
--- Gateway
+++ Repeater
@@ -20,26 +20,6 @@
   }
   currentIndex = sizeOfMsg;
   memcpy(this->toAddress, encryptedMessage, 6);  // Extract toAddress for decryption
-
-  /* Debug output to check currentIndex and message content
-  Serial.print("Encrypted message received. Current index set to: ");
-  Serial.println(currentIndex);
-  Serial.print("Message content: ");
-  for (int i = 0; i < currentIndex; i++) {
-    if (message[i] < 0x10) Serial.print("0");
-    Serial.print(message[i], HEX);
-    Serial.print(" ");
-  }
-  Serial.println();
-  */
-}
-
-uint8_t LoraMsg::getFromByte(const uint8_t byteNumber) {
-  if (byteNumber < 6) {
-    return message[6 + byteNumber];
-  } else {
-    return 0;
-  }
 }
 
 // Add PortValue to the message
@@ -63,7 +43,6 @@
 PortValue LoraMsg::getPortValue(int index) {
   PortValue portValue;
   int startIndex = 12 + index * 4;      // Start after addresses (12 bytes total)
-  //Serial.printf("\nGetPortValue index:%d location:%d currentIndex:%d\n",index,startIndex,currentIndex);
   if (startIndex + 3 < currentIndex) {  // Exclude the end marker #####removed -4 as initially there is no marker
     portValue.type[0] = message[startIndex];
     portValue.type[1] = message[startIndex + 1];
@@ -96,6 +75,10 @@
 
 // Encrypt the message starting from the fromAddress (6th byte)
 void LoraMsg::encryptMessage() {
+  Serial.print("\nCurIndex:");
+  Serial.println(currentIndex);
+  delay(100);
+  
   uint16_t messageID = getMessageID();
   // Encrypt fromAddress and beyond
   for (int i = 6; i < 12; i++) {
@@ -134,6 +117,15 @@
 
 // Print the message in a detailed format
 void LoraMsg::printMessage() {
+
+  //Serial.print("\nPV Method content: ");
+  //for (int i = 0; i < 128; i++) {
+  //  if (message[i] < 0x10) Serial.print("0");
+  //  Serial.print(message[i], HEX);
+  //  if (i == 11 || (i > 11 && (i - 11) % 4 == 0)) Serial.print(" ");
+  //}
+  //Serial.printf("\n currentIndex:%d\n", currentIndex);
+
   Serial.print("To: ");
   for (int i = 0; i < 6; i++) {
     if (message[i] < 0x10) Serial.print("0");  // Ensure two digits for each byte
@@ -160,3 +152,16 @@
   }
   //Serial.println("");
 }
+
+bool LoraMsg::setPortValue(const char type[2], uint16_t newValue) {
+  int num = numberOfPortValues();
+  for (int i = 0; i < num; i++) {
+    int startIndex = 12 + i * 4;
+    if (message[startIndex] == type[0] && message[startIndex + 1] == type[1]) {
+      message[startIndex + 2] = (newValue >> 8) & 0xFF;
+      message[startIndex + 3] = newValue & 0xFF;
+      return true;
+    }
+  }
+  return false;
+}
`

### Diff: Gateway vs Victron

`diff
--- Gateway
+++ Victron
@@ -20,26 +20,6 @@
   }
   currentIndex = sizeOfMsg;
   memcpy(this->toAddress, encryptedMessage, 6);  // Extract toAddress for decryption
-
-  /* Debug output to check currentIndex and message content
-  Serial.print("Encrypted message received. Current index set to: ");
-  Serial.println(currentIndex);
-  Serial.print("Message content: ");
-  for (int i = 0; i < currentIndex; i++) {
-    if (message[i] < 0x10) Serial.print("0");
-    Serial.print(message[i], HEX);
-    Serial.print(" ");
-  }
-  Serial.println();
-  */
-}
-
-uint8_t LoraMsg::getFromByte(const uint8_t byteNumber) {
-  if (byteNumber < 6) {
-    return message[6 + byteNumber];
-  } else {
-    return 0;
-  }
 }
 
 // Add PortValue to the message
@@ -63,7 +43,6 @@
 PortValue LoraMsg::getPortValue(int index) {
   PortValue portValue;
   int startIndex = 12 + index * 4;      // Start after addresses (12 bytes total)
-  //Serial.printf("\nGetPortValue index:%d location:%d currentIndex:%d\n",index,startIndex,currentIndex);
   if (startIndex + 3 < currentIndex) {  // Exclude the end marker #####removed -4 as initially there is no marker
     portValue.type[0] = message[startIndex];
     portValue.type[1] = message[startIndex + 1];
@@ -96,6 +75,10 @@
 
 // Encrypt the message starting from the fromAddress (6th byte)
 void LoraMsg::encryptMessage() {
+  Serial.print("\nCurIndex:");
+  Serial.println(currentIndex);
+  delay(100);
+  
   uint16_t messageID = getMessageID();
   // Encrypt fromAddress and beyond
   for (int i = 6; i < 12; i++) {
@@ -134,6 +117,15 @@
 
 // Print the message in a detailed format
 void LoraMsg::printMessage() {
+
+  //Serial.print("\nPV Method content: ");
+  //for (int i = 0; i < 128; i++) {
+  //  if (message[i] < 0x10) Serial.print("0");
+  //  Serial.print(message[i], HEX);
+  //  if (i == 11 || (i > 11 && (i - 11) % 4 == 0)) Serial.print(" ");
+  //}
+  //Serial.printf("\n currentIndex:%d\n", currentIndex);
+
   Serial.print("To: ");
   for (int i = 0; i < 6; i++) {
     if (message[i] < 0x10) Serial.print("0");  // Ensure two digits for each byte
`

### Diff: Gateway vs WaterTankLevel

`diff
--- Gateway
+++ WaterTankLevel
@@ -1,3 +1,4 @@
+#include <Arduino.h>
 #include "LoraMsg.h"
 
 uint16_t LoraMsg::messageCounter = 0;
@@ -20,26 +21,6 @@
   }
   currentIndex = sizeOfMsg;
   memcpy(this->toAddress, encryptedMessage, 6);  // Extract toAddress for decryption
-
-  /* Debug output to check currentIndex and message content
-  Serial.print("Encrypted message received. Current index set to: ");
-  Serial.println(currentIndex);
-  Serial.print("Message content: ");
-  for (int i = 0; i < currentIndex; i++) {
-    if (message[i] < 0x10) Serial.print("0");
-    Serial.print(message[i], HEX);
-    Serial.print(" ");
-  }
-  Serial.println();
-  */
-}
-
-uint8_t LoraMsg::getFromByte(const uint8_t byteNumber) {
-  if (byteNumber < 6) {
-    return message[6 + byteNumber];
-  } else {
-    return 0;
-  }
 }
 
 // Add PortValue to the message
@@ -63,7 +44,6 @@
 PortValue LoraMsg::getPortValue(int index) {
   PortValue portValue;
   int startIndex = 12 + index * 4;      // Start after addresses (12 bytes total)
-  //Serial.printf("\nGetPortValue index:%d location:%d currentIndex:%d\n",index,startIndex,currentIndex);
   if (startIndex + 3 < currentIndex) {  // Exclude the end marker #####removed -4 as initially there is no marker
     portValue.type[0] = message[startIndex];
     portValue.type[1] = message[startIndex + 1];
@@ -96,6 +76,10 @@
 
 // Encrypt the message starting from the fromAddress (6th byte)
 void LoraMsg::encryptMessage() {
+  Serial.print("\nCurIndex:");
+  Serial.println(currentIndex);
+  //delay(100);
+  
   uint16_t messageID = getMessageID();
   // Encrypt fromAddress and beyond
   for (int i = 6; i < 12; i++) {
@@ -134,6 +118,15 @@
 
 // Print the message in a detailed format
 void LoraMsg::printMessage() {
+
+  //Serial.print("\nPV Method content: ");
+  //for (int i = 0; i < 128; i++) {
+  //  if (message[i] < 0x10) Serial.print("0");
+  //  Serial.print(message[i], HEX);
+  //  if (i == 11 || (i > 11 && (i - 11) % 4 == 0)) Serial.print(" ");
+  //}
+  //Serial.printf("\n currentIndex:%d\n", currentIndex);
+
   Serial.print("To: ");
   for (int i = 0; i < 6; i++) {
     if (message[i] < 0x10) Serial.print("0");  // Ensure two digits for each byte
`

