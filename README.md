## **PRACTICA 7 : Buses de comunicación III (I2S)**
En esta práctica nos enfocamos en el protocolo de comunicación I2S (Inter-IC Sound), utilizado para transferir señales de audio digital 
entre dispositivos. Se realizaron dos ejercicios prácticos para entender su funcionamiento: el primero reproduce audio almacenado 
en la memoria interna del ESP32, y el segundo reproduce un archivo de audio WAV desde una tarjeta SD externa.
Para facilitar la reproducción de audio de alta calidad, se utilizó el módulo amplificador MAX98357A, que convierte señales digitales 
I2S en señales analógicas para ser emitidas por un altavoz.
El ESP32 fue elegido para esta práctica debido a su compatibilidad nativa con I2S, su capacidad de procesamiento y su flexibilidad en 
aplicaciones de audio.


## **Ejercicio Practico 1 reproducción desde memoria interna:**
**Codigo main.cpp:**
```
#include "AudioGeneratorAAC.h" 
#include "AudioOutputI2S.h" 
#include "AudioFileSourcePROGMEM.h" 
#include "sampleaac.h" 
AudioFileSourcePROGMEM *in; 
AudioGeneratorAAC *aac; 
AudioOutputI2S *out; 
void setup(){ 
Serial.begin(115200); 
in = new AudioFileSourcePROGMEM(sampleaac, sizeof(sampleaac)); 
aac = new AudioGeneratorAAC(); 
out = new AudioOutputI2S(); 
out -> SetGain(0.125); 
out -> SetPinout(40,39,38); 
aac->begin(in, out); 
} 
void loop(){ 
if (aac->isRunning()) { 
aac->loop(); 
} else { 
aac -> stop(); 
Serial.printf("Sound Generator\n"); 
delay(1000); 
} 
}

```
El primer código reproduce un archivo de audio en formato AAC almacenado directamente en la memoria interna del ESP32. Se utilizan 
las bibliotecas AudioGeneratorAAC, AudioOutputI2S y AudioFileSourcePROGMEM, junto con un archivo llamado sampleaac.h que contiene el audio.

En setup(), se inicializa la comunicación serial y se crean los objetos para la fuente de audio, el decodificador AAC y la salida I2S. 
Se configura la ganancia de audio a un volumen moderado y se asignan los pines I2S. Luego, se inicia la reproducción de audio conectando 
la fuente y la salida.

En loop(), mientras la reproducción esté activa, el archivo de audio se procesa continuamente; cuando termina, se detiene y se muestra en 
el monitor serie el mensaje "Sound Generator", indicando que el archivo se ha reproducido completamente.

La salida que aparece en el monitor es:
```
Sound Generator
Sound Generator
Sound Generator
...
```

## **Ejercicio Practico 2 reproducir un archivo WAVE en ESP32 desde una tarjeta SD externa:**
**Codigo main.cpp:**
```
#include "Audio.h"
#include "SD.h"
#include "FS.h"

// Definir los pines para ESP32-S3
#define SD_CS         10  
#define SPI_MOSI      11  
#define SPI_MISO      13  
#define SPI_SCK       12  
#define I2S_DOUT      6   
#define I2S_BCLK      5   
#define I2S_LRC       4   

Audio audio; 

void setup(){ 
  pinMode(SD_CS, OUTPUT); 
  digitalWrite(SD_CS, HIGH); 
  SPI.begin(SPI_SCK, SPI_MISO, SPI_MOSI); 
  Serial.begin(115200); 
  SD.begin(SD_CS); 
  audio.setPinout(I2S_BCLK, I2S_LRC, I2S_DOUT); 
  audio.setVolume(10); // 0...21 

  audio.connecttoFS(SD, "patito_juan_short.wav"); 
} 
 
void loop(){ 
    audio.loop(); 
} 
 
// optional 
void audio_info(const char *info){ 
    Serial.print("info        "); Serial.println(info); 
} 
void audio_id3data(const char *info){  //id3 metadata 
    Serial.print("id3data     ");Serial.println(info); 
} 
void audio_eof_mp3(const char *info){  //end of file 
    Serial.print("eof_mp3     ");Serial.println(info); 
} 
void audio_showstation(const char *info){ 
    Serial.print("station     ");Serial.println(info); 
} 
void audio_showstreaminfo(const char *info){ 
    Serial.print("streaminfo  ");Serial.println(info); 
} 
void audio_showstreamtitle(const char *info){ 
    Serial.print("streamtitle ");Serial.println(info); 
} 
void audio_bitrate(const char *info){ 
    Serial.print("bitrate     ");Serial.println(info); 
} 
void audio_commercial(const char *info){  //duration in sec 
    Serial.print("commercial  ");Serial.println(info); 
} 
void audio_icyurl(const char *info){  //homepage 
    Serial.print("icyurl      ");Serial.println(info); 
} 
void audio_lasthost(const char *info){  //stream URL played 
    Serial.print("lasthost    ");Serial.println(info); 
} 
void audio_eof_speech(const char *info){ 
    Serial.print("eof_speech  ");Serial.println(info); 
} 



```
El segundo código permite reproducir un archivo de audio WAV guardado en una tarjeta SD externa. Se usan las bibliotecas Audio, SD y 
FS para manejar tanto la lectura de la tarjeta como la reproducción de audio mediante I2S.

En setup(), se inicializa la comunicación SPI para conectar la tarjeta SD, se inicia la comunicación serial, y se configuran los pines 
de salida para I2S. También se ajusta el volumen y se conecta el objeto de audio al archivo WAV especificado en la tarjeta.

El loop() simplemente llama a audio.loop(), encargado de mantener la reproducción activa durante la ejecución. Opcionalmente, el programa 
incluye funciones para mostrar información adicional del audio en el monitor serie, como la tasa de bits, el nombre del archivo o datos ID3.


