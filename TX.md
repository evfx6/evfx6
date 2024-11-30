#include <SPI.h>
#include "nRF24L01.h"
#include "RF24.h"
#define LED_PINgreen 3
#define LED_PINred 5



RF24 radio(9, 10); // "создать" модуль на пинах 9 и 10 Для Уно
//RF24 radio(9,53); // для Меги

byte recieved_data[4]; // массив принятых данных
byte relay = 2; //----- реле на 2 цифрово(ОТКРЫТИЕ)
byte relay2 = 4; //------ реле на 4 цифрово(ЗАКРЫТИЕ)
byte dynamic = 6;
byte LAZER = 7;
boolean conOpen;
boolean conClouse;
boolean FTRST;
unsigned long last_time; //-----переменная времени 




byte address[][6] = {"1Node", "2Node", "3Node", "4Node", "5Node", "6Node"}; //возможные номера труб

void setup() { 

  digitalWrite(relay,0);
  digitalWrite(relay2,0);
  digitalWrite(dynamic, 0);
  digitalWrite(LAZER, 0);
  
  pinMode(relay, OUTPUT); // настроить пин реле как выход
  pinMode(relay2, OUTPUT);
  pinMode(dynamic, OUTPUT);
  pinMode(LAZER, OUTPUT);
  pinMode(LED_PINgreen, OUTPUT);
  pinMode(LED_PINred, OUTPUT);
  
  pinMode(A1, INPUT_PULLUP); //-----пин концевика ОТКРЫТИЯ
  pinMode(A2, INPUT_PULLUP);//-----пин концевика  ЗАКРЫТИЯ
  pinMode(A3, INPUT_PULLUP);//------пин датчика препятствия

  


  radio.begin(); //активировать модуль
  radio.setAutoAck(1);         //режим подтверждения приёма, 1 вкл 0 выкл
  radio.setRetries(0, 15);    //(время между попыткой достучаться, число попыток)
  radio.enableAckPayload();    //разрешить отсылку данных в ответ на входящий сигнал
  radio.setPayloadSize(32);     //размер пакета, в байтах

  radio.openReadingPipe(1, address[0]);     //хотим слушать трубу 0
  radio.setChannel(0x60);  //выбираем канал (в котором нет шумов!)

  radio.setPALevel (RF24_PA_MAX); //уровень мощности передатчика. На выбор RF24_PA_MIN, RF24_PA_LOW, RF24_PA_HIGH, RF24_PA_MAX
  radio.setDataRate (RF24_250KBPS); //скорость обмена. На выбор RF24_2MBPS, RF24_1MBPS, RF24_250KBPS
  radio.powerUp(); //начать работу
  radio.startListening();  //начинаем слушать эфир, мы приёмный модуль
}

void loop() {
     conOpen = digitalRead(A1); //--------------концевик на открытия ворот на пине А1
    conClouse = digitalRead(A2); //-------------концевик на закрытия ворот на пине А2
    FTRST = digitalRead(A3);
    
  lampaClouse();  //--------------------------сигнал на лампу(КРАСНОГО цвета) на ЗАКРЫТИЯ ворот                   
 lampaOpen();  //-------------------------------сигнал на лампу(ЗЕЛЕННОГО цвета) на ОТКРЫТИЯ ворот
  releOpen();  //------------------------------сигнал реле на ОТКРЫТИЯ ворот
  releClouse();  //----------------------------сигнал реле на ЗАКРЫТИЕ ворот
  stoped();
  lazer();
  
   byte pipeNo;
  while ( radio.available(&pipeNo)) {  // слушаем эфир со всех труб
    radio.read( &recieved_data, sizeof(recieved_data) ); // чиатем входящий сигнал 
        }     
      }
      
 void lampaClouse(){
  if( round(millis() / 500) % 2 == 0 && conClouse == 1)
  digitalWrite(LED_PINred, recieved_data[2]);          //------МИГАНИЕ КРАСНЫМ СВЕТОДИОДОМ 
  else
   digitalWrite(LED_PINred, LOW);
  }  
     
   void lampaOpen(){
  if( round(millis() / 500) % 2 == 0 && conOpen == 1)
  digitalWrite(LED_PINgreen, recieved_data[3]);         //------МИГАНИЕ ЗЕЛЕННЫМ СВЕТОДИОДОМ
  else
   digitalWrite(LED_PINgreen, LOW);
  }
  
 void releOpen(){
  if (conOpen == 1)
  digitalWrite(relay, recieved_data[0]);              //-------СИГНАЛ НА РЕЛЕ ОТКРЫТИЯ 
  else
  digitalWrite(relay, LOW);
  }
  
   void releClouse(){
  if (conClouse == 1 && FTRST == 0)
  digitalWrite(relay2, recieved_data[1]); //-------СИГНАЛ НА РЕЛЕ ЗАКРЫТИЯ
  else 
  digitalWrite(relay2, LOW);
  }
  
 void stoped(){ 
    if (FTRST == 1 && recieved_data[1] && conClouse == 1)
     digitalWrite(dynamic, 1);           //------------------СИГНАЛ НА ДИНАМИК ЕЛИ ЕСТЬ ПРИПЯТСЯТВИЕ                       
     else
     digitalWrite(dynamic, 0);
    }
 void lazer(){ 
    if (recieved_data[1] && conClouse == 1)
     digitalWrite(LAZER, 1);           //------------------СИГНАЛ НА ДИНАМИК ЕЛИ ЕСТЬ ПРИПЯТСЯТВИЕ                       
     else
     digitalWrite(LAZER, 0);
    }

    
  // пин 2 (relay) - РЕЛЕ НА ОТКРЫТИЕ
 // ПИН 4 (relay2) - РЕЛЕ НА ЗАКРЫТИЕ
 // ПИН 3 (LED_PINgreen) - СИГНАЛ НА ЛАМПУ ЗЕЛЕННЫЙ 
 // ПИН 5 (LED_PINred) - СИГНАЛ НА ЛАМПУ КРАСНЫЙ
 // ПИН 6 (dynamic) - ДИНАМИК НА ПРИПЯТСТВИЕ 
 // ПИН 7 (LAZER) - ЛАЗЕР НА ФОТОРЕЗИСТОР
 // ПИН А3 (FTRST) - ДАТЧИК ФОТОРЕЗИСТОР 
 // ПИН А2 (conClouse) - КОНЦЕВИК НА ЗАКРЫТИЕ 
 // ПИН А1 (conOpen) - КОНЦЕВИК НА ОТКРЫТИЕ
 
