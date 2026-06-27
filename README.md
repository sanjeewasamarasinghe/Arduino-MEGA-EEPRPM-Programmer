My old EEPROM Programmer – try to document for further development.

frant side

<img width="400" height="384" alt="pic-333" src="https://github.com/user-attachments/assets/a1c36a71-8607-43df-a85e-2bb214ef5482" />

back side

<img width="400" height="421" alt="pic-444" src="https://github.com/user-attachments/assets/5310ddbd-184a-4bb5-b364-5405fe0a3932" />







Manly I develop this device using local suppler materials, and internet resources. 

This are links I refer and I’m very much thanks for them for this knowledge
Links…  
-	https://www.qcontinuum.org/eeprom-programmer#h.sorddbft6u9r
-	https://oshwlab.com/dana_peters/arduino-mega-eeprom-programmer
-	https://sites.google.com/site/ericmklaus/projects-1/arduino-mega-flash-programmer
-	https://github.com/crmaykish/AT28C-EEPROM-Programmer-Arduino

I use EEPROM Chip Atmega 8K X 8 bit – AT 28 C 64 K - to program 
But in these resources have 32K X 8 bit – AT 28 C 256 K – chip to program





<img width="100" height="175" alt="pic-6666" src="https://github.com/user-attachments/assets/cada544c-cc62-4457-8f47-9590be68c5d8" />

Then I change Some program according to it …


<img width="500" height="229" alt="pic-555" src="https://github.com/user-attachments/assets/c9b1550d-7dce-45e0-825b-b8696caff4b1" />
<img width="500" height="268" alt="pic4444" src="https://github.com/user-attachments/assets/b69d06a7-a317-4416-8f26-ccb3f01c826f" />


Ardiuno program for Read this EEPROM 
----------------------------------------------------------------------------------------------------------------------------------------
#define DELAY_TIME    

// Control pins
#define CHIP_ENABLE   41
#define OUTPUT_ENABLE 39
#define WRITE_ENABLE  40

// I/O pins
const int I[] = {49, 48, 47, 46, 45, 44, 43, 42};

// Address pins
const int A[] = {37, 36, 35, 34, 33, 32, 31, 30, 22, 23, 24, 25, 26};

// Input buffers
char readAddr[5];
char readData[3];

void setup()
{
  // Define control pins and turn them all off
  pinMode(OUTPUT_ENABLE, OUTPUT);
  pinMode(WRITE_ENABLE, OUTPUT);
  pinMode(CHIP_ENABLE, OUTPUT);
  digitalWrite(OUTPUT_ENABLE, HIGH);
  digitalWrite(WRITE_ENABLE, HIGH);
  digitalWrite(CHIP_ENABLE, HIGH);

  // Set address bus to outputs
  for (int i = 0; i < (sizeof(A) / sizeof(int) ); i++)
  {
    pinMode(A[i], OUTPUT);
  }

  Serial.begin(115200);
}

void loop()
{
  while (Serial.available() > 0)
  {
    String input = Serial.readStringUntil('\n');
    String command = input.substring(0, 2);

    input.substring(2, 6).toCharArray(readAddr, 5);

    uint16_t addr = (uint16_t)strtol(readAddr, NULL, 16);
    
    if (command.equals("RD"))
    {
      Serial.println(readByte(addr), HEX);
    }

    else if (command.equals("WR")){
      input.substring(6, 8).toCharArray(readData, 3);
      byte data = (byte)strtol(readData, NULL, 16);
      writeByte(addr, data);
      Serial.println("DONE");
    }
    else
    {
      Serial.println("Bad input: " + input);
    }
  }
}

// Set data bus to INPUT or OUTPUT
void setDataBusMode(int mode)
{
  if (mode == INPUT || mode == OUTPUT)
  {
    for (int i = 0; i < 8; i++)
    {
      pinMode(I[i], mode);
    }
  }
}

// Write an address to the address bus
void setAddress(int addr)
{
  for (int i = 0; i < (sizeof(A) / sizeof(int) ); i++)
  {
    int a = (addr & (1 << i)) > 0;
    digitalWrite(A[i], a);
  }
}

void writeByte(uint16_t addr, byte val)
{
  digitalWrite(OUTPUT_ENABLE, HIGH);
  setDataBusMode(OUTPUT);
  setAddress(addr);

  // Send data value to data bus
  for (int i = 0; i < 8; i++)
  {
    int a = (val & (1 << i)) > 0;
    digitalWrite(I[i], a);
  }

  // Commit data write
  digitalWrite(CHIP_ENABLE, LOW);
  delay(DELAY_TIME);
  digitalWrite(WRITE_ENABLE, LOW);
  delay(DELAY_TIME);
  digitalWrite(WRITE_ENABLE, HIGH);
  delay(DELAY_TIME);
  digitalWrite(CHIP_ENABLE, HIGH);
  delay(DELAY_TIME);
}

byte readByte(uint16_t addr)
{
  byte data = 0;

  setDataBusMode(INPUT);

  // Write the addr
  for (int i = 0; i < (sizeof(A) / sizeof(int) ); i++)
  {
    int a = (addr & (1 << i)) > 0;
    digitalWrite(A[i], a);
  }

  digitalWrite(CHIP_ENABLE, LOW);
  digitalWrite(OUTPUT_ENABLE, LOW);
  delay(DELAY_TIME);

  // Read data bus
  for (int i = 0; i < 8; i++)
  {
    int d = digitalRead(I[i]);
    data += (d << i);
  }

  digitalWrite(OUTPUT_ENABLE, HIGH);
  digitalWrite(CHIP_ENABLE, HIGH);
  delay(DELAY_TIME);

  return data;
}
-------------------------------------------------------------------------------------------------------------------------------




